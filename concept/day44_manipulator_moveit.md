# MoveIt?
`MoveIt!`은 매니퓰레이터를 위한 통합 라이브러리입니다. 모션 플래닝을 위한 빠른 역기구학 해석, 매니퓰레이션을 위한 고급 알고리즘, 로봇 핸드 제어, 동역학, 제어기 등 다양한 기능을 제공하고 있으며, `URDF`와 `SRDF`를 통해 로봇에 대한 정보를 입력 받습니다.

`RViz` 플러그인 형태의 GUI 툴을 제공하기 때문에 학습이 쉬우며, C++ 기반 move_group interface, Python 기반 move_commander, RViz Motion Planning Plugin 이외에도 OMPL, KDL, FCL, RRT 등 다양한 오픈소스 기반 라이브러리를 제공합니다.

# MoveIt! 구조
`MoveIt!`에는 SPA(Sensing-Plan-Action) 기능이 모두 갖춰져 있습니다.

![MoveIt 구조도](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqRLyI%2FbtqGIIEOdKB%2FH2JUVKSsbUpI7roiQ7VeS1%2Fimg.png)

- 센싱
  - 관절 위치
  - TF
  - 3D sensing
- 계획
  - FK, IK
  - Grasping
  - 충돌 회피
  - 모션 계획
- 실행
  - 위치/속도 제어
  - Pick and Place

이러한 MoveIt! 패키지 생성은 `MoveIt! Setup Assistant`에 `URDF` 파일을 입력하여 손쉽게 완료할 수 있습니다.

# MoveIt! 실행하기
다음 명령어로 `MoveIt! Setup Assistant`를 실행합니다.

```sh
roslaunch moveit_setup_assistant setup_assistant.launch
```

![initial setup](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbUwM0t%2FbtqGH2KfIy3%2F5LPFKNazIbUVKiSwAinKpK%2Fimg.png)

다음과 같은 화면이 표시되면 `Create New MoveIt Configuration Package`를 클릭합니다.

![select browse](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbDMNz2%2FbtqGH1YRSrA%2FPJGbvU0PSBG4vlEJezBAJ0%2Fimg.png)

Browse 버튼을 눌러 urdf, xacro 등 로봇 정보가 입력된 파일을 가져옵니다. `Open Manipulator`의 xacro 파일을 가져와보겠습니다. 가져온 뒤에는 Load Files를 눌러줍니다.

![browse](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FenexyW%2FbtqGK3VVX0e%2FBLiRxeUbkpjrUWahXWOsZK%2Fimg.png)

![Load Files](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FyWDb7%2FbtqGDPSS1DB%2FffKbv5TF3D3cWNX8jOpNe0%2Fimg.png)

Self-Collisions에서 Generate Collision Matrix를 눌러주면 다음과 같이 맞닿은 링크, 절대 만나지 않는 링크 등을 미리 계산합니다. Density가 커질수록 복잡도가 증가합니다.

![Self Collision](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FKwV5M%2FbtqGNCDL0oC%2F3G4FsZpBaeeH25iiK7aiKK%2Fimg.png)

Virtual Joint는 사용되지 않았으므로 생략, 바로 Planning Group을 설정하도록 하겠습니다. Add Group을 누르고, 다음과 같이 Arm과 Gripper로 구분하여 그룹을 생성합니다. Kinetic Solver는 다양한 알고리즘을 선택 가능하지만 지금은 KDL을 사용하겠습니다.

![Arm](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqkeaL%2FbtqGFaoZURD%2FW24mmeIwUu1VBUdsVRiLu1%2Fimg.png)

![Arm2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcx0hHS%2FbtqGK4ggVSc%2FNuMMOsjHKtTLHa52i6Ijbk%2Fimg.png)

위와 같이 Joint 1, 2, 3, 4를 넣어 Arm을 만듭니다.

![Gripper](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdDsiKg%2FbtqGHrXLZjm%2Fg1SRIFUnUFvL78b4Qameg1%2Fimg.png)

마찬가지로 gripper와 gripper_sub를 넣은 그룹 Gripper를 생성합니다.

![Pose](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdDsiKg%2FbtqGHrXLZjm%2Fg1SRIFUnUFvL78b4Qameg1%2Fimg.png)

Robot Pose에서 로봇이 취할 다양한 포즈를 설정할 수 있습니다. Add Pose를 누른 뒤 자세를 변경하고 해당 자세에 이름을 붙입니다.

![pose1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FexBxDr%2FbtqGHxqr6o3%2FOkNRQIhLYuGqDHPPMJJk0k%2Fimg.png)

![pose2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbDIm2k%2FbtqGGGA8lG0%2FemOGdAABaalaGoSkMEM6s0%2Fimg.png)

End Effectors는 말단 장치의 이름과 로봇 그룹, Parent Link를 설정해줍니다. 사진을 못 찍었네요ㅎㅎ; 실제 동작하지 않는, Passive Joint가 있다면 설정해주시면 되고, 마지막으로 저자 정보를 등록해주시면 됩니다. 저자 이름과 이메일이 등록되지 않으면 패키지가 생성되지 않습니다.

![passive joint](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb32nF8%2FbtqGNF8mNLi%2FrAofW9zV7I2kxX8Fi9rPuK%2Fimg.png)

![author](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdK9onL%2FbtqGIp6hhmr%2FGj0pIwZ29VnSGsq384bdp0%2Fimg.png)

마지막으로 패키지 생성 경로를 설정해주시면 되는데, 경로는 ~/catkin_ws/src 아래에 새로운 폴더를 하나 생성하셔서 설정해주시면 됩니다.

![package](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbrIsCU%2FbtqGK4tNe7Y%2FPIEh3YddcaHHOtu37uY2R0%2Fimg.png)

![fin](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FA5Q2i%2FbtqGJvLWFeX%2Fn7Ji6ew3uhanFlGsYoFT70%2Fimg.png)

다음 명령어를 통해 OpenManipulator 데모를 실행해볼 수 있습니다.

```
roslaunch open_manipulator_moveit demo.launch
```

![moveit](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F1K9kQ%2FbtqGHLWhzP4%2FmMVWoPDbHMnSCZajFCqiE0%2Fimg.png)

RViz가 실행되지만, 평소와는 다른 모습입니다.