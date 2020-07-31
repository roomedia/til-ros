# 서론
SLAM을 통해 지도를 만들었으니 실제 네비게이션 작업을 해보도록 하겠습니다. 아래 명령어를 통해 네비게이션에 필요한 노드와 rviz를 실행합니다.

```sh
# Remote PC
roscore &
export TURTLEBOT3_MODEL=burger
roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map.yaml
```

```sh
roslaunch turtlebot3_bringup turtlebot3_robot.launch
```

실행되는 구문은 다음과 같습니다.

```xml
<launch>
  <!-- Arguments -->
  <arg name="model" default="$(env TURTLEBOT3_MODEL)" doc="model type [burger, waffle, waffle_pi]"/>
  <arg name="map_file" default="$(find turtlebot3_navigation)/maps/map.yaml"/>
  <arg name="open_rviz" default="true"/>
  <arg name="move_forward_only" default="false"/>

  <!-- Turtlebot3 -->
  <include file="$(find turtlebot3_bringup)/launch/turtlebot3_remote.launch">
    <arg name="model" value="$(arg model)" />
  </include>

  <!-- Map server -->
  <node pkg="map_server" name="map_server" type="map_server" args="$(arg map_file)"/>

  <!-- AMCL -->
  <include file="$(find turtlebot3_navigation)/launch/amcl.launch"/>

  <!-- move_base -->
  <include file="$(find turtlebot3_navigation)/launch/move_base.launch">
    <arg name="model" value="$(arg model)" />
    <arg name="move_forward_only" value="$(arg move_forward_only)"/>
  </include>

  <!-- rviz -->
  <group if="$(arg open_rviz)"> 
    <node pkg="rviz" type="rviz" name="rviz" required="true"
          args="-d $(find turtlebot3_navigation)/rviz/turtlebot3_navigation.rviz"/>
  </group>
</launch>
```

로봇 `urdf` 모델을 불러오고, `tf2` 정보를 퍼블리시 하는 `turtlebot3_remote.launch`, 이전에 작성한 지도를 불러오는 `map_server` 노드, 확률 기반 위치 추정 노드 `amcl`에 터틀봇에 알맞는 변수를 설정하여 실행하는 `amcl.launch`, 모션 계획에 필요한 costmap 파라미터와 이동속도를 넘겨주는 `move_base.launch`, 마지막으로 `open_rviz` 값에 따라 `rviz`를 실행합니다. 이 중 `amcl.launch`와 `move_base.launch`의 코드를 더 살펴보도록 하겠습니다.

```xml
<!-- amcl.launch -->
<launch>
  <!-- Arguments -->
  <arg name="scan_topic"     default="scan"/>
  <arg name="initial_pose_x" default="0.0"/>
  <arg name="initial_pose_y" default="0.0"/>
  <arg name="initial_pose_a" default="0.0"/>

  <!-- AMCL -->
  <node pkg="amcl" type="amcl" name="amcl">
    <!-- 파티클 필터 관련 파라미터 -->
    <param name="min_particles"             value="500"/>
    <param name="max_particles"             value="3000"/>
    <param name="kld_err"                   value="0.02"/>
    <param name="update_min_d"              value="0.20"/>
    <param name="update_min_a"              value="0.20"/>
    <param name="resample_interval"         value="1"/>
    <param name="transform_tolerance"       value="0.5"/>
    <param name="recovery_alpha_slow"       value="0.00"/>
    <param name="recovery_alpha_fast"       value="0.00"/>
    <param name="initial_pose_x"            value="$(arg initial_pose_x)"/>
    <param name="initial_pose_y"            value="$(arg initial_pose_y)"/>
    <param name="initial_pose_a"            value="$(arg initial_pose_a)"/>
    <param name="gui_publish_rate"          value="50.0"/>

    <!-- 거리 센서 파라미터 -->
    <!-- 센서 토픽명 변경 -->
    <remap from="scan"                      to="$(arg scan_topic)"/>
    <param name="laser_max_range"           value="3.5"/>
    <param name="laser_max_beams"           value="180"/>
    <param name="laser_z_hit"               value="0.5"/>
    <param name="laser_z_short"             value="0.05"/>
    <param name="laser_z_max"               value="0.05"/>
    <param name="laser_z_rand"         독     value="0.5"/>
    <param name="laser_sigma_hit"           value="0.2"/>
    <param name="laser_lambda_short"        value="0.1"/>
    <param name="laser_likelihood_max_dist" value="2.0"/>
    <param name="laser_model_type"          value="likelihood_field"/>

    <!-- 오도메트리 관련 파라미터 -->
    <param name="odom_model_type"           value="diff"/>
    <param name="odom_alpha1"       터        value="0.1"/>
    <param name="odom_alpha2"               value="0.1"/>
    <param name="odom_alpha3"               value="0.1"/>
    <param name="odom_alpha4"               value="0.1"/>
    <param name="odom_frame_id"             value="odom"/>
    <param name="base_frame_id"             value="base_footprint"/>
  </node>
</launch>
```

