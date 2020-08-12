# 41일차 - 매니퓰레이션 소개 및 URDF 작성법
# 목표
- 매니퓰레이터 모델링 (URDF 작성법)
- MoveIt! 사용법

# 매니퓰레이터 소개
![매니퓰레이터 유형](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcYNGF7%2FbtqGuGPjpwR%2FaDWvSUKPYejvhvxnRVjUZK%2Fimg.png)
수평다관절형, 수직다관절형이 특히 많이 사용되며, 직교좌표형 형태는 3D 프린터 등에서 사용되는 형태입니다. 이번 시간에는 수직다관절형을 다루겠습니다.

![매니퓰레이터 예시](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fk4zlq%2FbtqGAwxaBN4%2Fpj4COkPcU7Z9A7buxieBJk%2Fimg.png)
위 매니퓰레이터는 6축으로 움직일 수 있는 관절(조인트)을 가지며, 조인트와 조인트를 연결하는 링크가 있습니다.

# 매니퓰레이터 제어
매니퓰레이터 제어는 세 가지 부분으로 나눌 수 있으며, 각각의 핵심 역할은 다음과 같습니다.

- 센싱(Sensing):
  - 위치, 토크
  - 주변 사물
- 계획(Plan):
  - 수동 / 자동
  - 충돌 회피
  - 파지(Grasping)
  - 궤적 생성
- 실행(Action):
  - 위치, 속도, 힘 제어

# 매니퓰레이터를 위한 ROS 툴
- URDF(Unified Robot Description Format): 로봇 모델링을 위한 시각화 툴, XML로 손쉽게 제작 가능
- Gazebo: 실제 동작 환경을 구현할 수 있는 3D 시뮬레이터. 플러그인을 사용하여 여러 센서에서 데이터를 수집하거나 로봇 제어, ROS-CONTROL 기능 수행 가능
- MoveIt!: 매니퓰레이터를 위한 통합 라이브러리. KDL(Kinematics and Dynamics Library)과 OMPL(The Open Motion Planning Library) 등의 오픈 라이브러리 제공. 충돌 계산, 모션 플래닝, Pick and Place 데모처럼 매니퓰레이터의 여러 기능을 쉽게 사용 가능

# 매니퓰레이터의 구성 요소
- base: 매니퓰레이터가 고정되어 있는 부분
- link: base, joint, end effector를 연결해주는 강체 역할
- joint: 로봇의 회전(revolute)/병진(prismatic) 운동을 담당하는 부분
- end effector: 물체와 로봇의 상호작용을 담당하는 부분/장비

# joint의 종류
joint는 운동 방향에 따라 다양한 형태가 가능합니다. 각각의 조인트의 형태와 이름은 다음과 같습니다.

![joint의 유형](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcio3Zi%2FbtqGzXaYgYx%2FO1FTBBNbaktLReYM0K3Z5K%2Fimg.png)

