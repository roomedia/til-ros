# 서론
위 강의는 네이버 카페 오로카에 게시된 표윤석님의 ROS 2 강의를 이해하면서, 제 식대로 재구성한 게시글입니다.

서비스: https://cafe.naver.com/openrt/24128
액션: https://cafe.naver.com/openrt/24142

추후 강의로는 

1. SI 단위 작성
1. 좌표계 맞추기
1. 코딩 스타일 가이드
1. ament_lint와 같은 lint 계열
1. 토픽, 서비스, 액션 파라미터에 대한 기본적인 프로그램
1. rqt gui 작성

같은 강의가 이루어질 예정입니다. 오픈채팅 방에 참가하시고, 매주 화요일 7시 반에 ROS 2 강의에 참석하세요!!

- 오로카: https://cafe.naver.com/openrt
- 오로카 ROS 유저모임 오픈채팅 방: https://open.kakao.com/o/gAeOb2nc

# 서비스
서비스는 동기식 양방향 메시지 송수신 방식이라고 말씀 드렸었죠. 그림과 같이 서비스를 요청하는 쪽은 서비스 클라이언트, 서비스 수행 이후 응답하는 쪽은 서비스 서버라고 불립니다. 서비스 요청 및 응답에 사용되는 ros 인터페이스를 srv 인터페이스라고 합니다.

토픽과 비교했을 때 토픽은 센서라던지, 로봇의 속도 전달, 좌표 전달에 사용되며, 
로봇의 특정 관절 각도 요청 -> 단답형 응답 등의 데이터는 서비스를 이용하게 됩니다. 토픽에 비해 사용 빈도는 적지만 여전히 중요합니다.

![클라이언트 서버 그림](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjdfNTEg/MDAxNTk4NDg0NDIzODcx.bd-pW-erigaVrfu09zjNGXFZz4JuvCwpIMjfn5z5kXkg.HKE0co31jefxc4cnuthcL8ZQiVdJAfPay555K3QpzUYg.PNG/Selection_042.png?type=w1600)

서비스는 하나의 서비스 서버가 여러 개의 클라이언트를 가질 수 있습니다. 단, 서비스 응답은 서비스를 요청한 클라이언트에게만 전달됩니다. 1대N이라기 보단, 1대1이 여러 개 있다고 이해하시면 될 것 같습니다.

![여러 서비스 클라이언트](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjdfMzAw/MDAxNTk4NDg0NDIzODg0.gqwyOK5if11dGzhUbMZ3vR6FJDEUREVjBCthGSB2rVEg.k6XgPF2fQT8TYcaiFQD-uhnFTxBALBb1g5EKkQd-cokg.PNG/Selection_043.png?type=w1600)

# 서비스 목록 확인
서비스 목록 확인을 위해 예제 노드를 실행합니다.

```sh
ros2 run turtlesim turtlesim_node
```

다음 명령어를 통해 현재 실행 중인 서비스 목록을 확인할 수 있습니다.

```sh
ros2 service list
/clear
/kill
/reset
/spawn
/turtle1/set_pen
/turtle1/teleport_absolute
/turtle1/teleport_relative
/turtlesim/describe_parameters
/turtlesim/get_parameter_types
/turtlesim/get_parameters
/turtlesim/list_parameters
/turtlesim/set_parameters
/turtlesim/set_parameters_atomically
```

이름에 parameters가 들어간 서비스는 파라미터와 관련된 내용으로, 모든 노드에 기본으로 포함되는 서비스라고 합니다. 이에 대한 부분은 파라미터에서 다루겠습니다.

```sh
/clear
/kill
/reset
/spawn
/turtle1/set_pen
/turtle1/teleport_absolute
/turtle1/teleport_relative
```

# 서비스 형태 확인
특정 서비스의 형태를 확인해봅시다.

```sh
ros2 service type /clear
std_srvs/srv/Empty
```

```sh
ros2 service type /kill
turtlesim/srv/Kill
```

```sh
ros2 service type /spawn
turtlesim/srv/Spawn
```

이전에 사용하였던 `ros2 service list` 명령어에  `-t` 옵션을 붙이면 서비스 목록과 함께 각각의 서비스의 형태를 확인할 수 있습니다.

