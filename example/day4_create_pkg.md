이전에 생성하였던 `catkin_ws/src`로 이동하여 패키지를 생성합니다.
```
cd ~/catkin_ws/src`
catkin_create_pkg my_first_ros_pkg std_msgs roscpp
```
패키지 생성 명령어는 `catkin_create_pkg 생성할_패키지_이름 [의존_패키지_1, ...]`로 이루어져 있습니다. 여기서는 `std_msgs`와 `roscpp`를 이용합니다.

패키지 생성 이후에는 `package.xml` 파일을 수정합니다. `package.xml` 파일은 패키지에 대한 설명을 담고 있는 파일로, 초기 파일에는 각 항목에 대한 설명이 적혀 있습니다.
```
<?xml version="1.0"?>
<package format="2">
  <name>my_first_ros_pkg</name>
  <version>0.0.0</version>
  <description>The my_first_ros_pkg package</description>

  <!-- One maintainer tag required, multiple allowed, one person per tag -->
  <!-- Example:  -->
  <!-- <maintainer email="jane.doe@example.com">Jane Doe</maintainer> -->
  <maintainer email="root@todo.todo">root</maintainer>


  <!-- One license tag required, multiple allowed, one license per tag -->
  <!-- Commonly used license strings: -->
  <!--   BSD, MIT, Boost Software License, GPLv2, GPLv3, LGPLv2.1, LGPLv3 -->
  <license>TODO</license>


  <!-- Url tags are optional, but multiple are allowed, one per tag -->
  <!-- Optional attribute type can be: website, bugtracker, or repository -->
  <!-- Example: -->
  <!-- <url type="website">http://wiki.ros.org/my_first_ros_pkg</url> -->


  <!-- Author tags are optional, multiple are allowed, one per tag -->
  <!-- Authors do not have to be maintainers, but could be -->
  <!-- Example: -->
  <!-- <author email="jane.doe@example.com">Jane Doe</author> -->


  <!-- The *depend tags are used to specify dependencies -->
  <!-- Dependencies can be catkin packages or system dependencies -->
  <!-- Examples: -->
  <!-- Use depend as a shortcut for packages that are both build and exec dependencies -->
  <!--   <depend>roscpp</depend> -->
  <!--   Note that this is equivalent to the following: -->
  <!--   <build_depend>roscpp</build_depend> -->
  <!--   <exec_depend>roscpp</exec_depend> -->
  <!-- Use build_depend for packages you need at compile time: -->
  <!--   <build_depend>message_generation</build_depend> -->
  <!-- Use build_export_depend for packages you need in order to build against this package: -->
  <!--   <build_export_depend>message_generation</build_export_depend> -->
  <!-- Use buildtool_depend for build tool packages: -->
  <!--   <buildtool_depend>catkin</buildtool_depend> -->
  <!-- Use exec_depend for packages you need at runtime: -->
  <!--   <exec_depend>message_runtime</exec_depend> -->
  <!-- Use test_depend for packages you need only for testing: -->
  <!--   <test_depend>gtest</test_depend> -->
  <!-- Use doc_depend for packages you need only for building documentation: -->
  <!--   <doc_depend>doxygen</doc_depend> -->
  <buildtool_depend>catkin</buildtool_depend>
  <build_depend>roscpp</build_depend>
  <build_depend>std_msgs</build_depend>
  <build_export_depend>roscpp</build_export_depend>
  <build_export_depend>std_msgs</build_export_depend>
  <exec_depend>roscpp</exec_depend>
  <exec_depend>std_msgs</exec_depend>


  <!-- The export tag contains other, unspecified, tags -->
  <export>
    <!-- Other tools can request additional information be placed here -->

  </export>
</package>
```
다음으론 빌드 설정 파일인 `CMakeLists.txt`를 살펴보겠습니다.
`project({project_name})`에 입력된 이름과 `package.xml`에서 입력한 패키지 이름이 다를 경우 에러가 발생합니다.

`find_package(패키지1 REQUIRED COMPONENTS [패키지2, ...])` 해당 옵션은 패키지1의 의존성 패키지를 먼저 설치하는 효과가 있습니다. 이후 나오는 `catkin_python_setup()`은 rospy를 사용한 프로젝트를 빌드할 때 사용되는 문구입니다.

다음으로 메시지를 정의하는 영역과 ROS dynamic reconfigure 영역이 있습니다. 해당 영역에서는 각각 메시지 파일/cfg 파일을 불러옵니다.

다음과 같이 빌드 설정 파일을 수정합니다.
```
cmake_minimum_required(VERSION 3.0.2)
project(my_first_ros_pkg)
find_package(catkin REQUIRED COMPONENTS roscpp std_msgs)
catkin_package(CATKIN_DEPENDS roscpp std_msgs)
include_directories(${catkin_INCLUDE_DIRS})
add_executable(hello_world_node src/hello_world_node.cpp)
target_link_libraries(hello_world_node ${catkin_LIBRARIES})
```
빌드 설정 파일에서 실행 파일을 만들 때 src 폴더 아래 있는 `hello_world_node.cpp`를 사용하기로 했습니다. 해당 파일을 만들고 아래와 같은 내용을 작성합니다.
```
#include <sstream>
#include <ros/ros.h>
#include <std_msgs/String.h>

int main(int argc, char** argv) {
  ros::init(argc, argv, "hello_world_node");
  ros::NodeHandle nh;
  ros::Publisher chatter_pub = nh.advertise<std_msgs::String>("say_hello_world", 1000);
  ros::Rate loop_rate(10);
  int count = 0;

  while (ros::ok()) {
    std_msgs::String msg;
    std::stringstream ss;
    ss << "hello world: " << count++;
    msg.data = ss.str();
    ROS_INFO("%s", msg.data.c_str());
    chatter_pub.publish(msg);
    ros::spinOnce();
    loop_rate.sleep();
  }
  return 0;
}
```
해당 파일 작성을 마쳤으면 먼저 작성한 패키지를 ROS 패키지 목록에 반영합니다.
```
rospack profile
```
이후 catkin을 사용하여 빌드합니다.
```
cd ~/catkin_ws && catkin_make
```
`roscore`가 구동된 상태에서 아래 명령어를 실행하면 패키지를 실행할 수 있습니다.
```
rosrun my_first_ros_pkg hello_world_node
```
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2F03HYW%2FbtqFoQwOZ1c%2FK3730FQVdwvSuzuNuvJKok%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2F03HYW%2FbtqFoQwOZ1c%2FK3730FQVdwvSuzuNuvJKok%2Fimg.png)