- Revolute Joint(회전 관절): Pin Joint나 Hinge Joint라고도 하며, 경첩과 같은 회전 운동을 할 때 사용합니다.
![회전 관절의 생김새](https://www.researchgate.net/profile/Farzin_Piltan3/publication/311414943/figure/fig1/AS:435816678858753@1480918201188/Revolute-1-DOF-Joint.png)

- Prismatic Joint(병진 관절): 두 몸체 사이에서 선형 슬라이딩 운동을 할 때 사용합니다.
![병진 관절의 생김새](https://fastenerengineering.com/wp-content/uploads/2019/12/010-Prismatic-joint-scaled.jpg)

- Screw Joint(나사 관절): 회전을 통해 앞으로 나아갑니다.
![나사 관절의 생김새](https://www.finemech.com/mm5/4.24/bohlender/screw_joint.gif)

- Cylindrical Joint(원통 관절): 회전을 통해 위아래로 움직입니다. (딱풀, 립밤??)
![원통 관절의 생김새](https://upload.wikimedia.org/wikipedia/en/thumb/5/56/Cylindrical_joint.svg/1200px-Cylindrical_joint.svg.png)

- Universal Joint(혼합 관절): 2개의 축을 결합하되, 한쪽 축에서 다른 축으로 회전력을 전달합니다.
![혼합 관절의 생김새](https://i.pinimg.com/originals/3b/63/cd/3b63cd853a5e84f6c535189f1fc0d788.gif)

- Spherical Joint(구형 관절): 구의 형태로 회전 움직임이 자유로운 관절입니다.
![구형 관절의 생김새](https://www.mdpi.com/energies/energies-11-03488/article_deploy/html/images/energies-11-03488-g003a.png)

# 자유도(DOF, degrees of freedom)
물체의 위치, 포즈는 6개의 변수(x, y, z, roll, pitch, yaw)로 표현이 가능하다고 배웠습니다. 이를 6 자유도를 갖는다고 하며, 6 자유도를 갖는 물체를 조작하기 위해서는 로봇 또한 6 자유도를 가져야 합니다.

여자유도는 로봇이 6 자유도보다 많은 자유도를 가지고 있을 때를 의미합니다.

# 작업 공간과 관절 공간(work space & joint space)
- 작업 공간: 말단 장치의 위치(x, y, z)와 자세(roll, pitch, yaw)로 표현한 공간
- 관절 공간: 각 관절의 각도로 표현한 공간

즉, 작업 공간을 제시했을 때 로봇이 여기 도달하기 위해서는 작업 공간을 관절 공간으로 변환하는 작업이 필요합니다.

# 순운동학과 역운동학(Forward Kinematics & Inverse Kinematics)
로봇을 목적대로 움직이기 위해 다음 세 가지 작업이 필요합니다.
1. 기하학적 운동 계획 세우기 (Kinematics)
1. 동력학적 효과 계산 (Dynamics)
1. 각 관절에 적절한 명령 내리기

![순운동학 수식](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcfS9Ts%2FbtqGvZOnPpE%2F399kDrp8EOcQO1qrv7bcBK%2Fimg.png)
- 순운동학(FK, Forward Kinematics)
  - 로봇의 각 관절 각도가 주어졌을 때 말단 장치의 위치 및 자세를 구하는 것
  - 관절 공간 -> 작업 공간
  - 방정식의 해가 하나 나옴

![역운동학 수식](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbusZl4%2FbtqGAw400ia%2FN8s7VruDJCRE3buutIkv1k%2Fimg.png)
- 역운동학(IK, Inverse Kinematics)
  - 말단 장치의 위치 및 자세가 주어졌을 때 각 관절 각도를 구하는 것
  - 작업 공간 -> 관절 공간
  - 방정식의 해의 개수가 미정

# MoveIt!에서 필요한 정보
- 링크(link) 정보
  - 기하학적 정보
  - STL, DAE(COLLDA) 3차원 설계 파일
  - 무게(mass, kg)
  - 관성 모멘트(moments of inertia, kg * m^2) 3차원 설계 툴에서 지원하니 설계자에게 물어봐야함

- 관절(joint) 정보
  - 전/후 링크와의 관계
  - 운동 종류(회전/병진 운동)
  - 회전 및 병진 운동의 기준 축
  - 한계 값(관절에 부여되는 힘, 최대/최소 관절 값, 속도)

이 모든 정보는 `URDF(Unified Robot Description Format)` 하나로 대체할 수 있습니다. 엄밀히 말해서 MoveIt!에서 필요로 하는 정보는 URDF가 아닌 `SRDF(Semantic Robot Description Format)`이지만, MoveIt! Setup Assistant에서 URDF를 SRDF로 변환할 수 있습니다.

마찬가지로 Gazebo도 URDF가 아닌 `SDF(Simulation Description Format)`을 이용하지만 URDF에서 발전된 형태이므로 약간의 수정만 거치면 됩니다.

# 3-DOF Manipulator URDF 작성하기
![3축 매니퓰레이터 그림](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbG9I3b%2FbtqGxcUFxDx%2F0bsn5alVaDzOc5kEXcqDYK%2Fimg.png)

다음 코드를 통해 urdf 패키지에 의존하는 새로운 예제 패키지를 생성하고, 로봇의 링크, 관절 정보를 기술할 urdf 파일을 생성합니다.

```sh
cd ~/catkin_ws/src
create_catkin_pkg testbot_description urdf
cd testbot_description
mkdir urdf
cd urdf
vi testbot.urdf
```

다음 내용을 파일 안에 작성합니다. urdf 파일의 구조를 익히기 위해 따라서 타이핑 해보는 걸 추천드립니다.

```xml
<?xml version='1.0' ?>
<robot name='testbot'>
  <material name='black'>
    <color rgba='0.0 0.0 0.0 1.0'/>
  </material>
  <material name='orange'>
    <color rgba='1.0 0.4 0.0 1.0'/>
  </material>

  <!-- 기저 -->
  <link name='base'/>
  <joint name='fixed' type='fixed'>
    <!-- 일반적으로 기저에 가까운 link가 parent이다 -->
    <parent link='base'/>
    <child link='link1'/>
  </joint>

  <!-- 로봇 팔이 향할 방향을 설정하는 링크 -->
  <link name='link1'>
    <collision>
      <origin xyz='0 0 0.25' rpy='0 0 0'/>
      <geometry>
        <!-- Z축의 경우 origin을 기준으로 -0.25, +0.25 -->
        <box size='0.1 0.1 0.5'/>
      </geometry>
    </collision>
    <visual>
      <origin xyz='0 0 0.25' rpy='0 0 0'/>
      <geometry>
        <box size='0.1 0.1 0.5'/>
        <!-- STL, DAE 등의 매쉬 파일을 사용하여 표현 가능 -->
        <!-- <mesh filename='package://open_manipulator_description/meshes/chain_link1.stl' scale='0.001 0.001 0.001'/> -->
      </geometry>
      <material name='black'/>
    </visual>
    <inertial>
      <origin xyz='0 0 0.25' rpy='0 0 0'/>
      <mass value='1'/>
      <inertia ixx='1.0' ixy='1.0' ixz='0.0' iyy='1.0' iyz='0.0' izz='1.0'/>
    </inertial>
  </link>

  <joint name='joint1' type='revolute'>
    <parent link='link1'/>
    <child link='link2'/>
    <origin xyz='0 0 0.5' rpy='0 0 0'/>
    <!-- 로봇 팔이 z축을 기준으로 회전한다 -->
    <axis xyz='0 0 1'/>
    <limit effort='30' lower='-2.617' upper='2.617' velocity='1.571'/>
  </joint>

  <link name='link2'>
    <collision>
      <origin xyz='0 0 0.25' rpy='0 0 0'/>
      <geometry>
        <box size='0.1 0.1 0.5'/>
      </geometry>
    </collision>
    <visual>
      <origin xyz='0 0 0.25' rpy='0 0 0'/>
      <geometry>
        <box size='0.1 0.1 0.5'/>
      </geometry>
      <material name='orange'/>
    </visual>
    <inertial>
      <origin xyz='0 0 0.25' rpy='0 0 0'/>
      <mass value='1'/>
      <inertia ixx='1.0' ixy='0.0' ixz='0.0' iyy='1.0' iyz='0.0' izz='1.0'/>
    </inertial>
  </link>

  <joint name='joint2' type='revolute'>
    <parent link='link2'/>
    <child link='link3'/>
    <origin xyz='0 0 0.5' rpy='0 0 0'/>
    <axis xyz='0 1 0'/>
    <limit effort='30' lower='-2.617' upper='2.617' velocity='1.571'/>
  </joint>

  <link name='link3'>
    <collision>
      <origin xyz='0 0 0.5' rpy='0 0 0'/>
      <geometry>
        <box size='0.1 0.1 1'/>
      </geometry>
    </collision>
    <visual>
      <origin xyz='0 0 0.5' rpy='0 0 0'/>
      <geometry>
        <box size='0.1 0.1 1'/>
      </geometry>
      <material name='black'/>
    </visual>
    <inertial>
      <origin xyz='0 0 0.5' rpy='0 0 0'/>
      <mass value='1'/>
      <inertia ixx='1.0' ixy='0.0' ixz='0.0' iyy='1.0' iyz='0.0' izz='1.0'/>
    </inertial>
  </link>

  <joint name='joint3' type='revolute'>
    <parent link='link3'/>
    <child link='link4'/>
    <origin xyz='0 0 1' rpy='0 0 0'/>
    <axis xyz='0 1 0'/>
    <limit effort='30' lower='-2.617' upper='2.617' velocity='1.571'/>
  </joint>

  <link name='link4'>
    <collision>
      <origin xyz='0 0 0.25' rpy='0 0 0'/>
      <geometry>
        <box size='0.1 0.1 0.5'/>
      </geometry>
    </collision>
    <visual>
      <origin xyz='0 0 0.25' rpy='0 0 0'/>
      <geometry>
        <box size='0.1 0.1 0.5'/>
      </geometry>
      <material name='orange'/>
    </visual>
    <inertial>
      <origin xyz='0 0 0.25' rpy='0 0 0'/>
      <mass value='1'/>
      <inertia ixx='1.0' ixy='0.0' ixz='0.0' iyy='1.0' iyz='0.0' izz='1.0'/>
    </inertial>
  </link>
</robot>
```

`check_urdf` 명령어로 urdf가 올바르게 작성되었는지 확인합니다. `check_urdf` 명령어를 사용하기 위해선 먼저 `sudo apt-get install liburdfdom-tools`를 통해 패키지를 설치해야 합니다.

```sh
check_urdf testbot.urdf
```

이상이 없을 때의 아웃풋은 다음과 같습니다.

```
root@869b5a874518:~/catkin_ws/src/testbot_description/urdf# check_urdf testbot.urdf 
robot name is: testbot
---------- Successfully Parsed XML ---------------
root Link: base has 1 child(ren)
    child(1):  link1
        child(1):  link2
            child(1):  link3
                child(1):  link4
```

`urdf_to_graphiz testbot.urdf`를 사용하면 연결 상태를 시각화하여 `.gv`, `.pdf` 파일로 출력합니다.

![urdf_to_graphiz로 출력된 testbot.pdf](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbMipLb%2FbtqGvedUYO0%2FWKW1kSKTM53hqCqWJyQquK%2Fimg.jpg)

최종적으로 RViz를 통해 확인해봅시다. `testbot_description` 폴더에 `launch` 폴더를 만들고 해당 폴더 안에 `testbot.launch` 파일을 생성합니다.

```sh
cd ~/catkin_ws/src/testbot_description
mkdir launch
cd launch
touch 
vi testbot.launch
```

```xml
<launch>
  <!-- set args (used in params) -->
  <arg name='model' default="$(find testbot_description)/urdf/testbot.urdf"/>
  <arg name='gui' default='True'/>
  <!-- set params (used in node) -->
  <param name='robot_description' textfile='$(arg model)'/>
  <param name='use_gui' value='$(arg gui)'/>
  <!-- URDF로 설정된 로봇의 관절 상태를 sensor_msgs/JointState 형태로 publish -->
  <node pkg='joint_state_publisher' type='joint_state_publisher' name='joint_state_publisher'/>
  <!-- URDF로 설정된 로봇 정보와 sensor_msgs/JointState로 Forward Kinematics를 계산하여 tf 메시지로 publish -->
  <node pkg='robot_state_publisher' type='robot_state_publisher' name='robot_state_publisher'/>
</launch>
```

작성한 launch 파일과 RViz를 실행합니다. 함께 실행되는 `JointStatePublisher` GUI 툴을 사용하여 매니퓰레이터가 제대로 작동하는지 구부려볼 수 있습니다.

```sh
roslaunch testbot_description testbot.launch &
rviz &
```

![rviz](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbjARhi%2FbtqGyxp2k7G%2FtB8MYKw5sepkd5eTaCAKjk%2Fimg.png)

# 유의점
외형으로 보이는 모든 것은 링크(액츄에이터도 링크), 회전축만 조인트로 생각하여야 합니다.
