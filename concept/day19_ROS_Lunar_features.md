# 마이그레이션 가이드
ROS Lunar Loggerhead에서 아래와 같은 패키지가 변경되었고, 이에 따라 마이그레이션할 때 주의할 점과 튜토리얼을 제공합니다.

Jade나 그 이전 버전 사용자라면, ROS Kinetic 마이그레이션 가이드를 먼저 보고 오십시오: [kinetic/Migration](http://wiki.ros.org/kinetic/Migration)

# 1. roscpp
## 1.1 Multiple spinners on the same queue
Kinetic에서 동일한 큐에 여러 spinner를 사용하면 이러한 작동은 더이상 제공되지 않으며 이전 버전과의 호환성을 위해서만 존재한다는 경고 메시지가 표시되었습니다. 이러한 지원은 Lunar에서 완전히 제거되었습니다. 이러한 코드를 사용하면 이제 예외가 발생합니다. 더 알아보려면 [ros/ros_comm#988](https://github.com/ros/ros_comm/pull/988)를 확인하세요.

# 2. rosout
## 2.1 로그 파일 교체 방식
로그 파일이 교체되는 방식이 흔히 사용되는, 기존 파일 이름에 하나씩 증가하는 숫자를 붙이고, 기존과 같은 이름을 사용하여 로그 파일을 작성하도록 변경되었습니다. 이전까지 ROS의 방식은 다른 프로젝트와 일치하지 않았으며, 기존 파일의 이름을 변경하지 않고 새로운 파일에 로그를 작성하였습니다. 더 알아보려면 [ros/ros_comm#854](https://github.com/ros/ros_comm/pull/854)를 확인하세요.

## 2.2 \`rosout.log` 파일
`rosout.log` 파일이 이제 가독성을 위해 노드 이름을 포함합니다. 더 알아보려면 [ros/ros_comm#912](https://github.com/ros/ros_comm/pull/912)를 확인하세요.

# 3. rosgraph
## 3.1 Python 로깅 구성
`ROS_PYTHON_LOG_CONFIG_FILE`이 이제 [python logging configuration dictionary schema](https://docs.python.org/2/library/logging.config.html#configuration-dictionary-schema)를 따라 yaml 형식을 지원합니다. 더 알아보려면 [ros/ros_comm#1061](https://github.com/ros/ros_comm/pull/1061)을 확인하세요.

# 4. robot_model
## 4.1 Metapackage Deprecation
메타 패키지 `robot_model`은 Lunar에서 더이상 사용되지 않으며, ROS M에는 릴리즈 되지 않습니다. 내부 패키지는 새로운 레포지토리로 옮겨져 유지보수 될 것입니다. `robot_model`를 의존하는 모든 패키지는 메타 패키지 내부 패키지를 의존하도록 변경하여야 합니다.