# 마이그레이션 가이드
ROS Noetic Ninjemys에서 아래와 같은 패키지가 변경되었고, 이에 따라 마이그레이션할 때 주의할 점과 튜토리얼을 제공합니다.

# 1. Python 3
ROS Noetic은 Python 3만을 타겟으로 합니다. ROS 패키지를 Python 2에서 Python 3로 전환하려면 [다음 안내서](http://wiki.ros.org/UsingPython3)을 읽어보세요.

# 2. author warning을 피하려면 CMake 요구 버전을 높이세요.
Debian Buster와 Ubuntu Focal에서는 CMake가 [CMP0048](https://cmake.org/cmake/help/v3.0/policy/CMP0048.html)에 의해 사용자에게 경고를 줍니다. 이 경고를 피하려면, CMake 최소 버전을 3.0.2 이상으로 높이세요. 패키지 업데이트 방법을 알고 싶다면 [RethinkRobotics-opensource/gennodejs#24](https://github.com/RethinkRobotics-opensource/gennodejs/pull/24) 예제를 확인하세요.

`CMakeLists.txt`에서 첫 번째 줄이 적어도 버전 3.0.2를 요청하도록 하세요
```
cmake_minimum_required(VERSION 3.0.2)
```

# 3. Distutils 대신 Setuptools
catkin은 [ros/catkin#1048](https://github.com/ros/catkin/pull/1048) PR부터 `setuptools`를 우선적으로 사용하며 `distutils`를 옵션으로 지원합니다. 패키지 업데이트 방법을 알고 싶다면 [ros/genlisp#17](https://github.com/ros/genlisp/pull/17) 예제를 확인하세요.

`setup.py`에서 `distutils-core`가 아닌 `setuptools`로부터 `setup`을 임포트합니다.
```python
## Replace this line in your setup.py
# from distutils.core import setup
## With this one
from setuptools import setup
```

만약 레포지토리가 여러 ROS 배포판(Melodic, Noetic, etc.) 같은 브랜치를 사용 중이라면, `package.xml`에 format 3을 사용하여 `python-setuptools`와 `python3-setuptools`에 대한 조건부 의존성을 사용하세요.

```xml
<?xml version="1.0"?>
<?xml-model
  href="http://download.ros.org/schema/package_format3.xsd"
  schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  
<!- ... ->
 
  <buildtool_depend condition="$ROS_PYTHON_VERSION == 2">python-setuptools</buildtool_depend>
  <buildtool_depend condition="$ROS_PYTHON_VERSION == 3">python3-setuptools</buildtool_depend>
```

# 4. Orocos KDL과 BFL rosdep keys 사용하기
패키지가 Orocos KDL 또는 BFL ROS 패키지에 의존하고 있다면, `package.xml`에서 이러한 의존성을 rosdep keys로 변경하여야 합니다.

Old ROS Package|Replacement rosdep keys
---|---
orocos_kdl|liborocos-kdl-dev
&nbsp;|liborocos-kdl
python_orocos_kdl|python3-pykdl
orocos_kinematics_dynamics|liborocos-kdl-dev
&nbsp;|liborocos-kdl
&nbsp;|python3-pykdl
bfl|liborocos-bfl-dev
&nbsp;|liborocos-bfl

예제: [ros/geometry2#447](https://github.com/ros/geometry2/pull/447), [ros-planning/robot_pose_ekf#6](https://github.com/ros-planning/robot_pose_ekf/pull/6)

# 5. setup.py에서 불필요한 package_dir 옵션 제거
ROS 패키지가 `setup.py`에 `package_dir={'': ''}`를 포함하고 있고 다음과 같은 빌드 에러를 겪고 있다면, `package_dir` 인자를 제거하세요. [이를 보여주는 예제](https://github.com/ros/kdl_parser/pull/36/commits/f8f6d8f9984b4666ea7ef33a757a6aa25e5a004c)

```
running install_egg_info
error: error in 'egg_base' option: '' does not exist or is not a directory
CMake Error at catkin_generated/safe_execute_install.cmake:4 (message):

  execute_process(/tmp/ws/build_isolated/kdl_parser_py/catkin_generated/python_distutils_install.sh)
  returned error code
Call Stack (most recent call first):
  cmake_install.cmake:147 (include)
```

이 옵션은 어떤 Python 패키지가 setup.py와 관련이 있는지 Python에 전달하는 역할을 합니다. `package_dir={'': ''}`는 파이썬 패키지가 setup.py와 같은 디렉토리에 있다는 의미이나, [이는 기본 실행 사항](https://docs.python.org/3.8/distutils/setupscript.html#listing-whole-packages)이기 때문에 제거해도 괜찮습니다. **만약 패키지가 src/my_python_pkg/__init__.py와 같이 다른 폴더에 있다면 제거하지 마세요.**

# 6. Joint State Publisher GUI
이전 버전의 ROS는 [joint_state_publisher](http://wiki.ros.org/joint_state_publisher) 패키지에는 `use_gui` 파라미터가 있었으며, 이를 통해 `joint_state_publisher`가 시작될 때 GUI를 실행했습니다. 이 패키지는 2020년 초 `joint_state_publisher`와 `joint_state_publisher_gui` 패키지로 분리되었습니다. `use_gui` 옵션은 Noetic에서 완전히 사라졌으며, 대신 사용자는 `joint_state_publisher_gui`를 통해 GUI를 실행해야 합니다.

# 7. RViz Python API import
Python에서 RViz를 임포트하는 모듈이 변경되었습니다. 아래처럼 작성된 모든 코드를 바꾸세요:
```python
import rviz
```
변경 후
```
from rviz import bindings as rviz
```

# 8. xacro.py 대신 xacro를 사용
더이상 사용되지 않는 `xacro.py` 실행 파일은 [xacro](http://wiki.ros.org/xacro)에서 제거되었습니다. 대신 `xacro`를 사용하세요. 여기 관련 예제가 있습니다. [ros/urdf_sim_tutorial#8](https://github.com/ros/urdf_sim_tutorial/pull/8)