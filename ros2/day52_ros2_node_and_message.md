# 오로카 화요일 온라인 마곡모임
아래 게시글은 오로카에서 이루어지고 있는 ROS 2 강의 게시글과 화요모임에서 표윤석 박사님이 말씀해주신 것을 토대로 정리한 게시글입니다.

오로카: https://cafe.naver.com/openrt
강의 원글: https://cafe.naver.com/openrt/24086
오로카 ROS 유저모임 오픈채팅 방: https://open.kakao.com/o/gAeOb2nc

# ROS2 노드와 메시지 통신
노드는 최소 단위 실행 가능한 프로세스, 즉, 하나의 프로그램을 이야기합니다. ROS에서는 재사용성을 높이기 위해 최소한의 실행 단위(노드 단위)로 프로그램을 나누어 작업하게 됩니다.

노드는 핵심 기능 별로 잘게 나누어져 작성되지만, 한 노드에서 작업한 데이터를 다른 노드가 받아 처리할 수 있도록, 유기적으로 연결되어 있어야 합니다. 오늘은 이러한 노드 간 연결 방식(토픽, 서비스, 액션, 파라미터)에 대해 알아보도록 하겠습니다. 사실 ROS 1과는 크게 차이가 없어보이네요^^;

![그림1: 노드 간 통신](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjFfMTAx/MDAxNTk3OTY0NzAwMTY4.JtPqCFO48n6oxBuliUfEcWvaenIrhy-C7oHgnci0whYg.rfpPwjV4wf3vcK-4OfntOfqpDOCaGekbEc0ERL-juBEg.PNG/Selection_002.png?type=w1600)

각각의 노드가 서로 유기적으로 연결되는 통신 방식은 ROS의 가장 특이한 점이자 대표적인 특징입니다. ROS를 이용하여 로봇을 제작하면 많은 노드가 유기적으로 연결되어야 함. 최소한 50개 정도의 노드, 어느 정도 기능을 갖춘 모바일 로봇의 경우 노드의 수가 100개를 넘어간다고 합니다. 카메라 하나를 사용하더라도 적게는 15개, 많게는 20 ~ 30개의 노드가 사용됩니다.

- Q: 노드가 너무 많으면 PC가 버벅거리나요?
- A: 네. 당연히.
- Q: 네트워크에 연결이 되어있긴 하지만, 하나의 컴퓨터에서 노드 간 통신하는 경우에도 네트워크 트래픽이 발생하나요?
- A: 기본적으로는 발생합니다. intra-process communication 설정을 통해 이를 방지할 수 있습니다.

여러가지 통신을 살펴봅시다.

## 토픽
![토픽](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjFfNTgg/MDAxNTk3OTY0NzE1Njk3.UAc5Zanj2JvppsI7O-l084XPR8aD-_jUsREEWBJiGTEg.J1gUTk34A-llO7oWGplNdJVfFw7DYfqX4r5tvb6yRR4g.PNG/Selection_003.png?type=w1600)

토픽은 단방향, 비동기식, 연속성을 가지는 형태의 통신입니다. 토픽은 메시지를 발행하는 노드를 퍼블리셔, 수신하는 노드를 서브스크라이버라고 합니다. 토픽 명에 따라 1대1, 1대N, N대N이던 모두 구현 가능합니다. ROS 1과의 차이는 `roscore`가 사라지면서 IP, 통신 방식을 고려하지 않아도 됩니다. 다 DDS 덕분이죠.

## 서비스
![서비스](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjFfMzgg/MDAxNTk3OTY0NzI4NTY5.PePQzDYHwI5BT0uci36-5WVKlvE16xCnNV5mMmDbTscg.e_QOdb7dEhC37VNyXpPzmzgpaa8ffrVJmkPZUppYkQMg.PNG/Selection_004.png?type=w1600)

서비스는 서비스를 요청하는 쪽을 서비스 클라이언트, 서비스를 받아 무언가 수행 후 결과를 리턴하는 쪽을 서비스 서버라고 합니다. 서비스는 Request를 받아야만 Response를 보내는 동기식이며, 양방향적인 통신 방식입니다.

