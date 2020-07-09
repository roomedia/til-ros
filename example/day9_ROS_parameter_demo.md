# 파라미터 사용법
## ros_tutorials_service 노드 수정
이전에 작성하였던 `ros_tutorials_service` 노드를 덧셈만 하던 노드에서 사칙연산을 하는 노드로 수정하겠습니다. `server.cpp`를 다음과 같이 수정합니다.
```c++
#include "ros/ros.h"
#include "ros_tutorials_service/SrvTutorial.h"

enum { PLUS, MINUS, MULTIPLICATION, DIVISION };

int g_op = PLUS;

// 서비스 요청이 있을 경우, 아래의 처리를 수행
// 요청 = req / 응답 = res
bool calculation(ros_tutorials_service::SrvTutorial::Request &req, ros_tutorials_service::SrvTutorial::Response &res) {
  char c;
  switch (g_op) {
  case PLUS:
    // add and save result
    res.result = req.a + req.b;
    c = '+';
    break;
  case MINUS:
    res.result = req.a - req.b;
    c = '-';
    break;
  case MULTIPLICATION:
    res.result = req.a * req.b;
    c = '*';
    break;
  case DIVISION:
    if (req.b == 0) {
      ROS_ERROR("Zero Division Error!");
      return false;
    }
    res.result = req.a / req.b;
    c = '/';
    break;
  }
  // print info
  ROS_INFO("request: %ld %c %ld", (long int)req.a, c, (long int)req.b);
  ROS_INFO("response: %ld", (long int)res.result);
  return true;
}

int main(int argc, char** argv) {
  ros::init(argc, argv, "service_server");
  ros::NodeHandle nh;
  nh.setParam("calculation_method", PLUS);
  // service name = ros_tutorial_srv
  // callback = add
  ros::ServiceServer ros_tutorials_service_server = nh.advertiseService("ros_tutorial_srv", calculation);
  ROS_INFO("ready srv server");

  ros::Rate r(10);
  while (1) {
    nh.getParam("calculation_method", g_op);
    ros::spinOnce();
    r.sleep();
  }
  return 0;
}
```
`NodeHandle` 인스턴스에 `getParam`, `setParam` 문구가 추가되었고, 상태에 따른 사칙연산 코드가 추가되었습니다. 이를 실행해보면 이전과 다르지 않게 동작하는 것을 볼 수 있습니다. 이는 파라미터가 변경되지 않아, default 값인 PLUS가 작용한 탓입니다.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FO0pLw%2FbtqFuVe3GTi%2Fz63wP9XicRHSjYSig3ZakK%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FO0pLw%2FbtqFuVe3GTi%2Fz63wP9XicRHSjYSig3ZakK%2Fimg.png)

파라미터를 전달해보겠습니다.
```bash
rosparam set /calculation_method 0 # 덧셈
rosparam set /calculation_method 1 # 뺄셈
rosparam set /calculation_method 2 # 곱셈
rosparam set /calculation_method 3 # 나눗셈
```

전달된 파라미터에 따라 실행 시 값이 달라지는 걸 볼 수 있습니다.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbUVOsZ%2FbtqFujAux1f%2FlbhZvotiLuoMaVWYSVwO01%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbUVOsZ%2FbtqFujAux1f%2FlbhZvotiLuoMaVWYSVwO01%2Fimg.png)