```sh
ros2 service list -t
/clear [std_srvs/srv/Empty]
/kill [turtlesim/srv/Kill]
/reset [std_srvs/srv/Empty]
/spawn [turtlesim/srv/Spawn]
/turtle1/set_pen [turtlesim/srv/SetPen]
/turtle1/teleport_absolute [turtlesim/srv/TeleportAbsolute]
/turtle1/teleport_relative [turtlesim/srv/TeleportRelative]
```

# 서비스 요청
이전에 토픽을 보내고 받을 때 `ros2 topic` 명령어를 사용한 것처럼, 이번에는 CLI 환경에서 서비스를 요청해봅시다.

```sh
ros2 service call <service_name> <service_type> "<arguments>"
```

해당 명령어는 위와 같이 사용하며, 활용을 위해 터틀심의 이동 궤적을 지워봅시다. 이를 위해 먼저 터틀심 이동 노드를 실행합니다.

```sh
ros2 run turtlesim turtle_teleop_key
```

![turtlesim](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjdfODUg/MDAxNTk4NDg0OTM1NDc5.be9st1qIXxixBdLYwv2TVGx8WAhQ2ixFk1zaIqqSF5Ag.iDQbcP8Qtk_FiFD3YKHyu_yJPMxv6tQMdm1JdCyLrUMg.PNG/TurtleSim_032.png?type=w1600)

`/clear` 서비스를 요청하여 이동 궤적을 삭제할 수 있습니다. `/clear` 서비스는 아까 확인한 것처럼 `std_srvs/srv/Empty` 형태이기 때문에 `"<arguments>"`를 생략할 수 있습니다.

```sh
ros2 service call /clear std_srvs/srv/Empty
waiting for service to become available...
requester: making request: std_srvs.srv.Empty_Request()

response:
std_srvs.srv.Empty_Response()

```

![cleared](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjdfNzMg/MDAxNTk4NDg0OTU2NDM1.2BCI6Q-uRKqUIfRVeP938p29Dor7zW2PXnLocPvaf3Ig.Iryul_cYgsgosmyJ8zyMGh6gJZVcAMPDWEIeWIkxU20g.PNG/TurtleSim_033.png?type=w1600)

`/kill`을 통해 거북이를 삭제할 수 있습니다ㅠㅠ 삭제를 위해서는 거북이의 이름을 `"<arguments>"` 형태로 입력하여야 합니다.

```sh
ros2 service call /kill turtlesim/srv/Kill "name: 'turtle1'"
waiting for service to become available...
requester: making request: turtlesim.srv.Kill_Request(name='turtle1')

response:
turtlesim.srv.Kill_Response()
```

![killed](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjdfMTMg/MDAxNTk4NDg0OTY3Njc3.pTBN85GsAxr-CZpoOlQFMjzYr5g629YZSrQ3zx9YD0Qg.mzrrU4PWp6TDFV8w2edKH2xSRLAWhZoP93iC9NRx5SEg.PNG/TurtleSim_034.png?type=w1600)

`/reset` 서비스로 거북이가 다시 돌아옵니다ㅎㅎ 궤적과 위치, 포즈는 모두 재설정됩니다.

```sh
ros2 service call /reset std_srvs/srv/Empty
requester: making request: std_srvs.srv.Empty_Request()

response:
std_srvs.srv.Empty_Response()

```

![come back](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjdfMjUg/MDAxNTk4NDg0OTgwMDQ2.eB26iplsDBkB398to5DsBmapbTOk9RbuNLJ4EWxmabkg.ERqkquYU56AYm7cEs9QnawFGRTRFDzccuSMFBUju8NYg.PNG/TurtleSim_035.png?type=w1600)

`/set_pen` 서비스는 궤적의 색과 크기를 변경할 수 있습니다. 궤적의 색은 r, g, b로 결정되며, 굵기는 width로 지정합니다.

```sh
ros2 service call /turtle1/set_pen turtlesim/srv/SetPen "{r: 255, g: 255, b: 255, width: 10}"
requester: making request: turtlesim.srv.SetPen_Request(r=255, g=255, b=255, width=10, off=0)
response:
turtlesim.srv.SetPen_Response()
```