## 액션
![액션](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjFfMTM4/MDAxNTk3OTY0NzM3Mzk3.n36V-JZnTgFivsb4Ky2IuGK266cLRXU0pzuakELklVgg.k-vedoYChAZ4c_uB-WTXRmcUTmkBmZFEAkRweBc4UhAg.PNG/Selection_005.png?type=w1600)

액션은 Task 레벨의 정보를 주고 받을 때 사용되는 통신 방식입니다. 제일 먼저 액션 꼴을 던지고, A는 무언가를 처리한 후 중간 결과 값(=Feedback)을 순차적으로 보냅니다. 마지막으로 결과(=Result)가 반환됩니다. 1, 3은 서비스와 유사하며, 2번 피드백은 토픽 기능과 매우 유사합니다. 액션은 토픽과 서비스의 결합 형태로 이해할 수 있습니다.

## 파라미터
![파라미터](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjFfMTc1/MDAxNTk3OTY0NzQ3MTc1.eZ0kq_YgOWwwCMMbNiL74EsVBoPozzGwyM-ebVuT8_gg.K0skBFoMl6ofEvTWKiSeKqx_vG_I1iBlKmoZ1Nq7CWog.PNG/Selection_006.png?type=w1600)

ROS 1과는 달리 각각의 노드에 파라미터 서버가 있으며, 각각의 노드의 서버에 접속하여 파라미터를 설정해줘야 합니다. 노드 내외부에서 쉽게 지정하거나 변경할 수 있고, 쉽게 가져올 수 있습니다. 파라미터를 공유하지 않아 충돌할 위험성도 없습니다. ROS 1에서 ROS 2로 넘어오면서 구현 방식은 바뀌었지만 컨셉 자체는 살아있다고 할 수 있습니다.

# 노드 실행
노드를 실행하기 위해서는 다음과 같이 `ros2 run <패키지 이름> <노드 이름>` 명령어를 사용하여야 합니다. 다음 명령어는 터틀심과 이를 조종하기 위한 teleop을 실행합니다.

```
ros2 run turtlesim turtlesim_node
```

```
ros2 run turtlesim turtle_teleop_key
```

![터틀심](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjFfMjA4/MDAxNTk3OTY0ODM0MjE5.uWZ_u4AABfKpETCaXW4PSGG8iFEhUwJ0Ws19UeqOUPog.jofdg_YRcAT_pM1REBM2heLkXw2j-x64YJPJhohwQCwg.PNG/TurtleSim_008.png?type=w1600)

하나의 노드를 실행할 때는 `ros2 run`, 여러 개의 노드를 실행하고 싶을 때에는 먼저 `.launch` 파일에 실행 설정을 한 뒤, 해당 파일을 `ros2 launch`를 통해 불러와 사용합니다. 이외에도 ROS 2 패키지 중 `rqt`, `rqt_graph`, `rviz2` 등은 따로 실행 명령어로 등록되어 있습니다. 다음은 노드 간 메시지 통신을 그래프로 표시하는 `rqt_graph`를 실행한 모습입니다.

```
rqt_graph
```

![rqt_graph](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjFfMTI1/MDAxNTk3OTY0OTc2NzM1.a77bx0Mm8HaxnyGcvRCRGAg7Weu2k34xmvMADUnZD6Yg.paZkOBYGx-llGhFYRXl6eQ66iyOfqYQsxZ3C31tmwCUg.PNG/rqt_graph__RosGraph_-_rqt_011.png?type=w1600)

# 노드 목록
현재 실행 중인 노드의 목록을 보려면 다음 명령어를 사용하면 됩니다. `turtlesim`, `turtle_teleop_key`, `rqt_graph` 총 3개의 노드 목록이 결과값으로 출력됩니다. 노드 파일 명과 실제 노드 이름은 다를 수 있는 것도 알아둡시다. 특히 `rqt_graph`의 경우 `rqt`의 플러그인 중 하나로, 여러 개의 툴이 이름을 겹치지 않고 실행될 수 있도록 노드 이름 뒤에 임의의 번호가 붙게 됩니다.

