# 마이그레이션 가이드
ROS Melodic Morenia에서 아래와 같은 패키지가 변경되었고, 이에 따라 마이그레이션할 때 주의할 점과 튜토리얼을 제공합니다.

Kinetic이나 그 이전 버전 사용자라면, ROS Lunar 마이그레이션 가이드를 먼저 보고 오십시오.

# 1. C++14
[target platforms document](http://www.ros.org/reps/rep-0003.html#melodic-morenia-may-2018-may-2023)에 명시된 것처럼, Melodic은 C++11 대신 C++14를 사용하는 첫 번째 버전입니다. 때문에 몇몇 프로젝트의 경우 버전 간 호환성 문제가 발생할 수 있습니다.
# 2. class_loader
## 2.1 Header deprecation
멀티 플랫폼 지원 및 ROS 2 대응을 위해 class_loader 헤더는 `.h`에서 `.hpp`로 이름이 변경되었습니다. 마이그레이션을 위한 [스크립트](https://github.com/ros/class_loader/commit/1515546103de4d987daa8f5519ca43fe6cffbca6)가 제공되며, 이전 ROS 배포판에 릴리즈된 모든 패키지에 PR이 생성됩니다.
# 3. kdl_parser
kdl_parser 패키지(https://github.com/ros/kdl_parser)는 tinyxml을 사용하는 메소드를 더이상 사용하지 않습니다.
```
bool treeFromXml(TiXmlDocument * xml_doc, KDL::Tree & tree)
```
다음 코드는 아래와 같이 tinyxml2를 이용하도록 변경되어야 합니다.
```
bool treeFromXml(const tinyxml2::XMLDocument * xml_doc, KDL::Tree & tree)
```
더이상 사용되지 않는 tinyxml API는 ROS Noetic에서 제거되었습니다.
# 4. opencv
표준화를 위해 opencv 버전은 [최소 3.2 버전](http://www.ros.org/reps/rep-0003.html#melodic-morenia-may-2018-may-2023)을 사용합니다. ROS Lunar에서 opencv3 패키지를 사용하고 있었다면 반드시 [libopencv-dev rosdep rule](https://github.com/ros/rosdistro/blob/16a0418db0b120852ff78e015d22512ada6be415/rosdep/base.yaml#L2431)로 변경하여야 합니다.
# 5. pluginlib
## 5.1 Header deprecation
멀티 플랫폼 및 ROS 2에 대응하기 위해 pluginlib의 헤더 또한 이름이 변경되었습니다. [ros/pluginlib#79](https://github.com/ros/pluginlib/pull/70) 마이그레이션 스크립트가 [제공되며](https://github.com/ros/pluginlib/pull/104) 이전 버전을 대상으로 릴리즈 된 모든 패키지에 PR이 생성됩니다.
## 5.2 plugin_tool 제거
plugin_tool은 제대로 동작하지 않으며 수 년간 유지보수되지 않아 ROS Melodic에서 제거되었습니다. 논의와 PR은 [여기](https://github.com/ros/pluginlib/pull/61)에서 확인할 수 있습니다.
# 6. rosconsole
`rosconsole` 패키지가 `ros_comm` 레포지토리로부터 독립했습니다. 사유는 `ros_comm` 레포 내부 `rosbag_storage`가 의존성으로 사용하려는 `pluginlib`이 `rosconsole`에 의존적이기 때문입니다. 자세한 내용은 [여기](https://github.com/ros/rosdistro/issues/17919)에서 확인할 수 있습니다.
# 7. rviz
## 7.1 tf API가 tf2 API로 대체되었습니다.
tf (version 1) API가 더이상 사용하지 않게 되었고 ROS의 다음 버전에서는 tf2를 사용하게 되었습니다. rviz는 이제 tf를 포함하지 않습니다.

이 변경 사항으로 인해 다른 플러그인 (특히 디스플레이 타입의 플러그인)도 더이상 사용되지 않을 수 있습니다.

아래 예제는 새로운 API로 마이그레이션 하는 방법을 설명하고 있습니다.
- MessageFilterDisplay: [0f227e5](https://github.com/ros-visualization/rviz/commit/0f227e5fa7fd1c38389479936545157733b5b571)
- Image related displays (image and camera displays): [ffdfad0](https://github.com/ros-visualization/rviz/commit/ffdfad011c399cd3a9598d5be7bef26d46d9f6b3)
- DepthCloudDisplay: [2dc400d](https://github.com/ros-visualization/rviz/commit/2dc400d68e2d7813f0801f0164553de4349048f8)

주의사항을 무시하고 사용하고 싶다면 다음 예제를 확인하세요.
- GridCellsDisplay: [89b51e7](https://github.com/ros-visualization/rviz/commit/89b51e7a0ddcc35f0aba25580084c66f9d90623b)
# 8. urdf
urdf 패키지는 tinyxml과 관련된 2가지 메소드를 더이상 사용하지 않습니다.
```
bool initXml(TiXmlElement * xml)
bool initXml(TiXmlDocument * xml)
```
다음과 같이 tinyxml2를 사용하도록 변경해야 합니다.
```
bool initXml(const tinyxml2::XMLElement *xml)
bool initXml(const tinyxml2::XMLDocument *xml)
```
tinyxml API는 ROS Noetic에서 완전히 제거되었습니다.
# 9. xacro
ROS Jade부터 사용된 xacro의 --inorder 옵션이 ROS Melodic부터는 디폴트로 적용됩니다. 더이상 사용되지 않는 이전 방법을 사용하려면 --legacy 옵션을 사용합니다. 이 옵션 또한 ROS Noetic에서 완전히 제거되었습니다.