![pen](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjdfMjgx/MDAxNTk4NDg0OTkyNDYz.JEnlmOIkCRamM0AvQ8JE3LPrPcBtnopuB9RAB7iEZwYg.uiFlsn_yYRQgZOysYILz26NV6gwQzsKX317d82klZDUg.PNG/TurtleSim_041.png?type=w1600)

`/spawn` 서비스를 요청하면 지정한 위치 및 자세로 지정한 이름의 거북이를 추가할 수 있습니다. 이름을 옵션으로 지정하지 않으면 turtle2, 3, 4, ... 하나씩 증가하며, 같은 이름은 사용할 수 없습니다.

```
$ ros2 service call /kill turtlesim/srv/Kill "name: 'turtle1'"
requester: making request: turtlesim.srv.Kill_Request(name='turtle1')
response:
turtlesim.srv.Kill_Response()

$ ros2 service call /spawn turtlesim/srv/Spawn "{x: 5.5, y: 9, theta: 1.57, name: 'leonardo'}"
requester: making request: turtlesim.srv.Spawn_Request(x=5.5, y=9.0, theta=1.57, name='leonardo')
response:
turtlesim.srv.Spawn_Response(name='leonardo')

$ ros2 service call /spawn turtlesim/srv/Spawn "{x: 5.5, y: 7, theta: 1.57, name: 'raffaello'}"
requester: making request: turtlesim.srv.Spawn_Request(x=5.5, y=7.0, theta=1.57, name='raffaello')
response:
turtlesim.srv.Spawn_Response(name='raffaello')

$ ros2 service call /spawn turtlesim/srv/Spawn "{x: 5.5, y: 5, theta: 1.57, name: 'michelangelo'}"
requester: making request: turtlesim.srv.Spawn_Request(x=5.5, y=5.0, theta=1.57, name='michelangelo')
response:
turtlesim.srv.Spawn_Response(name='michelangelo')

$ ros2 service call /spawn turtlesim/srv/Spawn "{x: 5.5, y: 3, theta: 1.57, name: 'donatello'}"
requester: making request: turtlesim.srv.Spawn_Request(x=5.5, y=3.0, theta=1.57, name='donatello')
response:
turtlesim.srv.Spawn_Response(name='donatello')
```

![turtle](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MjdfMjI5/MDAxNTk4NDg1MDA5MTUy.WGewW36EJqf9DHn5X9sGqZI1nGqd7Jdnk1G8fDJvp7sg.202NbAqG1p7zM0EyyZndstNa-sdOGWTB5auwTlQ_PqIg.PNG/TurtleSim_037.png?type=w1600)

```sh
$ ros2 topic list
/donatello/cmd_vel
/donatello/color_sensor
/donatello/pose
/leonardo/cmd_vel
/leonardo/color_sensor
/leonardo/pose
/michelangelo/cmd_vel
/michelangelo/color_sensor
/michelangelo/pose
/new_turtle/cmd_vel
/new_turtle/pose
/parameter_events
/raffaello/cmd_vel
/raffaello/color_sensor
/raffaello/pose
/rosout
/turtle1/cmd_vel
/turtle1/color_sensor
/turtle1/pose
/turtle2/cmd_vel
/turtle2/pose
/turtle3/cmd_vel
/turtle3/color_sensor
/turtle4/cmd_vel
/turtle4/color_sensor
/turtle4/pose
```

# 서비스 인터페이스
`/spawn` 서비스의 인터페이스는 어떻게 생겼을까요? `/spawn`의 서비스 타입은 `turtlesim/srv/Spawn.srv`입니다. `ros2 interface show` 명령어로 해당 타입의 형태를 확인해봅시다.

```sh
$ ros2 interface show turtlesim/srv/Spawn.srv
float32 x
float32 y
float32 theta
string name
---
string name
```

아까 거북이를 생성할 때 표시되었던 내용과 동일합니다. 이름, x, y, theta를 입력하고, 이름을 반환했습니다.

```
requester: making request: turtlesim.srv.Spawn_Request(x=5.5, y=7.0, theta=1.57, name='raffaello')
response:
turtlesim.srv.Spawn_Response(name='raffaello')
```