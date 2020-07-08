## Service Demo 패키지 생성
이번에는 service를 이용하는 패키지를 만들어보도록 하겠습니다. client에 인자로 전달된 두 int64 값을 server가 더하여 반환하는 데모입니다.
```
catkin_create_pkg ros_tutorials_service roscpp std_msgs message_generation
```
## package.xml 수정
```
<?xml version="1.0"?>
<package format="2">
  <name>ros_tutorials_service</name>
  <version>0.0.1</version>
  <description>request/response service</description>
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
## CMakeLists.txt 수정
```
cmake_minimum_required(VERSION 3.0.2)
project(ros_tutorials_service)

find_package(catkin REQUIRED COMPONENTS
  message_generation
  std_msgs
  roscpp
)

add_service_files(FILES SrvTutorial.srv)
generate_messages(DEPENDENCIES std_msgs)
catkin_package(
  LIBRARIES ros_tutorials_service
  CATKIN_DEPENDS std_msgs roscpp
)

include_directories(${catkin_INCLUDE_DIRS})

add_executable(srv_server src/server.cpp)
add_dependencies(srv_server ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(srv_server ${catkin_LIBRARIES})

add_executable(srv_client src/client.cpp)
add_dependencies(srv_client ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(srv_client ${catkin_LIBRARIES})
```
## 서비스 파일 작성
먼저 서비스 파일이 위치할 srv 폴더를 생성한 뒤, 서비스 파일을 생성합니다.
```
roscd ros_tutorials_service
mkdir srv
vi srv/SrvTutorial.srv
```
서비스를 선언합니다.
```
int64 a
int64 b
---
int64 result
```
## srv_server
```
add_executable(srv_server src/server.cpp)
```
`CMakeLists.txt`에서 선언한대로 `srv_server` 노드를 위한 코드 `src/server.cpp`를 작성합니다.
```
#include "ros/ros.h"
#include "ros_tutorials_service/SrvTutorial.h"

// 서비스 요청이 있을 경우, 아래의 처리를 수행
// 요청 = req / 응답 = res
bool add(ros_tutorials_service::SrvTutorial::Request &req, ros_tutorials_service::SrvTutorial::Response &res) {
  // add and save result
  res.result = req.a + req.b;
  // print info
  ROS_INFO("request: %ld + %ld", (long int)req.a, (long int)req.b);
  ROS_INFO("response: %ld", (long int)res.result);
  return true;
}

int main(int argc, char** argv) {
  ros::init(argc, argv, "service_server");
  ros::NodeHandle nh;
  // service name = ros_tutorial_srv
  // callback = add
  ros::ServiceServer ros_tutorials_service_server = nh.advertiseService("ros_tutorial_srv", add);
  ROS_INFO("ready srv server");
  // 요청이 들어오면 add 호출
  // 아니면 대기합니다.
  ros::spin();
  return 0;
}
```
## srv_client
```
add_executable(srv_client src/client.cpp)
```
`CMakeLists.txt`에서 선언한대로 `srv_client`를 위한 `src/client.cpp` 코드를 작성합니다.
```
#include <cstdlib>
#include "ros/ros.h"
#include "ros_tutorials_service/SrvTutorial.h"

int main(int argc, char** argv) {
  ros::init(argc, argv, "service_client");
  if (argc != 3) {
    ROS_INFO("cmd: rosrun ros_tutorials_service service_client arg0 arg1");
    ROS_INFO("arg0: double number, arg1: double number");
    return 1;
  }

  ros::NodeHandle nh;
  ros::ServiceClient client = nh.serviceClient<ros_tutorials_service::SrvTutorial>("ros_tutorial_srv");

  ros_tutorials_service::SrvTutorial srv;
  srv.request.a = atoll(argv[1]);
  srv.request.b = atoll(argv[2]);

  if (client.call(srv)) {
    ROS_INFO("send srv: %d + %d", (long int)srv.request.a, (long int)srv.request.b);
    ROS_INFO("receive srv: %d", (long int)srv.response.result);
  } else {
    ROS_ERROR("Failed to call service ros_tutorial_srv");
    return 1;
  }
  return 0;
}
```
`srv_client` 노드는 인자로 int64 숫자 2개를 넘겨주지 않으면 사용법을 출력하고 종료됩니다.
## 빌드 및 실행
`catkin_ws`로 돌아와 `catkin_make` 명령어를 통해 패키지를 빌드합니다. 이후 아래 명령어를 실행하여 server/client를 테스트합니다.
```
# one shell
rosrun ros_tutorials_service srv_server
# another shell
rosrun ros_tutorials_service srv_client 1 2
```
아래와 같이 표시되어야 합니다.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2F0tnPZ%2FbtqFtwzIutd%2F55raEwTcH8GBom1Un0igkK%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2F0tnPZ%2FbtqFtwzIutd%2F55raEwTcH8GBom1Un0igkK%2Fimg.png)
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FAhMwo%2FbtqFvIFhpsA%2FBR3oNsw52aH2HTL8bH8ka0%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FAhMwo%2FbtqFvIFhpsA%2FBR3oNsw52aH2HTL8bH8ka0%2Fimg.png)