```
ros2 node list
/rqt_gui_py_node_28168
/teleop_turtle
/turtlesim
```

동일 노드를 여러 개 실행하고 싶다면 `ros2 run turtlesim turtlesim_node`와 같이 그대로 실행해도 되지만 동일한 노드 이름으로 실행되면 시스템 구성이 상당히 복잡해지게 됩니다. 노드 이름을 다르게 하고 싶다면 다음과 같이 실행합니다.

```
ros2 run turtlesim turtlesim_node __node:=new_turtle
```

실행 노드 목록을 다시 확인해보면 노드 이름이 `new_turtle`로 설정된 걸 알 수 있습니다.

```
ros2 node list
/rqt_gui_py_node_29017
/teleop_turtle
/new_turtle
/turtlesim
```

![rqt_graph](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjFfMjQ2/MDAxNTk3OTY1MDAyMjM2.PDU4AVMGGNO9Jmsv2Q9Fw-6QYFb4MskDBTFtYEAm4h8g.EVLlPXFyJMitOiMO_FdtVm4JUOKMEH_eOTBguKIVE30g.PNG/rqt_graph__RosGraph_-_rqt_010.png?type=w1600)

`rqt_graph` 또한 갱신됩니다. 터틀심을 새로 실행할 때 토픽 이름은 변경하지 않았기 때문에 두 개의 노드가 하나의 토픽을 사용하게 됩니다. 즉, `turtle_teleop_key`에서 키를 입력하면 두 개의 터틀심이 동시에 움직이게 됩니다.

# 노드 정보
노드의 정보를 확인하기 위해서는 아래 명령어를 사용합니다. 이를 통해 노드의 Publishers, Subscribers, Services, Actions, Parameters를 확인할 수 있습니다.

```
ros2 node info /turtlesim
/turtlesim
  Subscribers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /turtle1/cmd_vel: geometry_msgs/msg/Twist
  Publishers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /rosout: rcl_interfaces/msg/Log
    /turtle1/color_sensor: turtlesim/msg/Color
    /turtle1/pose: turtlesim/msg/Pose
    /turtle1/rotate_absolute/_action/feedback: turtlesim/action/RotateAbsolute_FeedbackMessage
    /turtle1/rotate_absolute/_action/status: action_msgs/msg/GoalStatusArray
  Services:
    /clear: std_srvs/srv/Empty
    /kill: turtlesim/srv/Kill
    /reset: std_srvs/srv/Empty
    /spawn: turtlesim/srv/Spawn
    /turtle1/rotate_absolute/_action/cancel_goal: action_msgs/srv/CancelGoal
    /turtle1/rotate_absolute/_action/get_result: turtlesim/action/RotateAbsolute_GetResult
    /turtle1/rotate_absolute/_action/send_goal: turtlesim/action/RotateAbsolute_SendGoal
    /turtle1/set_pen: turtlesim/srv/SetPen
    /turtle1/teleport_absolute: turtlesim/srv/TeleportAbsolute
    /turtle1/teleport_relative: turtlesim/srv/TeleportRelative
    /turtlesim/describe_parameters: rcl_interfaces/srv/DescribeParameters
    /turtlesim/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
    /turtlesim/get_parameters: rcl_interfaces/srv/GetParameters
    /turtlesim/list_parameters: rcl_interfaces/srv/ListParameters
    /turtlesim/set_parameters: rcl_interfaces/srv/SetParameters
    /turtlesim/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
```

굉장히 많아보이지만, 파라미터 관련이 대부분이기 때문에 실제 정보는 적습니다.

# 추후 강의 목차
1. SI 단위 작성
1. 좌표계 맞추기
1. 코딩 스타일 가이드
1. ament_lint와 같은 lint 계열
1. 토픽, 서비스, 액션 파라미터에 대한 기본적인 프로그램
1. rqt gui 작성

같은 강의가 이루어질 예정입니다. 오픈채팅 방에 참가하시고, 매주 화요일 7시 반에 ROS 2 강의에 참석하세요!!

오로카: https://cafe.naver.com/openrt
오로카 ROS 유저모임 오픈채팅 방: https://open.kakao.com/o/gAeOb2nc