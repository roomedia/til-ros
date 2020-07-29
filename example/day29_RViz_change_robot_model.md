# 서론

로보티즈에서 제공하는 예제만 이용해서는 다른 로봇 모델을 불러오거나 커스텀 설정을 할 수 없습니다. 이번 시간에는 독립심을 기르기 위해 RViz 파일을 직접 구성해보도록 하겠습니다.

# RViz 구성

RViz를 실행시켰을 때 초기 화면은 다음과 같습니다.

![##_Image|kage@ehmbBb/btqF7LWpcKX/jNYKONFUj1Lwm6wGyf3tj1/img.png|alignLeft|data-origin-width="0" data-origin-height="0" data-ke-mobilestyle="widthContent"|||_##](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FehmbBb%2FbtqF7LWpcKX%2FjNYKONFUj1Lwm6wGyf3tj1%2Fimg.png)

다음은 로보티즈에서 제공하는 `turtlebot3_gmapping.rviz`의 화면 구성입니다.

![##_Image|kage@xmdM8/btqF80FBCom/PbAapUerKFjxXRMbei42L0/img.png|alignLeft|data-origin-width="0" data-origin-height="0" data-ke-mobilestyle="widthContent"|||_##](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxmdM8%2FbtqF80FBCom%2FPbAapUerKFjxXRMbei42L0%2Fimg.png)

# RobotModel

`RobotModel`은 주어진 경로에서 .stl, .dae 혹은 .obj 형식의 3D 모델링 파일을 받아 RViz 화면에 표시해줍니다. 이때 모델은 정확한 위치와 자세에 표시되는데 이는 자세 추정 값인 `/tf` 토픽을 사용한 결과입니다. 표시될 모델은 URDF에 정의되어 있습니다.

[http://wiki.ros.org/rviz/DisplayTypes/RobotModel](http://wiki.ros.org/rviz/DisplayTypes/RobotModel)

RViz에서 사용되는 모델은 `rosrun turtlebot3_bringup turtlebot3_remote.launch`을 실행할 때 결정됩니다.

```xml
  <!-- in turtlebot3_remote.launch -->
  <include file="$(find turtlebot3_bringup)/launch/includes/description.launch.xml">
    <arg name="model" value="$(arg model)" />
  </include>
```

해당 파일을 따라가보면 urdf 파일을 불러오는 모습을 볼 수 있습니다.

```xml
<launch>
  <arg name="model"/>
  <arg name="urdf_file" default="$(find xacro)/xacro --inorder '$(find turtlebot3_description)/urdf/turtlebot3_$(arg model).urdf.xacro'" />
  <param name="robot_description" command="$(arg urdf_file)" />
</launch>
```

위 파일은 모델을 설정하고 모델에 따른 urdf.xacro 파일을 읽어 `xacro`를 이용하여 include 된 모든 파일을 하나의 xml로 합칩니다. 해당 파일을 확인해볼까요?

```xml
<?xml version="1.0" ?>
<!-- http://ros.org/wiki/xacro에 서술되어 있는 속성을 네임스페이스로 사용합니다. -->
<robot name="turtlebot3_burger" xmlns:xacro="http://ros.org/wiki/xacro">
  <!-- 공통되는 부분 -->
  <xacro:include filename="$(find turtlebot3_description)/urdf/common_properties.xacro"/>
  <!-- gazebo를 위한 xacro 파일 -->
  <xacro:include filename="$(find turtlebot3_description)/urdf/turtlebot3_burger.gazebo.xacro"/>

  <!-- <link> 태그에는 관성(inertial), 모양(visual), 충돌(collision) 속성을 갖는 강체(rigid body)를 기술합니다. -->
  <link name="base_footprint"/>

  <!-- <joint> 태그에는 운동학(kinematics), 역학(dynamics)을 기술하고 safety limit를 지정합니다. -->
  <joint name="base_joint" type="fixed">
    <!-- <parent>: (required) 상위 <link> 이름. 상위 <link>가 움직이면 하위 <link>는 따라서 움직입니다. -->
    <parent link="base_footprint"/>화
    <!-- <child>: (required) 하위 <link> 이름 -->
    <child link="base_link"/>
    <!-- <origin>: (optional) 물체의 원점. 중심? 회전 축? -->
    <!--   xyz: (optional) 기본 값은 0을 가리키는 벡터입니다. 단위는 미터를 사용합니다. -->
    <!--   rpy(roll, pitch, yaw): (optional) 기본 값은 0을 향하는 벡터입니다. 순서대로 x, y, z축 고정 후 회전한 값입니다. 단위는 라디안입니다. -->
    <origin xyz="0.0 0.0 0.010" rpy="0 0 0"/>
    <!-- <axis>: (optional) 기본 (1, 0, 0)입니다. 회전 관절의 회전 축입니다. -->
    <!--   xyz: (required) 정규화된 벡터여야 합니다. -->
  </joint>

  <!-- <link> 태그는 관성, 모양, 충돌 속성을 갖는 강체(rigid body)를 표현합니다. -->
  <link name="base_link">
    <!-- <visual>: (optional) 물체의 형태를 나타내는 속성의 집합입니다. -->
    <visual>
      <!-- <origin>: (optional) <joint>에서와 마찬가지로 물체의 중심(회전 축?)을 설정합니다. -->
      <origin xyz="-0.032 0 0.0" rpy="0 0 0"/>
      <!-- <geometry>: (required) box/cylinder/sphere/mesh 중 하나의 태그를 포함해야 합니다. -->
      <geometry>
        <!-- <mesh>: (optional) 기본 도형 이외의 커스텀 도형. filename에 경로 명시, scale로 크기 조정합니다. -->
        <!-- 텍스쳐와 색상을 가장 잘 지원하는 형식은 Collada .dae입니다. 파일은 항상 로컬에 있어야 합니다. -->
        <mesh filename="package://turtlebot3_description/meshes/bases/burger_base.stl" scale="0.001 0.001 0.001"/>
      </geometry>
      <!-- <material>: (optional) 재질을 나타냅니다. -->
      <!--   <color>: (optional) 각각 빨강/초록/파랑/투명도를 나타내는 rgba로 나타냅니다. 각각의 범위는 [0, 1]입니다. -->
      <!--             e.g. <color rgba="0.4 0.4 0.4 1.0"/> -->
      <!--   <texture>: (optional) filename 속성으로 텍스쳐 파일 또한 불러올 수 있습니다. -->
      <!-- 여기서는 name 속성을 이용하여 common_properties에 정의한 light_black 재질을 불러왔습니다. -->
      <material name="light_black"/>
    </visual>

    <!-- <collision>: (optional) 충돌 판정을 위한 영역을 설정합니다. name 속성을 사용하여 이름을 지정할 수 있습니다. -->
    <collision>
      <!-- <origin>: (optional) -->
      <origin xyz="-0.032 0 0.070" rpy="0 0 0"/>
      <!-- <geometry>: (optional) 충돌 판정 시에는 간소화 된 매쉬를 사용합니다. -->
      <geometry>
        <!-- size: 가로 세로 높이 -->
        <box size="0.140 0.140 0.143"/>
      </geometry>
    </collision>

    <!-- <inertial>: (optional) 기본 값은 질량 0, 관성 0입니다. -->
    <inertial>
      <!-- <link>의 <origin>에 대한 상대 좌표입니다. -->
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <!-- <mass>: 실제 질량 -->
      <mass value="8.2573504e-01"/>
      <!-- <inertia>: 3x3 관성 행렬. symmetric한 부분을 제외한 6개만 지정합니다. -->
      <inertia ixx="2.2124416e-03" ixy="-1.2294101e-05" ixz="3.4938785e-05"
               iyy="2.1193702e-03" iyz="-5.0120904e-06"
               izz="2.0064271e-03" />
    </inertial>
  </link>

  <joint name="wheel_left_joint" type="continuous">
      <!-- ... -->
  </joint>

  <link name="wheel_left_link">
    <!-- ... -->
  </link>

  <!-- ... -->

</robot>

```

[ros.org/wiki/xacro에](http://ros.org/wiki/xacro%EC%97%90)

`<link>` 다음에 `<joint>`가, 다음에 다시 `<link>`가 이어지는 걸 볼 수 있으며, `<joint>`는 자신의 앞뒤로 연결된 `<link>`를 연결하고 있어, 보기 편합니다.

[http://wiki.ros.org/urdf/XML](http://wiki.ros.org/urdf/XML)

![##_ImageGrid|kage@u9hUN/btqF7KXAcnF/DMdkYC5Dkc73ASEWBIjrkk/img.png,kage@bcksMI/btqF64a9mM4/wgZtDE08PIMNChC0kPObb0/img.png|data-origin-width="0" data-origin-height="0" style="width: 61.5761%; margin-right: 10px;",data-origin-width="0" data-origin-height="0" style="width: 37.2611%;"|_##](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fu9hUN%2FbtqF7KXAcnF%2FDMdkYC5Dkc73ASEWBIjrkk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbcksMI%2FbtqF64a9mM4%2FwgZtDE08PIMNChC0kPObb0%2Fimg.png)

위 링크에서 설명하고 있는 링크와 조인트에 대한 설명 그림

다행히 이 모든 걸 직접 작성하지 않고, Gazebo에서 설정하는 게 가능합니다. 특정 모델을 우클릭 > Edit Model을 눌러 Model Editor에 진입합니다. Insert 메뉴에서 mesh 등을 추가할 수 있으며, Model 메뉴에서 선택한 모델의 Link, Joint를 설정할 수 있습니다.

![##_ImageGrid|kage@XcXVR/btqF9B6surR/1IeAXk8b4GaOfwYCoExlRk/img.png,kage@CqSof/btqF6Dx7Dht/VdX0uip5BdjiRAnCaRpLy1/img.png,kage@dmOhMe/btqF6DZeo3g/pt0ygMgYTLHJDkoHWrB0s0/img.png|data-origin-width="0" data-origin-height="0" style="width: 32.7902%; margin-right: 10px;",data-origin-width="0" data-origin-height="0" style="width: 31.4519%; margin-right: 10px;",data-origin-width="0" data-origin-height="0" style="width: 33.4324%;"|_##](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FXcXVR%2FbtqF9B6surR%2F1IeAXk8b4GaOfwYCoExlRk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FCqSof%2FbtqF6Dx7Dht%2FVdX0uip5BdjiRAnCaRpLy1%2Fimg.png)
￼
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdmOhMe%2FbtqF6DZeo3g%2Fpt0ygMgYTLHJDkoHWrB0s0%2Fimg.png)

save as로 폴더를 선택한 뒤 저장하면, 하위 폴더 아래에 `model.config`와 `model.sdf`가 생성됩니다. `model.sdf`는 이전에 확인한 urdf 형식과 유사합니다...만, 아래 링크에 따르면 유사하지만 다른 형식으로 이해하여야 할 것 같습니다. urdf는 모델 자체만 기술하지만, sdf는 모델을 둘러싼 시뮬레이션 환경까지 기술하기 때문입니다. 그렇다면 sdf에서 우리가 원하는 부분만 뽑아오면 되겠죠?

[https://newscrewdriver.com/2018/07/31/ros-notes-urdf-vs-gazebo-sdf/](https://newscrewdriver.com/2018/07/31/ros-notes-urdf-vs-gazebo-sdf/)

[http://gazebosim.org/tutorials?tut=ros\_urdf](http://gazebosim.org/tutorials?tut=ros_urdf)

어찌되었든 `robot_description` 파라미터를 원하는 urdf로 변경하면 로봇 모델을 변경할 수 있습니다. 아래 코드를 원하는 이름의 .urdf.xacro 파일로 저장합니다.

```xml
<?xml version="1.0" ?>
<robot name="turtlebot3_burger" xmlns:xacro="http://ros.org/wiki/xacro">
  <xacro:include filename="$(find turtlebot3_description)/urdf/common_properties.xacro"/>
  <link name="a">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <box size="0.1 0.2 0.1"/>
      </geometry>
      <material name="light_black"/>
    </visual>
  </link>
</robot>
```

다음 명령어로 `/robot_description`을 재설정합니다.

```sh
rosparam set /robot_description "$(xacro model2.sdf)"
```

![##_Image|kage@O1nCc/btqF7thu8mM/mD2ko7vkkV4ddRzWGXOiU1/img.png|alignCenter|data-origin-width="0" data-origin-height="0" data-ke-mobilestyle="widthContent"|||_##](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FO1nCc%2FbtqF7thu8mM%2FmD2ko7vkkV4ddRzWGXOiU1%2Fimg.png)