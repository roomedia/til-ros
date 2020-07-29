# 서론

오늘은 딜리버리 서비스 로봇을 만들기 전에, 실제 집을 돌아다니면서 지도를 생성하면 얼마나 인식률이 좋은지, 유리나 거울 같은 장애물을 마주쳤을 때 어떤 모습을 보이는지 확인해보도록 하겠습니다.

# 집 구조

원래 로보티즈의 예제처럼, 메뉴를 주문 받으면 주방에 받으러 갔다가 거실로 돌아오는 걸 상상하고 있습니다. 따라서 거실과 주방만 지도를 생성하도록 하겠습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fm7VfN%2FbtqF5zWOtMs%2Fiz8laGyj245Jbrm3Q0Eie1%2Fimg.jpg)

거실에는 커다란 유리가 있고, 한 쪽 벽에는 유리장이 있습니다. 신발장에도 거울이 있죠.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcms0pm%2FbtqF7MnjFGI%2F5IKEUdafjdKlzC0TPGqQMK%2Fimg.jpg)

주방의 경우 유리, 거울은 없지만 대신 식탁 때문에 좀 복잡합니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbtHIwb%2FbtqF7LBUzYz%2F6rDa75pdpU5Wh3BWMbkV40%2Fimg.jpg)

창피하네요.. 아무튼 시작!

# SLAM

아래부터는 책 335p를 따라합니다.

## 11.2.4. SLAM 실행

저는 터틀봇 버거를 가지고 있기 때문에 다음과 같이 환경변수를 설정했습니다.

```sh
export TURTLEBOT3_MODEL=burger
```

### roscore

Remote PC

```sh
roscore & # & 옵션을 통해 백그라운드에서 실행할 수 있습니다. 터미널 창 절약 목적
```

### 로봇 구동

Turtlebot PC

```sh
roslaunch turtlebot3_bringup turtlebot3_robot.launch
```

레이저 센서 값, 바퀴 위치, 배터리 상태, tf 등 터틀봇의 상태를 나타내는 각종 토픽을 전달합니다.

### SLAM 패키지 실행

Remote PC

```sh
export TURTLEBOT3_MODEL=burger
roslaunch turtlebot3_slam turtlebot3_slam.launch
```

해당 파일의 내용은 다음과 같습니다.

```xml
<launch>
  <!-- Arguments -->
  <arg name="model" default="$(env TURTLEBOT3_MODEL)" doc="model type [burger, waffle, waffle_pi]"/>
  <arg name="slam_methods" default="gmapping" doc="slam type [gmapping, cartographer, hector, karto, frontier_exploration]"/>
  <arg name="configuration_basename" default="turtlebot3_lds_2d.lua"/>
  <arg name="open_rviz" default="true"/>

  <!-- TurtleBot3 -->
  <include file="$(find turtlebot3_bringup)/launch/turtlebot3_remote.launch">
    <arg name="model" value="$(arg model)" />
  </include>

  <!-- SLAM: Gmapping, Cartographer, Hector, Karto, Frontier_exploration, RTAB-Map -->
  <include file="$(find turtlebot3_slam)/launch/turtlebot3_$(arg slam_methods).launch">
    <arg name="model" value="$(arg model)"/>
    <arg name="configuration_basename" value="$(arg configuration_basename)"/>
  </include>

  <!-- rviz -->
  <group if="$(arg open_rviz)">
    <node pkg="rviz" type="rviz" name="rviz" required="true"
          args="-d $(find turtlebot3_slam)/rviz/turtlebot3_$(arg slam_methods).rviz"/>
  </group>
</launch>
```

`slam_methods` 인자를 통해 `gmapping`, `cartographer`, `hector`, `karto`, `frontier_exploration` 중 하나의 SLAM 알고리즘을 선택할 수 있습니다.

터틀봇이 보내는 메시지를 수신하는 `turtlebot3_remote.launch`가 실행되고, 설정한 SLAM 알고리즘에 대한 launch 파일이 실행됩니다.

`open_rviz` 인자가 `true`일 경우 해당하는 SLAM 알고리즘을 시각화하기 위한 rviz 파일이 실행됩니다. 이 경우 아래 RViz 실행 명령어를 입력하지 않아도 됩니다. (해당 명령어로 열리는 rviz 파일이 동일합니다.)

### RViz 실행

Remote PC

