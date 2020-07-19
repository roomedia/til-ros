[참고자료](http://docs.ros.org/latest/api/catkin/html/howto/format2/building_msgs.html)
# 메시지, 서비스, 액션 만들기
## package.xml
`package.xml`에는 반드시 `message_generation`에 대한 `<build_depend>`가 선언되어야 하며, `message_runtime`에 대한 `<build_export_depend>` 및 `<exec_depend>`가 선언되어야 합니다.
```xml
<build_depend>message_generation</build_depend>
<build_export_depend>message_runtime</build_export_depend>
<exec_depend>message_runtime</exec_depend>
```
메시지, 서비스, 액션은 [std_msgs](http://wiki.ros.org/std_msgs)처럼 다른 ROS 메시지에서 정의된 자료형을 사용할 수 있습니다. 다음과 같이 선언하세요:
```xml
<depend>std_msgs</depend>
```
메시지 의존성을 표시할 때에는 `<depend>` 태그가 적절합니다. 위 예제의 의존성은 `std_msgs` 밖에 없다고 가정합니다. 모든 메시지 패키지 의존성을 여기 기술해야 하며, `std_msgs`처럼 채워넣습니다.

액션을 만드려면, `actionlib_msgs`를 의존성으로 추가하세요:
```xml
<depend>actionlib_msgs</depend>
```
## CMakeLists.txt
CMake에서 `message_generation` 및 기타 다른 메시지, 서비스, 액션 의존성 catkin packages를 찾습니다.
```cmake
find_package(catkin REQUIRED COMPONENTS message_generation std_msgs)
```
액션을 만드려면 `actionlib_msgs`를 의존성으로 넣으세요:
```cmake
find_package(catkin REQUIRED
             COMPONENTS
             actionlib_msgs
             message_generation
             std_msgs)
```
그 다음, 메시지 정의 파일을 나열하세요:
```cmake
add_message_files(DIRECTORY msg
                  FILES
                  YourFirstMessage.msg
                  YourSecondMessage.msg
                  YourThirdMessage.msg)
```
서비스를 만드는 방법도 유사합니다:
```cmake
add_service_files(DIRECTORY srv
                  FILES
                  YourService.srv)
```
액션을 만드려면 다음을 추가하세요:
```
add_action_files(DIRECTORY action
                 FILES
                 YourStartAction.action
                 YourStopAction.action)
```
다음으로 아래 명령어를 사용하여 모든 메시지, 서비스, 액션 타겟을 생성합니다.
```
generate_messages(DEPENDENCIES std_msgs)
```
`catkin_package()` 명령어가  메시지, 서비스, 액션 의존성 패키지를 선언하도록 변경하세요
```
catkin_package(CATKIN_DEPENDS message_runtime std_msgs)
```
좋은 ROS 습관은 연관된 메시지, 서비스, 액션을 다른 API와는 독립된 패키지로 모으는 것입니다. 이는 패키지 의존성 그래프를 단순화합니다.

스크립트와 프로그램을 메시지 패키지와 함께 제공할 수 있습니다. 이를 위해, 메시지 생성 타겟은 여기 의존하는 다른 프로그램보다 먼저 빌드되어야 합니다. 직간접적으로 커스텀 메시지 헤더를 사용하는 모든 타겟은 명시적인 의존성을 선언해야 합니다.
```cmake
add_dependencies(your_program ${${PROJECT_NAME}_EXPORTED_TARGETS})
```
빌드 타겟이 다른 catkin 패키지에서 임포트한 메시지나 서비스 헤더를 사용한다면, 이러한 의존성도 선언해야 합니다:
```cmake
add_dependencies(your_program ${catkin_EXPORTED_TARGETS})
```
catkin이 메시지, 서비스, 액션 타겟을 자동으로 설치하기 때문에, `install()` 명령어는 필요하지 않습니다.