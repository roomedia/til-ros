# 서론
오로카에 `dynamixel_sdk`를 사용할 수 없다는 질문글이 달려 잠시 봤습니다.. `CMakeLists.txt` 파일에 `include_directories(...)` 안에 `../../../../../include/dynamixel_sdk`와 같은 방식으로 라이브러리 폴더를 포함하고 있었습니다. ...

# 상대경로의 문제점
- 헷갈립니다.
- 배포가 힘듭니다.

# 상대경로는 헷갈린다?
물론 간단한 경우에는 상대경로를 사용하는 게 좋습니다. 해당 프로젝트의 `src/` 폴더 안에 있는 `main.cpp` 파일을 찾는다던지. 하지만 위와 같이 `../../../../../`이 들어간 순간... 지금 어느 위치에 있는 건지 헷갈리기 시작합니다.

해당 문제가 일어난 환경은 sdk의, 특정 언어 폴더의, 예제 폴더, 임의의 프로젝트와 같은 식으로 복잡한 경로였기 때문에 현재 위치를 착각하기 좋았습니다. `../../../`로 고쳐주면 작동하는 모습을 볼 수 있었습니다. 더 큰 문제는 따로 있었습니다.

# 배포의 어려움
대부분의 경우, 코드는 배포를 위해 존재합니다. 하지만 위와 같은 경우에는 배포가 까다로워집니다. 해당 프로젝트를 00 폴더 하위 00 밑에 00 폴더에 다운로드 해주세요... 이상하지 않나요? 상대경로로 설정하면 패키지 위치가 달라지는 순간 의존성이 깨집니다. 이를 막으면서도 ROS에서 DynamixelSDK를 사용할 방법을 찾아야 했습니다.

# DynamixelSDK for ROS 설치
다행히 DynamixelSDK는 ros를 지원합니다. 아니.. 당연하다고 해야 할까요?? 터틀봇을 만든 로보티즈에서 만든 제품이고, 터틀봇은 공식 ROS 교육 도구이며, 등등등... 아무튼 설치해봅시다.

```sh
cd $HOME/catkin_ws/src
git clone https://github.com/ROBOTIS-GIT/DynamixelSDK
cd ..
catkin_make
```

해당 작업을 통해 `DynamixelSDK`를 `catkin_ws`에 빌드할 수 있고, 이후 `catkin_make install` 명령어를 실행하여 완전히 설치할 수 있습니다.

# 예제
이후 사용은 이렇게 합니다.

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.0.2)
project(test2)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  dynamixel_sdk
)

catkin_package(
  CATKIN_DEPENDS
  roscpp
  dynamixel_sdk
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  #../../../include/dynamixel_sdk
)

add_executable(test2 src/main.cpp)
```

```xml
<!-- package.xml -->
<?xml version="1.0"?>
<package format="2">
  <name>test2</name>
  <version>0.0.0</version>
  <description>The test package</description>
  <maintainer email="root@todo.todo">root</maintainer>
  <license>TODO</license>

  <buildtool_depend>catkin</buildtool_depend>
  <depend>roscpp</depend>
  <depend>dynamixel_sdk</depend>
</package>
```

```cpp
// src/main.cpp
#include <cstdio>
//#include "dynamixel_sdk.h"
#include "dynamixel_sdk/dynamixel_sdk.h"

int main(void) {
  printf("hello world\n");
  return 1;
}
```

주석 부분을 해제하면 상대경로 버전으로 실행할 수 있습니다. 상대경로 버전으로 실행하려면 해당 패키지가 `~/catkin_ws/src/DynamixelSDK/c++/example/protocol1.0/` 아래에 있어야 합니다.