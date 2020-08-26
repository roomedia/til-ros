# 오로카 화요일 온라인 마곡모임
아래 게시글은 오로카에서 이루어지고 있는 ROS 2 강의 게시글과 화요모임에서 표윤석 박사님이 말씀해주신 것을 토대로 정리한 게시글입니다.

오로카: https://cafe.naver.com/openrt
강의 원글: https://cafe.naver.com/openrt/24101
오로카 ROS 유저모임 오픈채팅 방: https://open.kakao.com/o/gAeOb2nc

# 토픽
기본적으로 노드 A와 노드 B 사이에 토픽을 주고 받고 있습니다.

![토픽](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjVfMTAy/MDAxNTk4MzA4ODU5OTY2.19m3XmUnusf0wx1V7B4KpuwMigAhqYYlnF7yPbseZeMg.zN58KraIH6PgU9DxSSmJCC-8zemku80NPueCZXrdPxUg.PNG/Selection_011.png?type=w1600)

이는 다음과 같은 복잡한 그림으로도 표현할 수 있습니다.

![복잡한 토픽](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjVfMjc4/MDAxNTk4MzA4ODU5OTc0.NSX6wFhcQGOEwDq0PBIh0tLYODONL9hI6Fe2vtbsAH0g.4brgjXjrULB4xxFAty_m-4_KZfENX5OI7IgLnZWFTqIg.PNG/Selection_012.png?type=w1600)

해당 그림에서 보여지듯, 하나의 노드는 퍼블리시와 동시에 서브스크라이브를 할 수 있습니다. 단순히 A란 노드에서 B, C, D 체인처럼 연결되는 게 아니라 다자 간 통신이 가능한 복잡한 구성이 가능합니다.

# 토픽 목록 확인
토픽에 대해 하나씩 알아가봅시다. 노드가 사용하는 토픽의 목록을 알아보기 위해 먼저 노드를 실행합니다.

```
ros2 run turtlesim turtlesim_node
```

![터틀심](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjVfMjIz/MDAxNTk4MzA5MjA2NjY1.IAj7DbHNunHP7n15nFOc23giNaBXnwvZRxK9Xmp_jmEg.PYhbwTk5zYVP0wZAEE9lOdEUnV59fZVZRo0nS66rgnog.PNG/TurtleSim_008.png?type=w1600)

다음과 같이 노드가 실행됩니다. 현재 발행되고 있는 토픽 목록에 대해 알아볼까요? 노드 정보에서 서비스, 액션, 파라미터를 제거하고 본다면 다음과 같습니다.

```
ros2 node info /turtlesim
/turtlesim
  Subscribers:
    /turtle1/cmd_vel: geometry_msgs/msg/Twist
    (생략)
  Publishers:
    /turtle1/color_sensor: turtlesim/msg/Color
    /turtle1/pose: turtlesim/msg/Pose
    (생략)
  Services:
    /clear: std_srvs/srv/Empty
    /kill: turtlesim/srv/Kill
    (생략)
```

좀더 간단하게 확인하는 명령어는 다음과 같습니다. 위 명령어가 turtlesim 노드에 대한 정보만 출력했다면, 아래 명령어는 현재 동작 중인 모든 노드의 토픽 정보를 보여줍니다. -t 옵션은 부가적인 것으로 각 메시지의 타입을 `[turtlesim/msg/Pose]`와 같은 식으로 토픽 이름 옆에 출력해줍니다.

```
ros2 topic list -t
/parameter_events [rcl_interfaces/msg/ParameterEvent]
/rosout [rcl_interfaces/msg/Log]
/turtle1/cmd_vel [geometry_msgs/msg/Twist]
/turtle1/color_sensor [turtlesim/msg/Color]
/turtle1/pose [turtlesim/msg/Pose]
```

현재 토픽은 발행되기만 하고 수신하는 노드가 없기 때문에, `rqt_graph`를 실행해보면 다음과 같이 어디에도 연결되지 않았음을 확인할 수 있습니다.

```
rqt_graph
```

![rqt_graph](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjVfMzEg/MDAxNTk4MzA5MjQ0ODEz.OLgB1-XVGh7YLfZXrE-4ZKMzVy2N-e5j5oaiYTYK3csg.e1IchAn29mTpVYeXswzLUZZX17p23o7OZOCGSkrClg0g.PNG/rqt_graph__RosGraph_-_rqt_013.png?type=w1600)

추가로 `turtle_teleop_key`를 실행해봅시다. 이후 `rqt_graph`에서 새로고침을 누르면 그래프 내용이 갱신됩니다.

```
ros2 run turtlesim turtle_teleop_key
```

![refresh](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjVfMTgg/MDAxNTk4MzA5MjY4MjYy.-lsFQ8olPRZrFLDTF9rBe7mCeKlfnP--_7uuW8RVJowg.YKPt1wxiPfSaK24OgyctCz4enAF41XflN-qW3bLTU_sg.PNG/rqt_graph__RosGraph_-_rqt_013-1.png?type=w1600)

![refreshed](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjVfMTg0/MDAxNTk4MzA5MzAwODYz.YwZ8FnaGhRH375PG3jMkURpa6gO394XugndnFNiyusMg.hRHRolG_ImYHwYcrB3XWYf5mhF9f9VKqdcWq-yVEknIg.PNG/rqt_graph__RosGraph_-_rqt_014.png?type=w1600)

그런데 node info에서는 `color_sensor`, `pose`와 같은 토픽도 있었는데 여기서는 보이지 않습니다. 이는 해당 토픽을 수신하는 노드가 없어 비활성화 된 상태이기 때문입니다. `rqt_graph`에서 다음과 같이 옵션을 제거하게 되면, 활성화되지 않은 토픽을 확인할 수 있습니다. 해당 토픽들은 다른 노드에 수신되지 않아 비활성화 된 걸 알 수 있습니다.

