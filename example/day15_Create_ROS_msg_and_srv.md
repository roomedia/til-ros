# 서론
로보티즈에서 공개한 delivery service robot의 코드를 분석하며 클론 코딩을 진행해보려 했는데... 너무 어렵습니다... 대형 프로젝트를 하기 위해선 기초를 다질 필요가 있어 기본적인 내용을 다뤄보려 합니다.

# 독립적인 msg, srv 패키지 만들기
다음 글을 참고했습니다.
http://wiki.ros.org/ROS/Tutorials/CreatingMsgAndSrv

# 1. Introduction to msg and srv
- msg: 메시지 파일은 ROS 메시지의 영역에 대해 기술한 단순한 텍스트 파일입니다. 이 파일은 다양한 언어의 소스코드에서 사용하는 메시지를 만드는 데 사용됩니다.
- srv: 서비스 파일은 request, response 부분으로 나뉩니다.

msg 파일은 패키지 내 msg 폴더에 저장되고, srv 파일은 srv 폴더에 저장됩니다.

메시지는 한 줄 당 필드 유형과 필드 이름이 기술된 단순한 텍스트 파일입니다. 사용할 수 있는 필드 타입은 다음과 같습니다.
- int8, int16, int32, int64 (plus uint*)
- float32, float64
- string
- time, duration
- 다른 메시지 파일
- 변수 크기에 맞는 배열[]과 고정 크기 배열[C]

ROS에는 `Header`라는 특별한 타입이 있습니다. 헤더는 ROS에서 자주 사용되는 타임스템프와 좌표 정보를 포함합니다. 아마 다음과 같이 첫 줄에 `Header header`가 쓰여있는 모습을 자주 보게 될 것입니다.

아래 예제는 헤더, 문자열, 두 개의 다른 메시지를 사용하는 메시지 파일입니다.
```
Header header
string child_frame_id
geometry_msgs/PoseWithCovariance pose
geometry_msgs/TwistWithCovariance twist
```
srv 파일은 msg 파일과 구성이 같으며, request와 response를 `---`로 나눕니다. srv 파일의 예시는 다음과 같습니다.
```
int64 A
int64 B
---
int64 Sum
```
A와 B는 request, Sum은 response입니다.
# 2. 메시지 사용하기
## 2.1 메시지 생성하기
이전에 만든 패키지에 새 메시지를 추가해봅시다.
(저는 이전 튜토리얼을 따라하지 않았기 때문에 새로 패키지를 만들어주었습니다.)
```
roscd beginner_tutorials
mkdir msg
echo "int64 num" > msg/Num.msg
```
예제 msg 파일은 1줄이지만, 아래와 같이 복잡하게 구성할 수도 있습니다.
```
string first_name
string last_name
uint8 age
uint32 score
``` 
메시지를 작성했지만, 메시지가 C++, Python 혹은 다른 언어의 소스 코드로 변환되도록 한 가지 과정을 더 거쳐야 합니다.

`package.xml`을 열고 아래 두 줄을 포함합니다.
```
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
```
빌드 타임에는 `message_generation`만 필요하며, 런타임에는 `message_runtime` 패키지만 필요합니다.

