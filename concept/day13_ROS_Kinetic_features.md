ROS는 버전 별로 지원하지 않는 패키지가 많습니다. 도대체 ROS의 버전 별 차이가 무엇이길래 패키지를 사용할 수 없는지, 오늘은 [https://wiki.ros.org/kinetic/Migration](https://wiki.ros.org/kinetic/Migration)를 살펴보며 차이점을 살펴보았습니다. 대부분의 강의 자료가 kinetic인만큼, jade와의 차이점은 간결하게 소개하고 넘어가도록 합니다.

1. catkin 변경 사항
find_package 사용 시 의존성 문제 해결하였고, CPATH를 사용할 수 없습니다.

2. genpy
genpy 사용 시 second, nanosecond가 반올림 되는 이슈가 수정되었습니다.

3. ROS C++
ros::Duration::sleep()이 값을 반환합니다. 실제로 sleep 되는지 확인할 수 있습니다.

4. roslaunch
roslaunch에서 파이썬 문법 $(eval <expression>)를 사용할 수 있습니다.

5. Qt4/Qt5 가이드라인
Platform vtk6 pcl OpenCV3 Gazebo7 rviz/rqt
Xenial qt5 qt5 qt5 qt4 qt5
Wily qt4 qt4 qt4 qt4 qt5
Jessie qt4 qt4 qt4 qt4 qt5

6. Gazebo
Gazebo가 5.x 버전에서 7.x 버전으로 변경되었습니다.

7. gennodejs/rosnodejs
nodejs로 메시지를 주고 받을 수 있는 패키지가 생겼습니다. roslibjs와 차이점은 communication briedge가 필요하지 않다는 점입니다.

8. robot_model
joint_state_publisher가 wxWidgets 기반에서 Qt 기반으로 변경되었습니다.

9. robot_state_publisher
JointStateListener, RobotStatePublisher의 private 속성이 public으로 변경되었고, 메소드가 virtual화 되어 서브클래스 작성이 가능해졌습니다.

10. opencv가 2.4에서 3.1로 변경되었습니다.

11. octomap이 1.7.x에서 1.8로 변경되었습니다. 커스텀 노드와 octree class 관련 변경사항이 있습니다.
