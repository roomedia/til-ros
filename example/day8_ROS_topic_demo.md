# ROS 기본 프로그래밍
## 표준 단위
ROS에서는 메시지에 표준 단위 사용을 권장하고 있습니다. 필요에 따라 재사용, 재정의도 가능하나 사용 단위만큼은 SI 단위를 반드시 지켜야 합니다.
Quantity | Unit
-- | --
length | meter
mass | kilogram
time | second
current | ampare
angle | radian
frequency | hertz
force | newton
power | watt
voltage | volt
temperature | celsius
## REP(ROS Enhancement Proposals)
REP는 ROS 커뮤니티에서 유저들이 제안하는 규칙, 새로운 기능, 관리 방법을 담은 제안서입니다. 유저 간에 정리된 표준 문서로, 아래 링크에서 확인해 볼 수 있습니다.
[http://www.ros.org/reps/rep-0000.html](http://www.ros.org/reps/rep-0000.html)
## axes
로봇의 회전 방향은 오른손 법칙을 따릅니다. 오른손으로 따봉을 했을 때 손가락이 감기는, 반시계 방향이 회전의 정방향(+)입니다.
회전축은 아래 사진과 같이 손가락을 펼쳤을 때, 엄지가 Z축, 검지가 X축, 중지가 Y축이 됩니다.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbrpAId%2FbtqFuhV5AG9%2FbdUvKEugrg3VmKSJprH0L1%2Fimg.jpg](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbrpAId%2FbtqFuhV5AG9%2FbdUvKEugrg3VmKSJprH0L1%2Fimg.jpg)
## style guide
변수 이름을 명명할 때에도 규칙이 있습니다. 필수는 아니지만, 남의 코드를 이해할 때 도움이 됩니다.
대상 | 명명 규칙 | 예시
-- | -- | --
패키지 | under_scored | first_ros_package
토픽, 서비스 | under_scored | raw_image
파일 | under_scored | turtlebot3_fake.cpp (* 단 메시지, 서비스, 액션 파일은 CamelCased 규칙을 따른다.)
네임스페이스 | under_scored | ros_awesome_package
변수 | under_scored | string table_name;
타입 | CamelCased | typedef int32_t PropertiesNumber;
클래스 | CamelCased | class UrlTable;
구조체 | CamelCased | struct UrlTableProperties;
열거형 | CamelCased | enum ChoiceNumber;
함수 | camelCased | addTableEntry();
메소드 | camelCased | void setNumEntries(int32_t num_entries);
상수 | ALL_CAPITALS | const unit8_t DAYS_IN_A_WEEK = 7;
매크로 | ALL_CAPITALS | #define PI_ROUNDED 3.0;
## 패키지 생성
이전에 했던대로 패키지를 생성해보겠습니다. 이번에 생성할 패키지의 이름은 `ros_tuturials_topic`으로 하겠습니다. `message_generation`과 `std_msgs`, `roscpp` 의존성을 주입해줍니다.
```
# ROS 1
catkin_create_pkg ros_tutorials_topic message_generation std_msgs roscpp
```
다음과 같이 파일이 생성된 것을 볼 수 있습니다.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FZ8sAx%2FbtqFtxyz04K%2FbNJjPNjhV4YokXdgxeSwE0%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FZ8sAx%2FbtqFtxyz04K%2FbNJjPNjhV4YokXdgxeSwE0%2Fimg.png)
## package.xml 수정
```xml
<?xml version="1.0"?>
<package format="2">
  <name>ros_tutorials_topic</name>
  <version>0.0.1</version>
  <description>publish/subscribe topic</description>
  <license>Apache License 2.0</license>
  <maintainer email="root@to.do">someone</maintainer>

  <buildtool_depend>catkin</buildtool_depend>
  <build_depend>message_generation</build_depend>
  <build_depend>roscpp</build_depend>
  <build_depend>std_msgs</build_depend>

  <build_export_depend>roscpp</build_export_depend>
  <build_export_depend>std_msgs</build_export_depend>

  <exec_depend>roscpp</exec_depend>
  <exec_depend>std_msgs</exec_depend>
  <exec_depend>message_runtime</exec_depend>

  <export></export>
</package>
```
`<run_depend>` 태그는 catkin format 2부터 `<exec_depend>`로 변경되었습니다. 개인적으로.. ROS는 버전 별로 빌드 시스템의 이름이나 이런 식의 키워드 변경이 잦아 매우 불편합니다. 이런 부분 하나하나가 하위 호환성을 망치고 생산성을 약화하는 것 같은데 왜 맨날 이름이나 바꾸고 있을까요...?
## CMakeLists.txt 수정
```
cmake_minimum_required(VERSION 3.0.2)
project(ros_tutorials_topic)

find_package(catkin REQUIRED COMPONENTS
  message_generation
  roscpp
  std_msgs
)

add_message_files(FILES MsgTutorial.msg)
generate_messages(DEPENDENCIES std_msgs)
catkin_package(
  LIBRARIES ros_tutorials_topic
  CATKIN_DEPENDS std_msgs roscpp
)

include_directories(${catkin_INCLUDE_DIRS})

add_executable(topic_publisher src/publisher.cpp)
add_dependencies(topic_publisher ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(topic_publisher ${catkin_LIBRARIES})

add_executable(topic_subscriber src/subscriber.cpp)
add_dependencies(topic_subscriber ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(topic_subscriber ${catkin_LIBRARIES})
```
## 메시지 파일 작성
먼저 메시지 파일이 위치할 폴더를 생성한 뒤, 메시지 파일을 생성합니다.
```
roscd ros_tutorials_topic
mkdir msg
vi msg/MsgTutorial.msg
```
메시지를 선언합니다.
```
time stamp
int32 data
```
int32 sec과 int32 nsec으로 이루어진 time 형태의 stamp와 int32 형태의 data를 선언했습니다.
## topic_publisher
```
add_executable(topic_publisher src/publisher.cpp)
```
`CMakeLists.txt`에서 선언한대로 `topic_publisher` 노드를 위한 코드 `publisher.cpp`를 작성합니다.
```
#include "ros/ros.h"
#include "ros_tutorials/MsgTutorial.h"

int main(int argc, char** argv) {
  ros::init(argc, argv, "publisher");
  ros::NodeHandle nh;

  // topic name = ros_tutorial_msg
  // queue size = 100
  ros::Publisher pub = nh.advertise<ros_tutorials::MsgTutorial>("ros_tutorial_msg", 100);
  
  // scale_of_hz = 1
  // 1초 간격으로 처리가 반복된다
  ros::Rate loop_rate(1);
  ros_tutorials::MsgTutorial msg;
  int count = 0;

  while (ros::ok()) {
    msg.stamp = ros::Time::now();
    msg.data = count;

    ROS_INFO("send msg = %d", msg.stamp.sec);
    ROS_INFO("send msg = %d", msg.stamp.nsec);
    ROS_INFO("send msg = %d", msg.data);

    pub.publish(msg);
    loop_rate.sleep();
    count++;
  }
  return 0;
}
```
ROS 1에서는 `ros/ros.h`를 이용해 관련 함수를 포함하지만, ROS 2에서는 `rclcpp/rclcpp.hpp`를 이용합니다.
## topic_subscriber
```
add_executable(topic_subscriber src/subscriber.cpp)
```
`CMakeLists.txt`에서 선언한대로 `topic_subscriber` 노드를 위한 코드 `subscriber.cpp`를 작성합니다.
```
#include "ros/ros.h"
#include "ros_tutorials/MsgTutorial.h"

void msgCallback(const ros_tutorials::MsgTutorial::ConstPtr& msg) {
  ROS_INFO("receive msg = %d", msg->stamp.sec);
  ROS_INFO("receive msg = %d", msg->stamp.nsec);
  ROS_INFO("receive msg = %d", msg->data);
}

int main(int argc, char** argv) {
  ros::init(argc, argv, "subscriber");
  ros::NodeHandle nh;
  // topic name = ros_tutorial_msg <- topic_publisher가 발행한 topic 이름과 같아야 합니다.
  // queue size = 100
  // callback = msgCallback <- 메시지를 받으면 실행됩니다.
  ros::Subscriber sub = nh.subscribe("ros_tutorial_msg", 100, msgCallback);

  // 메시지를 받을 때까지 대기합니다.
  // 메시지를 받으면 subscriber의 callback을 실행합니다.
  ros::spin();
  return 0;
}
```
## 빌드 및 실행
`catkin_ws`로 돌아와 `catkin_make` 명령어를 통해 패키지를 빌드합니다. 이후 아래 명령어를 실행하여 publisher/subscriber를 테스트합니다.
```
# one shell
rosrun ros_tutorials_topic topic_publisher
# another shell
rosrun ros_tutorials_topic topic_subscriber
```
아래와 같이 표시되어야 합니다.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FDv3mC%2FbtqFt4P9Xhm%2FQw6LvxYvkzkxHNECOOXpo1%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FDv3mC%2FbtqFt4P9Xhm%2FQw6LvxYvkzkxHNECOOXpo1%2Fimg.png)
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fbf18Sy%2FbtqFvIE87uy%2FZUkeeaFQpHeXHZCgYhmDh1%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fbf18Sy%2FbtqFvIE87uy%2FZUkeeaFQpHeXHZCgYhmDh1%2Fimg.png)