```sh
export TURTLEBOT3_MODEL=burger
rosrun rviz rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_slam.rviz
```

다음과 같이 표시됩니다. gmapping 알고리즘을 사용하였으며, 유리의 경우 유리 너머까지 측정되었고, 식탁 의자 등은 다리만 표시되었네요. 유리, 거울 등은 벽으로 인식하지 못하기 때문에 피하라는 듯 합니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb4oe2p%2FbtqF9lvTgA3%2FDclsZy66P5CuQ7ckXIbCUk%2Fimg.png)

### 토픽 메시지 저장

Remote PC

```
rosbag record -O scan_data /map /scan /tf
```

`/scan`, `/tf`, /map  토픽을 `scan_data.bag` 파일로 저장합니다. 실험 당시의 센서 값을 재현할 수 있습니다. 이거 안 하고 로봇 조종해서 아았~! 하고 다시 첨부터 했습니다.... 지도를 다 수집한 뒤 ctrl-c를 눌러 종료합니다. 책에는 /scan /tf만 받는데, /map을 안 받으면 rosbag으로 불러왔을 때 지도 저장이 안 됩니다...

### 로봇 조종

Remote PC

```
roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

이외에도 `turtle_teleop_multi_key` 등 다양한 원격 조종 패키지를 사용할 수 있습니다.

`turtle_teleop_multi_key` (ROS Melodic 지원) 사용법

[2020/07/25 - \[ros2 + cpp + python\] - 24일차 - turtlebot3\_teleop\_key로 다중 키 입력 받기\_2￼](https://roomedia.tistory.com/entry/24일차-turtlebot3teleopkey로-다중-키-입력-받기2)

```
cd ~/catkin_ws/src
git clone https://github.com/EngHyu/turtle_teleop_multi_key.git
cd ~/catkin_ws
catkin_make
```

### 지도 작성

Remote PC

```
rosrun map_server map_saver -f ~/map
```

~ 폴더 아래에 `map.pgm`, `map.yaml` 파일이 생성됩니다. 아았!~ 하기 전에 지도 저장부터 할 걸.... 로봇에서 수신한 데이터를 사용하여 현재 시점의 지도를 만들기 때문에 remote.launch를 끄시면 안 됩니다. 껐다가 왜 안 되지?? 하고 기다리길 30분.. 깨달았습니다.... 그래서 간단하게 방 하나만 측정한 후 지도를 만들어봤습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbglVff%2FbtqF6DdGoPn%2Fg7e7OGYrJYkceHDFd7PQr1%2Fimg.png)

가운데 동그랗게 뚤린 건 짐볼입니다ㅋㅋ

### bag 파일 이용 SLAM

시뮬레이션을 위해 다음 파일을 내려받습니다. 책과는 주소가 약간 달라졌습니다.

Remote PC

```
roscd turtlebot3_slam/bag
wget https://github.com/ROBOTIS-GIT/bags/blob/master/TB3_WAFFLE_SLAM.bag
```

기존 방법과 동일하게 진행하되, `rosbag` 명령어가 `recored` 대신 `play`로 변경되었습니다.

```
# shell 1
roscore &
export TURTLEBOT3_MODEL=burger
roslaunch turtlebot3_bringup turtlebot3_remote.launch

# shell 2
rosrun rviz rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_gmapping.rviz &
rosbag play `rospack find turtlebot3_slam`/bag/TB3_WAFFLE_SLAM.bag

# shell 3
rosrun map_server map_saver -f ~/map
```

map\_saver를 실행하는 시점에 따라 취득한 데이터 양이 달라지기 때문에 생성되는 지도 또한 달라집니다. 왼쪽은 play 초기, 오른쪽은 끝나기 직전입니다. rosbag play가 끝나고나면 map 데이터를 취득할 수 없기 때문에 지도를 생성할 수 없습니다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbMMPSH%2FbtqF8uNAGkO%2F9YE8rHyBKLFVH12RuRbGj0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbDBTu2%2FbtqF5eyHeJa%2FatH6LFBTzy3RWqby1UTy3k%2Fimg.png)

\-r 옵션으로 rosbag의 재생 속도를 조절할 수 있습니다. 다음 명령어는 열 배 속도로 재생합니다.

```
rosbag play -r 10 ~/scan_data.bag
```

이제 Gazebo로 시뮬레이션 해보기잇~!