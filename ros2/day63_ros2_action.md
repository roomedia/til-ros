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

# 액션
액션은 그림과 같이 Noda A - Node B처럼 비동기 + 동기식 양방향 메시지 송수신 방식으로 액션 Goal을 지정하는 Action Client와 액션 목표를 받아 특정 태스크를 수행하면서 중간 결과값에 해당하는 액션 Feedback과 최종 결과값에 해당하는 액션 Result를 전송하는 Action Server 간의 통신이라고 볼 수 있습니다.

![액션 노드](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MzFfMTQ5/MDAxNTk4ODMzNTE5NDgx.EfDFqEiC7yNX9CsuOlUmbwslAIE1ckGVcxNVF2QVsaIg.fk-7Y-AENQATJPNCjM0dIU4DC9oaFPkh6grYr6W9B1Qg.PNG/Selection_050.png?type=w1600)

액션은 토픽과 서비스의 결합이라고 볼 수 있는데 ROS 1의 액션이 토픽으로만 이루어진 것과 달리, ROS 2의 Action Client는 Service Client 3개와 Topic Subscriber 2개로 구성되어 있으며, Action Server는 Service Server 3개와 Topic Publisher 2개로 구성됩니다. 이렇게 복잡하게 구성된 이유는, 이후 설명할 cancel_goal 등의 기능을 추가하기 위해 구성이 달라진 것입니다.

![액션, 피드백, 결과](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MzFfMTk0/MDAxNTk4ODQyNTQzMzU2.m5EI-vAFEqUp1RCH3N6AXQLQ4qtuEdeK5xoDI8dBFkog.l4uHomJU0HUHJCWRV2Cz07dyR3FIi-b8kLbmZo7remcg.PNG/Selection_060.png?type=w1600)

액션에 사용되는 데이터 타입은 msg 및 srv 인터페이스의 변형으로 action 인터페이스라고 합니다. 아래 명령어를 사용하여 액션 인터페이스를 확인할 수 있으며, 위에서부터 순서대로 액션 목표, 결과, 피드백을 나타냅니다.

```sh
$ ros2 interface show turtlesim/action/RotateAbsolute.action
# The desired heading in radians
float32 theta
---
# The angular displacement in radians to the starting position
float32 delta
---
# The remaining rotation in radians
float32 remaining
```

ROS 1에서 토픽만으로 액션을 구성하였을 때에는 동기식 방식을 사용하게 되지만, ROS 2 액션은 토픽과 서비스로 이루어져 있기 때문에, 서버에 목표 전달(send_goal), 목표 취소(cancel_goal), 결과 받기(get_result)를 요청하여 새로운 기능을 이용할 수 있습니다.

비동기 방식의 약점은 실행 시점을 특정하기 어렵다는 것입니다. 이를 보완하기 위해 구현된 것이 ROS 2의 목표 상태(goal_state)입니다. 목표 상태는 목표 값의 상태 머신을 구동하여 액션의 프로세스를 쫒습니다. Goal State Machine은 다음과 같이 구성되어 있으며, 액션 목표 전달 이후 액션의 상태 값을 액션 클라이언트에게 전달할 수 있어 비동기/동기 액션의 처리를 원활하게 할 수 있습니다.

![Goal State Machine](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MzFfMTQ2/MDAxNTk4ODQzNDcwNTAz.l36V2Sm8tSlwZrrXriD-7-vSKuNq5gwk51hzYcuEfpgg.z6lfG25uL-PtWSHG_jV31JAGAvA4WNLCQZXDgUhY4pMg.PNG/goal_state_machine.png?type=w1600)

# 액션 서버 및 클라이언트
액션 테스트를 위해 turtlesim을 실행해봅시다. 아래와 같이 두 개의 노드를 실행합니다.

```
ros2 run turtlesim turtlesim_node
```

```
ros2 run turtlesim turtle_teleop_key
Reading from keyboard
---------------------------
Use arrow keys to move the turtle.
Use G|B|V|C|D|E|R|T keys to rotate to absolute orientations. 'F' to cancel a rotation.
'Q' to quit.
```

화면에 표시된 것처럼 G|B|V|C|D|E|R|T 중 한 키를 누르면 거북이가 해당 방향으로 고개를 돌립니다.

![방향](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MzFfMTc2/MDAxNTk4ODMzNTYxMTQz.j-WSi5diuX9jm5ImBmQtRirhxOtBfWQ1jEtRRvjuBFsg.7iKoPn82Nl9wiDP9qHqYDe5NgS3hmmj3mcteUBYr8G4g.PNG/Selection_058.png?type=w1600)

액션 목표를 취소/완료하면 turtlesim_node의 터미널 창에도 표시가 됩니다. 액션 목표를 취소하지 않고 목표 theta 값에 도달하면 다음과 같이 표시됩니다.

```sh
[INFO]: Rotation goal completed successfully
```

F 키를 눌러 액션 목표를 취소하면 거북이는 다음과 같은 내용을 출력하며 제자리에 멈추게 됩니다.

```sh
[INFO]: Rotation goal canceled
```

