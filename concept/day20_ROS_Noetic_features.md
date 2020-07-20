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