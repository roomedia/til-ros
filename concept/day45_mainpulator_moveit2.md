# MoveIt! 실행하기
`Open Manipulator` 깃허브 레포지토리를 다운받았고, `catkin_make`를 진행했다면, 다음 코드를 실행하여 기본 예제를 실행해볼 수 있습니다.

```sh
roslaunch open_manipulator_moveit demo.launch
```

![실행화면](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fqti4p%2FbtqGH2w4hxX%2FTZQRsrGTv8umHRFd00TeA1%2Fimg.png)

# MotionPlanning
## demo.launch
왼쪽 아래 표시되는 `MotionPlanning` 플러그인의 `Context` 탭에서 Planning Library를 선택합니다. 저는 가장 많이 사용된다고 하는 `RRTConnect`를 사용하였습니다.

![RRTConnect](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FNjpIM%2FbtqGH2jxojm%2Fk2khI4F6dv07BZIrOkw7vk%2Fimg.png)

다음으로 로봇 팔에 표시되는 화살표와 원(=인터랙티브 마커) 움직여 로봇 팔을 원하는 형태로 만들어봅시다. 화살표는 좌표 이동, 원은 회전을 담당합니다.

![포즈1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FKnCpL%2FbtqGHrjxBdP%2F2ni8MHlpeKMIuDsKlxLkR1%2Fimg.png)

`MotionPlanning` 플러그인의 `Planning` 탭에서 `Plan` 버튼을 누르면 로봇 팔이 기존 위치에서 새 위치로 부드럽게 이동합니다. `Plan` 버튼을 누르면 `Execute` 버튼이 활성화됩니다. `Execute` 버튼을 누르면 마찬가지로 로봇 팔이 기존 위치에서 새 위치로 이동합니다.

![Plan](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcQLRtD%2FbtqGHrDRQTp%2FbKNumPKC31INA8JqUBCAGK%2Fimg.png)

![Execute](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F5j3rV%2FbtqGQcyqeC5%2FkdF03Gco94g2SjG3kSWMoK%2Fimg.png)

## open_manipulator_gazebo.launch
이번에는 다른 예제를 구동해볼까요?

```sh
roslaunch open_manipulator_gazebo open_manipulator_gazebo.launch
```

`Gazebo`가 실행되면 왼쪽 아래 play 버튼을 누릅니다. `rostopic list`를 입력한 결과는 다음과 같습니다.

![gazebo](https://emanual.robotis.com/assets/images/platform/openmanipulator_x/OpenManipulator_Chain_gazebo_1.png)

```sh
/clock
/gazebo/link_states
/gazebo/model_states
/gazebo/parameter_descriptions
/gazebo/parameter_updates
/gazebo/set_link_state
/gazebo/set_model_state
/open_manipulator/gripper_position/command
/open_manipulator/gripper_sub_position/command
/open_manipulator/joint1_position/command
/open_manipulator/joint2_position/command
/open_manipulator/joint3_position/command
/open_manipulator/joint4_position/command
/open_manipulator/joint_states
/rosout
/rosout_agg
```

다음으로 `MoveIt!`을 연동하겠습니다.

```sh
roslaunch open_manipulator_controller open_manipulator_controller.launch use_platform:=false use_moveit:=true
```

아까와 마찬가지로 `MoveIt!`에서 `RRTConnect` 선택 후 인터랙티브 마커를 이용하여 포즈를 변경합니다. `Plan and Execute`를 실행하면!! `Gazebo` 상에서도 로봇이 이동하는 걸 볼 수 있습니다.

