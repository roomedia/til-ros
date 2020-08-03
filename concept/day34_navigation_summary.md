# 서론
터틀봇에서 네비게이션을 실행하면 어떤 과정을 거칠까? 책을 보며 다시 정리해봤습니다. 저번에도 혼자 궁리해봤지만.. 좀더 자세히 알아왔습니다.

# 구동
```sh
# remote pc
roscore &
roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map.yaml
# rviz는 알아서 실행됩니다.
```

```sh
# turtlebot pc
roslaunch turtlebot3_bringup turtlebot3_robot.launch
```

# 내비게이션에 필요한 정보
- 오도메트리(/odom, nav_msgs/Odometry): 오도메트리 정보는 국부 이동 계획 (local path planning)을 생성하거나 장애물을 회피하는 데에 사용됩니다. `local_planner`가 수신합니다.
- 좌표 변환(/tf, tf/tfMessage): 로봇의 센서는 각 로봇의 구성에 따라 위치가 다르기 때문에 ROS에서는 tf 상대 좌표계를 사용합니다. 오도메트리 정보로 로봇의 위치를 파악할 수 있고, 센서의 위치는 로봇의 위치로부터 일정 x, y, z 떨어저 있다고 볼 수 있기 때문에 주어진 로봇 모델과 `/odom`으로부터 `base_footprint`, `base_link`, `base_scan` 순으로 좌표를 얻을 수 있습니다. 퍼블리시 된 `/tf` 토픽은 `move_base` 노드가 서브스크라이브 합니다.
- 거리 센서(/scan, sensor_msgs/LaserScan or sensor_msgs/PointCloud): 센서로부터 측정된 거리 값, `costmap`을 계산할 때 사용됩니다.
- 지도(/map, map_server): SLAM을 통해 생성한 점유 격자 지도(occupancy grid map) `map.pgm`, `map.yaml`을 `map_server` 노드를 통해 퍼블리시 합니다. `global_costmap`을 계산할 때 사용합니다.
- 목표 좌표(/move_base_simple/goal, geometry_msgs/PoseStamped): 사용자가 직접 지정하는 목적지 좌표와 포즈입니다. 2차원 좌표 x, y와 포즈 i로 구성되어 있습니다.
- 속도 명령(/cmd_vel, geometry_msgs/Twist): 최종 계산된 이동 궤적에 따라 로봇이 움직일 속도와 방향을 퍼블리시 합니다.

# 구성
`turtlebot3_navigation.launch`는 다음과 같이 구성되어 있습니다.

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

로봇 모델, `robot_state_publisher`, `map_server`, `AMCL`, `move_base` 실행 및 설정에 대한 내용이 포함되어 있습니다.

이를 다시 세부적으로 나눠보면,

- `/launch/amcl.launch.xml`: `AMCL`(Adaptive Monte Carlo Localization)의 각종 파라미터 설정 값을 담은 파일
- `/param/move_base_params.yaml`: 모션 계획을 총괄하는, `move_base` 노드의 파라미터 설정 파일
- `/param/costmap_common_params_burger.yaml`,
`/param/costmap_common_params_waffle.yaml`,
`/param/global_costmap_params.yaml`
`/param/local_costmap_params.yaml`: costmap 계산을 위한 파라미터 설정 파일. costmap은 점유 격자 지도(occupancy grid map)에서 지도의 각 픽셀을 장애물, 이동 불가 영역, 이동 가능 영역으로 나눌 때 사용되는 개념입니다.
- `/param/dwa_local_planner_params.yaml`: `dwa_local_planner`는 최종적으로 이동 명령, 즉 `/cmd_vel`을 로봇에 넘겨주는 패키지로, 위 `.launch` 파일은 해당 패키지에 대한 설정 값을 담고 있습니다.

# 내일..
내일은 미뤄놨던 내비게이션 이론편, costmap, amcl, dwa에 대한 이론을 살펴보겠습니다...