# ROS 명령어 정리
ROS 명령어는 ros prefix와 개념적으로 유사한 리눅스 명령어를 붙여 사용합니다.
## ROS 쉘 명령어
- roscd: ros + cd, 지정한 ROS 패키지 디렉토리로 이동합니다.
- rosls: ros + ls, ROS 패키지의 파일 목록을 확인합니다.
- rosed: ros + ed, ROS 패키지의 파일을 편집합니다.
- roscp: ros + cp, ROS 패키지의 파일을 복사합니다.
- rospd: ros + pushd, ROS 디렉토리 인덱스에 디렉토리를 추가합니다.
- rosd: ros + directory, ROS 디렉토리 인덱스를 확인합니다.
## ROS 실행 명령어
- roscore: ros + core, 마스터를 실행합니다.
- rosrun: ros + run, 단일 노드를 실행합니다.
- roslaunch: ros + launch, 여러 노드를 실행하며, 실행 옵션을 설정할 수 있습니다.
- rosclean: ros + clean, ROS 로그 파일을 검사하거나 삭제합니다.
## catkin 명령어
ROS2의 빌드 시스템은 catkin에서 colcon으로 대체되었지만, ROS1의 공식 지원이 완전히 종료되지 않았기 때문에 catkin 명령어에 대해 알아보도록 하겠습니다.
- catkin_create_pkg: catkin 빌드 시스템으로 패키지를 자동 생성합니다.
- catkin_make: catkin 패키지를 빌드합니다.
- catkin_eclipse: catkin 패키지를 이클립스에서 사용할 수 있도록 변경합니다.
- catkin_prepare_release: catkin 패키지를 릴리즈 하기 위해 로그를 정리하고 버전 태깅을 진행합니다.
- catkin_generate_changelog: catkin 패키지를 릴리즈할 때 CHANGELOG.rst 파일을 생성 또는 업데이트 합니다.
- catkin_init_workspace: catkin 빌드 시스템 작업 폴더를 초기화합니다.
- catkin_find: catkin 패키지 내에서 검색 작업을 수행합니다.
## rosnode 명령어
- rosnode list: 활성화된 노드 목록을 확인합니다.

`turtlesim_node`와 `turtle_teleop_key`를 실행시킨 상태에서 아래 명령어를 입력합니다.
```
rosnode list
```
다음과 같은 결과가 출력되는 걸 볼 수 있습니다.
```
root@5b055f4ff5ae:/# rosnode list
/rosout
/teleop_turtle
/turtlesim
```
실행 노드의 이름은 `turtlesim_node`, `turtle_teleop_key`지만, 표시되는 이름은 `turtlesim`, `teleop_turtle`입니다. 이는 코드 내부에서 설정한 노드 이름과 실행 노드의 이름이 다르기 때문입니다.
- rosnode ping [node_name]: 지정된 노드와 연결 상태를 테스트합니다.

다음 명령어를 이용하여 `turtlesim` 노드의 연결 상태를 확인합니다.
```
rosnode ping /turtlesim
```
다음과 같이 응답 속도를 주기적으로 표시하며, interrupt가 발생할 시 평균 응답 속도를 출력한 뒤 종료됩니다.
```
root@5b055f4ff5ae:/# rosnode ping /turtlesim
rosnode: node is [/turtlesim]
pinging /turtlesim with a timeout of 3.0s
xmlrpc reply from http://5b055f4ff5ae:43121/	time=19.457340ms
xmlrpc reply from http://5b055f4ff5ae:43121/	time=1.332760ms
xmlrpc reply from http://5b055f4ff5ae:43121/	time=1.564980ms
xmlrpc reply from http://5b055f4ff5ae:43121/	time=1.409769ms
^Cping average: 5.941212ms
```
이때 실행 노드의 이름이 아니라 실제 노드 이름을 입력하여야 합니다. 실행 노드 이름을 사용할 경우, 아래와 같은 에러가 발생합니다.
```
root@5b055f4ff5ae:/# rosnode ping /turtlesim_node
rosnode: node is [/turtlesim_node]
cannot ping [/turtlesim_node]: unknown node
```
- rosnode info [node_name]: 지정된 노드의 정보를 확인합니다.

마찬가지로 `rosnode info` 명령어를 사용하여 마스터에 등록된 노드 정보를 확인합니다.
```
rosnode infp /turtlesim
```
노드 이름, publications, subscriptions, services를 포함하여 다양한 정보가 출력됩니다.
```
root@5b055f4ff5ae:/# rosnode info /turtlesim
--------------------------------------------------------------------------------
Node [/turtlesim]
Publications:
 * /rosout [rosgraph_msgs/Log]
 * /turtle1/color_sensor [turtlesim/Color]
 * /turtle1/pose [turtlesim/Pose]

Subscriptions:
 * /turtle1/cmd_vel [geometry_msgs/Twist]

Services:
 * /clear
 * /kill
 * /reset
 * /spawn
 * /turtle1/set_pen
 * /turtle1/teleport_absolute
 * /turtle1/teleport_relative
 * /turtlesim/get_loggers
 * /turtlesim/set_logger_level


contacting node http://5b055f4ff5ae:43121/ ...
Pid: 130
Connections:
 * topic: /rosout
    * to: /rosout
    * direction: outbound (53519 - 172.17.0.2:52722) [14]
    * transport: TCPROS
 * topic: /turtle1/cmd_vel
    * to: /teleop_turtle (http://5b055f4ff5ae:45033/)
    * direction: inbound (52440 - 5b055f4ff5ae:46757) [17]
    * transport: TCPROS
```
- rosnode machine [PC_name_or_IP]: 해당 PC에서 실행되고 있는 노드 목록을 확인합니다.
- rosnode kill [node_name]: 지정된 노드를 실행 중단합니다.

다음 명령어를 이용하여 `turtlesim` 노드를 종료할 수 있습니다.
```
rosnode kill /turtlesim
```
해당 명령어를 입력한 쉘에서는 아래와 같은 메시지가 출력됩니다.
```
root@5b055f4ff5ae:/# rosnode kill /turtlesim
killing /turtlesim
killed
```
`turtlesim`이 실행되고 있던 쉘에서는 다음과 같은 메시지가 출력됩니다.
```
[ WARN] [1593956008.895393900]: Shutdown request received.
[ WARN] [1593956008.895502100]: Reason given for shutdown: [user request]
```
rosnode kill /turtlesim
- rosnode cleanup: 연결 정보가 확인 안 되는 노드를 삭제합니다.

위 명령어는 예기치 못한 상황으로 노드가 비정상 종료되었을 때 사용하며, `roscore`를 재실행하지 않아도 되므로 매우 유용합니다.