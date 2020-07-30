`model2.urdf.xacro`가 링크와 관절을 포함하도록 다듬어 보겠습니다.  일부러 터틀봇과 같은 이름을 사용하였습니다.

```
<?xml version="1.0" ?>
<robot name="turtlebot3_burger" xmlns:xacro="http://ros.org/wiki/xacro">
  <xacro:include filename="$(find turtlebot3_description)/urdf/common_properties.xacro"/>

  <link name="base_footprint"/>

  <joint name="base_joint" type="fixed">
    <parent link="base_footprint"/>
    <child link="base_link"/>
    <origin xyz="0 0 0" rpy="0 0 0"/>
  </joint>

  <link name="base_link">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <box size="0.1 0.1 0.5"/>
      </geometry>
      <material name="light_black"/>
    </visual>

    <collision>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <box size="0.1 0.2 0.1"/>
      </geometry>
    </collision>
  </link>

  <joint name="top_joint" type="fixed">
    <parent link="base_link"/>
    <child link="top_link"/>
    <origin xyz="0 0 0" rpy="0 0 0"/>
  </joint>

  <link name="top_link">
    <visual>
      <origin xyz="0 0 0.5" rpy="0.2 0 0"/>
      <geometry>
        <cylinder radius="0.08" length="0.3"/>
      </geometry>
    </visual>

    <collision>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <cylinder radius="0.08" length="0.3"/>
      </geometry>
    </collision>
  </link>
</robot>
```

다음 명령어를 실행하여 해당 파일을 `robot_description` 파라미터로 설정합니다.  

```
# 마스터가 실행되고 있지 않다면
roscore &
# 파라미터 설정
rosparam set /robot_description "$(xacro model2.urdf.xacro)"
```

예제 파일처럼 RViz에서 `tf`, `LaserScan`, `image` 등 나머지 항목을 모두 추가한 모습입니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FzvgJk%2FbtqF8wednSx%2FEQNBBXwSDESWRnNsUJzbR0%2Fimg.png)

이 상태로는 `rosbag`을 실행하여도 움직이지 않습니다. RViz 상에서 `RobotModel`을 움직이기 위해선 `tf`와의 연결이 필요합니다. SLAM 예제에서 해당 부분은 `turtlebot3_remote.launch`에서 호출하며, 실제 Pose 정보를 `tf`로 퍼블리시하는 노드는 `robot_state_pulisher`입니다.

```
<!-- turtlebot3_remote.launch -->
<launch>
  <arg name="model" default="$(env TURTLEBOT3_MODEL)" doc="model type [burger, waffle, waffle_pi]"/>
  <arg name="multi_robot_name" default=""/>

  <!-- 로봇 모델을 설정합니다. -->
  <include file="$(find turtlebot3_bringup)/launch/includes/description.launch.xml">
    <arg name="model" value="$(arg model)" />
  </include>

  <!-- 각 조인트의 Pose를 TF로 퍼블리시 합니다. -->
  <node pkg="robot_state_publisher" type="robot_state_publisher" name="robot_state_publisher">
    <param name="publish_frequency" type="double" value="50.0" />
    <param name="tf_prefix" value="$(arg multi_robot_name)"/>
  </node>
</launch>
```

다음과 같이 이동하는 걸 볼 수 있습니다. 물론 저장된 터틀봇의 관절 상태가 움직이는 것이기 때문에 뼈대는 터틀봇과 일치합니다.

```
rosrun robot_state_publisher robot_state_publisher &
rosbag play -r 10 ~/slam.bag
```

![##_Image|kage@chR29u/btqF64CELQa/AVRav6rWERzPRhIfv6cknK/img.png|alignCenter|data-origin-width="0" data-origin-height="0" data-ke-mobilestyle="widthContent"|||_##](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FchR29u%2FbtqF64CELQa%2FAVRav6rWERzPRhIfv6cknK%2Fimg.png)

다음 코드를 통해 현재 불러온 모델에 대한 TF 정보를 확인할 수 있습니다.

```
rosrun rqt_tf_tree rqt_tf_tree &
```

![##_Image|kage@ejXKWA/btqF80lNqLC/secBGB3ntsTwh3Sa5tXNo1/img.png|alignCenter|data-origin-width="0" data-origin-height="0" data-ke-mobilestyle="widthContent"|||_##](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FejXKWA%2FbtqF80lNqLC%2FsecBGB3ntsTwh3Sa5tXNo1%2Fimg.png)