```xml
<!-- move_base.launch -->
<launch>
  <!-- Arguments -->
  <arg name="model" default="$(env TURTLEBOT3_MODEL)" doc="model type [burger, waffle, waffle_pi]"/>
  <arg name="cmd_vel_topic" default="/cmd_vel" />
  <arg name="odom_topic" default="odom" />
  <arg name="move_forward_only" default="false"/>

  <!-- move_base -->
  <node pkg="move_base" type="move_base" respawn="false" name="move_base" output="screen">
    <param name="base_local_planner" value="dwa_local_planner/DWAPlannerROS" />
    <!-- global costmap -->
    <rosparam file="$(find turtlebot3_navigation)/param/costmap_common_params_$(arg model).yaml" command="load" ns="global_costmap" />
    <!-- local costmap -->
    <rosparam file="$(find turtlebot3_navigation)/param/costmap_common_params_$(arg model).yaml" command="load" ns="local_costmap" />
    <!-- local costmap -->
    <rosparam file="$(find turtlebot3_navigation)/param/local_costmap_params.yaml" command="load" />
    <!-- global costmap -->
    <rosparam file="$(find turtlebot3_navigation)/param/global_costmap_params.yaml" command="load" />
    <rosparam file="$(find turtlebot3_navigation)/param/move_base_params.yaml" command="load" />
    <!-- 이동 명령을 로봇에게 넘기는 패키지 -->
    <rosparam file="$(find turtlebot3_navigation)/param/dwa_local_planner_params_$(arg model).yaml" command="load" />
    <!-- 토픽명 변경 -->
    <remap from="cmd_vel" to="$(arg cmd_vel_topic)"/>
    <remap from="odom" to="$(arg odom_topic)"/>
    <param name="DWAPlannerROS/min_vel_x" value="0.0" if="$(arg move_forward_only)" />
  </node>
</launch>

```

rviz 파일의 모습은 다음과 같습니다. 몇 차례 2D Pose Estimation을 해준 뒤, 2D Pose Navigation을 통해 목적지 위치와 포즈를 설정해주면 터틀봇이 이동합니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbPVGbr%2FbtqGcg27JJP%2F14k0muePN6jYLJpg7Fk600%2Fimg.png)

RViz를 사용하지 않고도 터틀봇을 이동시키는 방법은 간단합니다. `geometry_msgs/PoseStamped` 형태의 `/move_base_simple` 메시지로 원하는 좌표와 포즈를 설정해주면 됩니다.

```sh
rostopic pub -1 /move_base_simple/goal geometry_msgs/PoseStamped -- '[0, [0, 0], "map"]' '[[0, 0.2, 0], [0, 0, 0, 1]]'
```

해당 명령어를 실행하면 RViz에도 좌표와 포즈가 표시됩니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcNvCiI%2FbtqGbqkQXrm%2FDehLJCekQLCowN9kkaastK%2Fimg.png)

좌표는 지도 안에 표시할 수 있어야 하며, 포즈는 쿼터니언 형식을 따라야합니다. 다음 시간에는 이를 이용하여 웹 메뉴판을 누르면 터틀봇이 지정한 좌표까지 이동하도록 만들어 보겠습니다.
