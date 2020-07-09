## Action Demo 패키지 생성
이번에는 action을 이용하는 패키지를 만들어보도록 하겠습니다. goal로써 전달된 값만큼 Fibonacci 수열을 계산하며 계산 과정은 feedback, 최종 결과는 result로 반환하는 데모입니다.
```
catkin_create_pkg ros_tutorials_action actionlib actionlib_msgs roscpp std_msgs message_generation
```
이번 데모에서는 추가로 `actionlib`를 사용합니다. `actionlib`는 긴 시간 동안 요청에 대한 결과가 반환되지 않을 때 `timeout`을 해줍니다. 서버와 클라이언트 모두 클래스 형태로 제공됩니다.
## package.xml 수정
```xml
<?xml version="1.0"?>
<package format="2">
  <name>ros_tutorials_action</name>
  <version>0.0.0</version>
  <description>The ros_tutorials_action package</description>
  <maintainer email="root@todo.todo">root</maintainer>
  <license>TODO</license>

  <buildtool_depend>catkin</buildtool_depend>

  <build_depend>roscpp</build_depend>
  <build_depend>actionlib</build_depend>
  <build_depend>actionlib_msgs</build_depend>
  <build_depend>message_generation</build_depend>
  <build_depend>std_msgs</build_depend>

  <build_export_depend>actionlib</build_export_depend>
  <build_export_depend>actionlib_msgs</build_export_depend>
  <build_export_depend>std_msgs</build_export_depend>

  <exec_depend>roscpp</exec_depend>
  <exec_depend>actionlib</exec_depend>
  <exec_depend>actionlib_msgs</exec_depend>
  <exec_depend>std_msgs</exec_depend>

  <export></export>
</package>
```
## CMakeLists.txt 수정
```
cmake_minimum_required(VERSION 3.0.2)
project(ros_tutorials_action)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  actionlib
  actionlib_msgs
  message_generation
  std_msgs
)
find_package(Boost REQUIRED COMPONENTS system)

add_action_files(FILES Fibonacci.action)
generate_messages(DEPENDENCIES actionlib_msgs std_msgs)

catkin_package(
  LIBRARIES ros_tutorials_action
  CATKIN_DEPENDS roscpp actionlib actionlib_msgs std_msgs
  DEPENDS Boost
)

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)

add_executable(action_server src/action_server.cpp)
add_dependencies(action_server ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(action_server ${catkin_LIBRARIES})

add_executable(action_client src/action_client.cpp)
add_dependencies(action_client ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(action_client ${catkin_LIBRARIES})S} ${catkin_EXPORTED_TARGETS})
```
## 액션 파일 작성
먼저 액션 파일이 위치할 action 폴더를 생성한 뒤, `Fibonacci.action` 파일을 생성합니다.
```
# goal
int32 order
---
# result
int32[] sequence
---
# feedback
int32[] sequence
```
goal, result, feedback은 빌드 이후 각각 `ros_tutorials_action` namespace에 `FibonacciGoal`, `FibonacciFeedback`, `FibonacciResult` 형태로 존재합니다. 해당 namespace에는 이 세 클래스를 포함하는 `FibonacciAction`도 존재합니다.
## action_server
```
add_executable(action_server src/action_server.cpp)
```
`CMakeLists.txt`에서 선언한대로 `action_server` 노드를 위한 코드 `src/action_server.cpp`를 작성합니다.
```c++
#include "ros/ros.h"
#include "actionlib/server/simple_action_server.h"
#include "ros_tutorials_action/FibonacciAction.h"

class FibonacciAction {
protected:
  // 노드 핸들
  ros::NodeHandle nh_;
  // 액션 서버
  actionlib::SimpleActionServer<ros_tutorials_action::FibonacciAction> as_;
  // 액션 이름
  std::string action_name_;
  // 액션 피드백 및 결과
  ros_tutorials_action::FibonacciFeedback feedback_;
  ros_tutorials_action::FibonacciResult result_;
public:
  // 액션 서버 초기화
  FibonacciAction(std::string name) :
  // 핸들, 액션 이름, 콜백, auto_start
  // actionlib document에 따르면 auto_start가 true일때 ros::spin을 사용하지 않아도 된다고 합니다.
  // boost::bind는 메소드를 callback으로 넘겨주며,
  // 전달되는 첫 번째 값을 callback 함수의 2번째 파라미터로 넘겨줍니다. (첫 번째는 자기자신)
  as_(nh_, name, boost::bind(&FibonacciAction::executeCB, this, _1), false),
  action_name_(name) {
    as_.start();
  }

  ~FibonacciAction() {
  }

  // goal을 받아 지정된 액션을 수행
  void executeCB(const ros_tutorials_action::FibonacciGoalConstPtr &goal) {
    ros::Rate r(1); // 1hz
    bool isSuccess = true;
    // 시퀀스 vector를 초기화합니다.
    feedback_.sequence.clear();
    feedback_.sequence.push_back(0);
    feedback_.sequence.push_back(1);

    ROS_INFO("%s: Executing, creating fibonacci sequence of order %i with seeds %i, %i", action_name_.c_str(), goal->order, feedback_.sequence[0], feedback_.sequence[1]);

    for (int i=1; i<=goal->order; i++) {
      // cancel action
      if (as_.isPreemptRequested() || !ros::ok()) {
        ROS_INFO("%s: Preempted", action_name_.c_str());
        as_.setPreempted();
        isSuccess = false;
        break;
      }
      
      feedback_.sequence.push_back(feedback_.sequence[i] + feedback_.sequence[i-1]);
      as_.publishFeedback(feedback_);
      r.sleep();
    }

    if (isSuccess) {
      result_.sequence = feedback_.sequence;
      ROS_INFO("%s: Succeeded", action_name_.c_str());
      as_.setSucceeded(result_);
    }
  }
};

int main(int argc, char** argv) {
  ros::init(argc, argv, "action_server"); // set node name
  FibonacciAction fibonacci("ros_tutorial_action"); // set action name
  ros::spin();
  return 0;
}
```
## action_client
```
add_executable(action_client src/action_client.cpp)
```
`CMakeLists.txt`에서 선언한대로 `action_client`를 위한 `src/action_client.cpp` 코드를 작성합니다.
```c++
#include "ros/ros.h"
#include "actionlib/client/simple_action_client.h"
#include "actionlib/client/terminal_state.h"
#include "ros_tutorials_action/FibonacciAction.h"

int main(int argc, char** argv) {
  ros::init(argc, argv, "action_client"); // set node name
  actionlib::SimpleActionClient<ros_tutorials_action::FibonacciAction> ac("ros_tutorial_action", true); // input exact same action name

  ROS_INFO("Waiting for action server to start");
  ac.waitForServer();

  ROS_INFO("Action server started, sending goal.");
  ros_tutorials_action::FibonacciGoal goal;
  goal.order = 20;
  ac.sendGoal(goal);

  bool finished_before_timeout = ac.waitForResult(ros::Duration(30.0));
  if (finished_before_timeout) {
    actionlib::SimpleClientGoalState state = ac.getState();
    ROS_INFO("Action Finished: %s", state.toString().c_str());
  }
  else {
    ROS_INFO("Action did not finish before timeout");
  }
  return 0;
}
```
`action_client` 노드를 실행하면 `result`가 반환될 때까지 (혹은 30초의 timeout이 되기 전까지) 대기하다 `result`를 출력하고 종료됩니다.
## 빌드 및 실행
`catkin_ws`로 돌아와 `catkin_make` 명령어를 통해 패키지를 빌드합니다. 이후 아래 명령어를 실행하여 server/client를 테스트합니다. `rostopic`을 사용하여 출력되는 feedback을 확인할 수 있습니다.
```bash
# one shell
rosrun ros_tutorials_action action_server
# another shell
rosrun ros_tutorials_action action_client
# other shell
rostopic echo /ros_tutorial_action/feedback
```
아래와 같이 표시되어야 합니다. 위에서부터 서버, 클라이언트, rostopic입니다.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbGbp2G%2FbtqFuhWTFN4%2FjIMIQcp3ubkPpmtMn96Ukk%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbGbp2G%2FbtqFuhWTFN4%2FjIMIQcp3ubkPpmtMn96Ukk%2Fimg.png)
