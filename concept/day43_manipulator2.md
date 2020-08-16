# OpenManipulator & MoveIt! 의존성 패키지 설치
https://emanual.robotis.com/docs/en/platform/openmanipulator_x/quick_start_guide/

```sh
sudo apt-get install ros-melodic-moveit* ros-melodic-gazebo* ros-melodic-industrial-core ros-melodic-ros-controllers
```

```sh
cd ~/catkin_ws/src
git clone -b melodic-devel https://github.com/ROBOTIS-GIT/dynamixelSDK.git
git clone -b melodic-devel https://github.com/ROBOTIS-GIT/dynamixel-workbench.git
git clone -b melodic-devel https://github.com/ROBOTIS-GIT/dynamixel-workbench-msgs.git

git clone -b melodic-devel https://github.com/ROBOTIS-GIT/open_manipulator.git
git clone -b melodic-devel https://github.com/ROBOTIS-GIT/open_manipulator_msgs.git
git clone -b melodic-devel https://github.com/ROBOTIS-GIT/open_manipulator_simulations.git
git clone -b melodic-devel https://github.com/ROBOTIS-GIT/robotis_manipulator.git
cd ~/catkin_ws && catkin_make
```

# RViz 실행하기
다음 명령어를 실행하여 이전 시간에 했던 것처럼 `Open Manipulator`를 `RViz`를 통해 시각화 해볼 수 있습니다.

```sh
roslaunch open_manipulator_description open_manipulator_rviz.launch
```

다만, 이번에는 `urdf`가 약간 다릅니다. `Open Manipulator`는 이전에도 설명했었던 `xacro`(XML Macro)를 사용합니다. `xacro`를 사용하면 반복되는 부분을 쉽게 처리할 수 있습니다. `xacro` 문법이 기타 `urdf` 태그와 충돌하는 걸 방지하기 위해 `xmlns:xacro="http://ros.org/wiki/xacro"` 속성을 설정하여 `xacro` 태그에 대한 접두사를 지정해줍니다.

```xml
<robot name="open_manipulator" xmlns:xacro="http://ros.org/wiki/xacro">
  <!-- ... -->
  <xacro:include filename="$(find open_manipulator_description)/urdf/open_manipulator.gazebo.xacro" />
  <xacro:include filename="$(find open_manipulator_description)/urdf/materials.xacro" />
  <!-- ... -->
  <limit velocity="4.8" effort="1" lower="${-pi*0.9}" upper="${pi*0.9}" />
  <!-- ... -->
</robot>
```

이외에도 기존 urdf에서 불가능한 연산 작업 등을 할 수 있습니다.

더 알아보기: http://ros.org/wiki/xacro

![RViz 실행 화면](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbhODDY%2FbtqGJwKQkVl%2FlitUZGwbkuUCZB5iSD0Ikk%2Fimg.png)

# Gazebo 실행하기
다음 명령어를 실행하여 `Gazebo`에서 `Open Manipulator`를 시뮬레이션할 수 있습니다.

```sh
roslaunch open_manipulator_gazebo open_manipulator_gazebo.launch
```

![Gazebo1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FctKWbA%2FbtqGFabmskE%2FUGWkt4bLtfoMdxkMEo1AN0%2Fimg.png)

`rostopic`에서 다음과 같은 토픽을 발행하면 자세를 움직일 수 있습니다.

```sh
rostopic pub /open_manipulator/joint2_position/command std_msgs/Float64 "data: -1.0" --once
```

![Gazebo2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FnEZLb%2FbtqGDQdbPI1%2FsFwoG1zfw9IrnyztShgr2k%2Fimg.png)

`joint2` 대신 다른 관절 이름을 사용하면 해당 관절이 회전하는 걸 볼 수 있습니다.