![active](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjVfOCAg/MDAxNTk4MzA5MzQxNTE3.nzrgZ7KUN1K0mJtTy3l0_BHXDXlD-uoT1UjJeZaJtZEg.vzF94vGHCpd73HhS5QvrukrtE6Vyj6tkMTWheQF-hP8g.PNG/rqt_graph__RosGraph_-_rqt_015.png?type=w1600)

# 토픽 정보 확인
`rqt_graph`를 사용하지 않고 CLI 환경에서 토픽 정보를 확인해봅시다.

```
ros2 topic info /turtle1/cmd_vel
Type: geometry_msgs/msg/Twist
Publisher count: 1
Subscriber count: 1
```

토픽 정보를 세세하게 확인할 수 있습니다.

# 토픽 내용 확인
아래 명령어를 통해 현재 토픽의 값을 수신하여 출력할 수 있습니다.

```
ros2 topic echo /turtle1/cmd_vel
linear:
x: 1.0
y: 0.0
z: 0.0
angular:
x: 0.0
y: 0.0
z: 0.0
```

# 토픽 대역폭 확인
이번에는 송수신하는 메시지의 크기를 확인해봅시다. 크기 확인은 아래 명령어로 가능하며, 토픽의 초당 대역폭을 알 수 있습니다. 초당 대역폭은 사용하는 메시지의 형태 및 주기에 따라 달라질 수 있습니다.

```
ros2 topic bw /turtle1/cmd_vel
Subscribed to [/turtle1/cmd_vel]
average: 1.74KB/s
mean: 0.05KB min: 0.05KB max: 0.05KB window: 100
(생략)
```

# 토픽 주기 확인
토픽의 전송 주기를 확인하기 위한 명령어입니다. 지속적으로 `/turtle1/cmd_vel`을 발행한다면 평균 33.2 HZ 정도가 나옵니다. 즉, 0.03 초에 한 번 토픽을 발행한다는 소리입니다. 이는 설정에 따라 달라질 수 있습니다.

```
ros2 topic hz /turtle1/cmd_vel
average rate: 33.212
min: 0.029s max: 0.089s std dev: 0.00126s window: 2483
(생략)
```

# 토픽 지연 시간 확인
토픽은 RMW 및 네트워크 장비를 거치기 때문에 Latency가 반드시 존재합니다. 이 지연 시간을 체크하기 위해 다음 명령어를 사용합니다. 주의해야 할 점은, 메시지가 header를 포함하여야 한다는 점입니다.

```
ros2 topic delay /TOPIC_NAME
average delay: xxx.xxx
min: xxx.xxxs max: xxx.xxxs std dev: xxx.xxxs window: 10
```

헤더 안에는 발행 시간을 기록하는 stamp 메시지가 존재하는데, 발행 시간과 수신 시간의 차이를 통해 latency를 확인할 수 있습니다. 이는 ROS 1에는 없는 ROS 2의 고유 기능입니다.

# 토픽 발행
`ros2 topic pub <topic name> <msg_type> "<args>"` 형태를 사용하여 간단하게 토픽을 발행할 수 있습니다. 이때 `--once`로 실행하면 한 번만, `--rate 1`과 같은 옵션을 붙여 실행하면 1 hz 주기로 반복 가능합니다. 아래 명령어는 `/turtle1/cmd_vel` 토픽을 한 번 발행하며, 해당 토픽의 타입은 `geometry_msgs/msg/Twist`이며 값은 "" 안의 값과 같습니다.

```
ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```

![실행 모습](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjVfNjQg/MDAxNTk4MzA5NDMxMDAz.taLRrHPmINYWsy92Mpqz0GoU7zMDZCA2AauIsB0D_lcg.FhgvIOLn2x-oZPUuRRliyAHZsGNr1O-D0BHAp4pbnWcg.PNG/TurtleSim_022.png?type=w1600)

`--once`를 `--rate 1`로 변경한 모습입니다.

```
ros2 topic pub --rate 1 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```

![실행 모습](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjVfMiAg/MDAxNTk4MzA5NDU1MDc4.fCKyY1lYjBeTVQIb-tCt0Uct6GQGtA7EH-lghA6R4wwg.91FX1pCnNm197cVdY3RMtJoJJBXnCkAbJCVAjrcr89Ig.GIF/ezgif-6-78fccd7b23e1.gif?type=w1600)

# ROS 인터페이스
ROS2에서 사용하는 ROS 인터페이스란 msg, srv, action 세가지로 나눌 수 있습니다. 인터페이스는 노드 간 데이터를 주고받을 때 사용하는 데이터 형태로, 인터페이스 타입은 단순 자료형, 간단한 데이터 구조, 배열과 같은 구조가 있습니다.

원본 게시글 참고자료 7번을 보면 msg, srv, action 설명을 더 자세히 확인할 수 있습니다.
https://index.ros.org/doc/ros2/Concepts/About-ROS-Interfaces/

# 기타 QnA
- Q: 갑자기 생긴 ROS 2 Rolling 버전은 무엇인가요?
- A: 배포판이 아닌, 지속적으로 유지되는 개발용 버전입니다.
- Q: 어떤 과정으로 ROS 마이너 업데이트를 패치할 수 있나요?
- A: 패키지 매니저를 통해 설치하셨다면, 패키지 매니저 업데이트를 통해 ROS를 최신 업데이트로 유지할 수 있습니다.

```
sudo apt-get update
sudo apt-get dist-upgrade
```

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