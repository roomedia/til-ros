# TurtleBot3 개발 환경
- 공식 터틀봇3 위키를 참조합니다.
[http://turtlebot3.robotis.com](http://turtlebot3.robotis.com)
- 아래 패키지를 설치 후 SLAM, Navigation, Gazebo 실습을 진행합니다.
```
cd ~/catkin_ws/src/
git clone https://github.com/ROBOTIS-GIT/turtlebot3.git
git clone https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
git clone https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
cd ~/catkin_ws && catkin_make
```
실제 터틀봇을 작동할 때에는 연산량이 큰 rviz, gazebo 등을 실행하는 Remote PC와 터틀봇이 같은 네트워크 안에 있어야 합니다. 터틀봇과 Remote PC의 `ROS_MATER_URI`와 `ROS_HOSTNAME`을 다음과 같이 설정합니다.
```
# ~/.bashrc
# env for turtlebot
export ROS_MSTER_URI=http://{IP_OF_REMOTE}:11311/
export ROS_HOSTNAME={IP_OF_TURTLEBOT}
```
```
# ~/.bashrc
# env for remote pc
export ROS_MSTER_URI=http://{IP_OF_REMOTE}:11311/
export ROS_HOSTNAME={IP_OF_REMOTE}
```
PC를 위한 패키지도 설치합니다. `{VERSION}` 부분을 설치된 ROS1 버전으로 치환합니다.
```
sudo apt-get install ros-{VERSION}-joy ros-{VERSION}-teleop-twist-joy ros-{VERSION}-teleop-twist-keyboard ros-{VERSION}-laser-proc ros-{VERSION}-rgbd-launch ros-{VERSION}-depthimage-to-laserscan ros-{VERSION}-rosserial-arduino ros-{VERSION}-rosserial-python ros-{VERSION}-rosserial-server ros-{VERSION}-rosserial-client ros-{VERSION}-rosserial-msgs ros-{VERSION}-amcl ros-{VERSION}-map-server ros-{VERSION}-move-base ros-{VERSION}-urdf ros-{VERSION}-xacro ros-{VERSION}-compressed-image-transport ros-{VERSION}-rqt-image-view ros-{VERSION}-gmapping ros-{VERSION}-navigation ros-{VERSION}-interactive-markers
```
## 3차원 시각화 도구 RViz를 이용하여 시뮬레이션 하기
아래 명령어를 통해 Turtlebot 시뮬레이션을 실행합니다. 매번 export하기 귀찮다면, export 내용을 `.bashrc`에 작성해줍니다.
```
# one shell
export TURTLEBOT3_MODEL=burger
roslaunch turtlebot3_fake turtlebot3_fake.launch
# another shell
roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```
wxad가 전후좌우에 매칭되며 s는 멈춤을 의미합니다. 터틀봇을 이동하면 지속적으로 odometry 정보를 시각화하는 걸 볼 수 있습니다.
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FcbABE7%2FbtqFz3csGHj%2FxnVhNLPjhxLhaZXPpXGeA0%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FcbABE7%2FbtqFz3csGHj%2FxnVhNLPjhxLhaZXPpXGeA0%2Fimg.png)
미리 정의된 tf 정보를 확인할 수도 있습니다.
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbudeBC%2FbtqFzuV8d6M%2FGDkJTwjkgBk4f8kNSRLVH1%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbudeBC%2FbtqFzuV8d6M%2FGDkJTwjkgBk4f8kNSRLVH1%2Fimg.png)
이러한 시뮬레이션은 하드웨어 작업 이전에 정해진 병진 속도 및 회전 속도대로 작동하는 걸 확인하는 용도로 사용한다고 합니다.
## 3차원 물리 엔진 도구 Gazebo를 이용하여 시뮬레이션 하기
RViz가 센서 데이터를 시각화 하는 데에 특화되어 있다면 Gazebo는 시뮬레이션을 위한 도구입니다. 물리 엔진 기반으로 동작하며 지면 위의 마찰력, 장애물과 센서 데이터를 가상으로 생성합니다. 이런 데이터는 다시 RViz로 시각화 할 수 있어, Gazebo와 RViz에서 SLAM 데이터를 이용한 시뮬레이션을 터틀봇 없이 수행할 수 있습니다.
### SLAM
```
# run Gazebo
export TURTLEBOT3_MODEL=waffle
roslaunch turtlebot3_gazebo turtlebot3_world.launch
```
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FFt90h%2FbtqFzcg8wJa%2FvQzXUok2ImfmlBrQCHHT2K%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FFt90h%2FbtqFzcg8wJa%2FvQzXUok2ImfmlBrQCHHT2K%2Fimg.png)
```
# run SLAM
export TURTLEBOT3_MODEL=waffle
roslaunch turtlebot3_slam turtlebot3_slam.launch
```
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FcblaLB%2FbtqFz3Q5YpR%2FZFCu1vHZJzyPSiFOcQRFm0%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FcblaLB%2FbtqFz3Q5YpR%2FZFCu1vHZJzyPSiFOcQRFm0%2Fimg.png)
```
# run RViz
export TURTLEBOT3_MODEL=waffle
rosrun rviz rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_slam.rviz
```
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FA4VkV%2FbtqFzcnRQLT%2FKH0DuQX0DQWlIoKfPMmYcK%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FA4VkV%2FbtqFzcnRQLT%2FKH0DuQX0DQWlIoKfPMmYcK%2Fimg.png)
```
# turtlebot remote control
roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FLWkGe%2FbtqFAqygPDR%2FU8MRg9ptlCBb3KfwwgOezk%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FLWkGe%2FbtqFAqygPDR%2FU8MRg9ptlCBb3KfwwgOezk%2Fimg.png)
```
# export map
rosrun map_server map_saver -f ~/map
```
### Navigation
```
# run Gazebo
export TURTLEBOT3_MODEL=waffle
roslaunch turtlebot3_gazebo turtlebot3_world.launch
```
```
# run Navigation
export TURTLEBOT3_MODEL=waffle
roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map.yaml
```
```
# run RViz and set Destination
export TURTLEBOT3_MODEL=waffle
rosrun rviz rviz -d `rospack find turtlebot3_navigation`/rviz/turtlebot3_nav.rviz
```