# 노드 정보
노드의 액션 정보는 `ros2 node info` 명령어를 사용합니다. `/turtlesim` 노드는 `turtle1/rotate_absolute` 액션의 액션 서버 역할을 하며, `turtlesim/action/RotateAbsolute`가 해당 액션의 인터페이스로 사용되고 있는 걸 볼 수 있습니다.

```sh
$ ros2 node info /turtlesim
/turtlesim
(생략)
  Action Servers:
    /turtle1/rotate_absolute: turtlesim/action/RotateAbsolute
  Action Clients:
```

`/teleop_turtle` 노드는 `/turtle1/rotate_absolute`를 요청하는 액션 클라이언트 역할을 합니다.

```sh
$ ros2 node info /teleop_turtle
/teleop_turtle
(생략)
  Action Servers:

  Action Clients:
    /turtle1/rotate_absolute: turtlesim/action/RotateAbsolute
```

# 액션 목록 (ros2 action list -t)
액션 정보는 위와 같이 특정 노드의 정보 확인을 통해 알아낼 수도 있지만, 토픽, 서비스와 마찬가지로 모든 액션의 목록을 확인할 수도 있습니다. 액션 목록 `ros2 action list` 명령어에 `-t` 옵션을 이용하면 실행 중인 모든 액션의 목록의 형태도 확인할 수 있습니다.

```
$ ros2 action list -t
/turtle1/rotate_absolute [turtlesim/action/RotateAbsolute]
```

# 액션 정보 (ros2 action info)
검색된 액션의 더 자세한 정보를 확인하기 위해서는 `ros2 action info` 명령어를 이용합니다. 해당 명령어의 실행 결과로 액션의 이름과 해당 액션의 서버/클라이언트 노드의 이름, 갯수를 표시합니다.

```
$ ros2 action info /turtle1/rotate_absolute
Action: /turtle1/rotate_absolute
Action clients: 1
    /teleop_turtle
Action servers: 1
    /turtlesim
```

# 액션 목표 (action goal) 전달
`ros2 action send_goal <action_name> <action_type> "<values>"` 형태의 명령어를 사용하여 액션 목표를 전달할 수 있습니다. 다음과 같이 명령어를 입력하면 거북이는 12시 방향을 향하게되며, 목표 값과 UID, 변위 값 delta가 결과로 표시됩니다.

```sh
$ ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "{theta: 1.5708}"
Waiting for an action server to become available...
Sending goal:
     theta: 1.5708

Goal accepted with ID: b991078e96324fc994752b01bc896f49

Result:
    delta: -1.5520002841949463

Goal finished with status: SUCCEEDED
```

![send_goal list](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MzFfMjYx/MDAxNTk4ODMzNTc5ODAz.ZAZy9DY_WXPc89RJ7VnrUTV3tL7C-wi3QMIuD-F_5Log.rSAir8-vVPx2vxZYmnTgFE9TLHtoL0YNmRBq2vNe8nAg.PNG/TurtleSim_056.png?type=w1600)

피드백을 화면에 표시하기 위해서는 --feedback 옵션을 덧붙입니다. 거북이가 회전하며 발생하는, 남은 회전량을 추가로 표시합니다.

```sh
$ ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "{theta: -1.5708}" --feedback
Waiting for an action server to become available...
Sending goal:
     theta: -1.5708

Goal accepted with ID: ad7dc695c07c499782d3ad024fa0f3d2

Feedback:
    remaining: -3.127622127532959

Feedback:
    remaining: -3.111621856689453

Feedback:
    remaining: -3.0956220626831055

(생략)

Feedback:
    remaining: -0.03962135314941406

Feedback:
    remaining: -0.023621320724487305

Feedback:
    remaining: -0.007621288299560547

Result:
    delta: 3.1040005683898926

Goal finished with status: SUCCEEDED
```

![회전 모습](https://cafeptthumb-phinf.pstatic.net/MjAyMDA4MzFfMjA2/MDAxNTk4ODMzNTkxMzA3.T6NsdWWb6Zil9Dmn3rqXc_ghNJx3u95fVvV61tMuB-wg.qbTaZg2CWg0V-I-dQzM7BinipLRM4phbznlLF0vV5O4g.PNG/TurtleSim_057.png?type=w1600)

# 액션 인터페이스 (action interface, action)
액션 인터페이스는 .action 파일 포맷을 가지며, 서비스와 마찬가지로 목표와 결과, 피드백이 --- 라인으로 구분되어 있습니다. 위에서 확인했던 RotateAbsolute.action 인터페이스를 다시 확인해봅시다. `ros2 interface show` 명령어를 이용하여 확인할 수 있습니다.

```sh
$ ros2 interface show turtlesim/action/RotateAbsolute.action
# The desired heading in radians
float32 theta
---
# The angular displacement in radians to the starting position
float32 delta
---
# The remaining rotation in radians
float32 remaining
```

theta는 액션 목표, delta는 액션 결과, remaining는 액션 피드백에 해당합니다. 각 데이터는 각도의 SI 단위인 라디안(radian)을 사용한다.