텍스트 에디터에서 `CMakeLists.txt`를 엽니다. (저번 튜토리얼에서 확인한 것처럼, [rosed](http://wiki.ros.org/ROS/Tutorials/UsingRosEd)는 좋은 옵션이 될 수 있습니다.)

CMakeLists 파일 안에 존재하는 `find_package` 호출 부분에 `message_generation` 의존성을 주입하여 메시지를 생성합니다. `COMPONENTS` 부분에 `message_generation`을 넣으면 다음과 같은 모습이 됩니다.
```
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  message_generation
)
```
어떤 상황에선 `find_package`에 아무 의존성 없이 빌드해도 괜찮다는 걸 눈치챘을지도 모르겠습니다. 이는 catkin이 당신의 프로젝트를 하나로 결합하기 때문이며, 이전 프로젝트가 find_package를 호출했으면 당신의 프로젝트는 이를 그대로 사용하기 때문입니다. 하지만 호출을 잊는다면 프로젝트를 독립적으로 빌드했을 때 오류가 나기 쉽습니다.

마찬가지로 `message_runtime`을 export 합니다.
```
catkin_package(
  ...
  CATKIN_DEPENDS message_runtime
  ...
)
```
아래와 같은 코드를 찾으세요.
```
# add_message_files(
#   FILES
#   Message1.msg
#   Message2.msg
# )
```
\# 심볼을 지워 주석을 해제하고 `Message*.msg`를 이전에 생성한 .msg 파일 이름으로 변경하세요. 코드는 다음과 같아집니다.
```
add_message_files(
  FILES
  Num.msg
)
```
이렇게 .msg 파일을 수동으로 넣어주면, 다른 .msg 파일을 추가했을 때 CMake가 프로젝트를 재구성하도록 할 수 있습니다.

이제 `generate_messages()` 함수가 호출될 수 있게 만듭니다.

ROS Hydro와 이후 버전에선 다음의 주석을 해제합니다.
``` 
# generate_messages(
#   DEPENDENCIES
#   std_msgs
# )
```
  아래 코드와 같아집니다.
  ```
  generate_messages(
    DEPENDENCIES
    std_msgs
  )
  ```
_이전 버전_ 에서는 다음 라인을 주석 해제합니다.
```
generate_messages()
```
# 3. Using rosmsg
위 과정을 거치면 메시지가 생성됩니다. `rosmsg show` 커맨드를 사용하여 확인해보도록 합니다.
사용법:
```
rosmsg show [message type]
```
예시:
```
rosmsg show beginner_tutorials/Num
```
다음과 같은 결과가 출력됩니다.
```
int64 num
```
이전 예시에서, 메시지 타입은 두 가지 부분으로 구성되었습니다.
- beginner_tutorials -- 메시지가 정의된 패키지
- Num -- 메시지 파일의 이름
메시지가 어떤 패키지에 있는지 잊었다면 메시지 이름을 비워둘 수 있습니다. 다음을 시도해보세요.
```
rosmsg show Num
```
아래와 같이 표시될 것입니다.
```
[beginner_tutorials/Num]:
int64 num
```
# 4. srv 사용하기
## 4.1 srv 만들기
srv를 생성하기 위해 방금 사용한 프로젝트를 다시 쓰겠습니다.
```
roscd beginner_tutorials
mkdir srv
```
새로운 srv 파일을 만드는 대신 기존 패키지의 서비스 파일을 복사해오겠습니다.

사용법:
```
roscp [package_name] [file_to_copy_path] [copy_path]
```
`rospy_tutorials` 패키지의 서비스 파일을 복사해보겠습니다.
```
roscp rospy_tutorials AddTwoInts.srv srv/AddTwoInts.src
```
한 가지 과정이 더 남았습니다. srv 파일이 C++, Python, 기타 다른 언어의 소스코드로 변환되도록 해야 합니다.

이전에 했던 것처럼, `package.xml`에 다음 두 라인을 추가합니다.
```
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
```
이전에 설명한 것처럼, 빌드 시에는 `message_generation`만 필요하며, 런타임에는 `message_runtime`만 필요하다는 걸 명심하세요!

이전 단계를 진행하지 않았다면, `message_generation`을 `CMakeLists.txt`에 추가하세요.
```
# 아래 코드를 CMakeLists.txt에 복사-붙여넣기 하여 추가하지 마시고, 기존 코드를 수정하세요.
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  message_generation
)
```
(이름은 message_generation이지만, msg와 srv 모두 동작합니다.)

또한, 이전에 했던 것처럼 `package.xml`을 바꿔야 합니다.

아래 코드에서 #을 지워 주석을 해제하세요.
```
# add_service_files(
#   FILES
#   Service1.srv
#   Service2.srv
# )
```
그리고 Service*.srv 파일을 우리의 서비스 파일 이름으로 대체합니다.
```
add_service_files(
  FILES
  AddTwoInts.srv
)
```
이제 서비스 파일을 생성할 준비가 되었습니다. 지금 당장 하고 싶다면, 다음 섹션을 스킵하고 5장으로 넘어가세요.
## 4.2 rossrv 사용하기
srv를 생성하는 절차를 모두 확인했습니다. `rossrv show` 명령어를 사용하여 서비스를 확인할 수 있습니다.
사용법:
```
rossrv show <service type>
```
예제:
```
rossrv show beginner_tutorials/AddTwoInts
```
아래와 같은 메시지가 보입니다.
```
int64 a
int64 b
---
int64 sum
```
`rosmsg`와 마찬가지로, 패키지 이름 없이 서비스 파일을 찾을 수 있습니다.
```
rossrv show AddTwoInts
[beginner_tutorials/AddTwoInts]:
int64 a
int64 b
---
int64 sum

[rospy_tutorials/AddTwoInts]:
int64 a
int64 b
---
int64 sum
```
# 5. Common step for msg and srv
이전 스텝에서 변경하지 않았다면, CMakeLists.txt를 다음과 같이 변경합니다.
```
# generate_messages(
#   DEPENDENCIES
# #  std_msgs  # Or other packages containing msgs
# )
```
주석을 해제하고 .msg 파일의 자료형이 사용하는 (여기서는 std_msgs) 패키지를 넣어줍니다.
```
generate_messages(
  DEPENDENCIES
  std_msgs
)
```
이제 새로운 메시지를 만들었으니 패키지를 make 합니다.
```
roscd beginner_tutorials
cd ../..
catkin_make
cd -
```
msg 디렉토리의 모든 .msg 파일이 모든 지원되는 언어의 코드로 생성됩니다. C++ 헤더 파일은 `~/catkin_ws/devel/include/beginner_tutorials/`에 생성되며, python 스크립트는 `~/catkin_ws/devel/lib/python2.7/dist-packages/beginner_tutorials/msg`, lisp 파일은 `~/catkin_ws/devel/share/common-lisp/ros/beginner_tutorials/msg/`에 생성됩니다.

마찬가지로 srv 디렉토리 안 모든 .srv 파일도 비슷한 경로에 생성됩니다. C++의 경우 msg와 같은 폴더를 사용하며, python, list는 상위 폴더에 생성된 srv 폴더에 위치합니다.

메시지 포맷에 대한 전체 스펙은 [Message Description Language](http://wiki.ros.org/ROS/Message_Description_Language) 페이지에서 확인할 수 있습니다.

새로운 메시지를 이용하는 C++ 노드를 만들고 있다면, [catkin msg/srv build documentation](http://docs.ros.org/latest/api/catkin/html/howto/format2/building_msgs.html)에 나와있는대로 노드와 메시지 간 의존성을 선언해야 합니다.

# 6. 도움말
이번 장에서 ROS 툴을 잠시 살펴봤습니다. 각각의 명령어에 어떤 요소가 필요한지 파악하기 어려울 수 있습니다. 다행히도, 대부분의 ROS 툴은 도움말을 제공합니다.
시도하기:
```
rosmsg -h
```
아래와 같은 서브 커맨드를 볼 수 있습니다.
```
Commands:
  rosmsg show     Show message description
  rosmsg list     List all messages
  rosmsg md5      Display message md5sum
  rosmsg package  List messages in a package
  rosmsg packages List packages that contain messages
```
서브 커맨드에 대한 도움말도 제공됩니다.
```
rosmsg show -h
```
이는 rosmsg show가 필요로 하는 인자를 보여줍니다.
```
Usage: rosmsg show [options] <message type>

Options:
  -h, --help  show this help message and exit
  -r, --raw   show raw message text, including comments
```
# 7. 리뷰
이제까지 사용한 명령어를 정리합시다:

- rospack = ros+pack(age) : ROS 패키지와 관련한 정보를 제공합니다.
- roscd = ros+cd : ROS 패키지, ROS 스택 경로로 이동합니다.
- rosls = ros+ls : ROS 패키지 내 파일을 보여줍니다.
- roscp = ros+cp : ROS 패키지에서 다른 패키지로 파일을 복사합니다.
- rosmsg = ros+msg : ROS 메시지 정의와 관련된 정보를 보여줍니다.
- rossrv = ros+srv : ROS 서비스 정의와 관련된 정보를 보여줍니다.
- catkin_make : ROS 패키지를 컴파일합니다.
- rosmake = ros+make : ROS 패키지를 컴파일합니다. catkin을 사용하지 않는 *이전 버전*에서 사용됩니다.
# 8. 다음 튜토리얼
메시지와 서비스를 만들었으니 다음 시간에는 간단한 publisher, subscriber를 만들어 보도록 하겠습니다.
[(python)](http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28python%29) [(c++)](http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28c%2B%2B%29)