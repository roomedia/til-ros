# Turtlebot3 Bringup 코드 분석
`turtlebot3_robot.launch`와 `turtlebot3_remote.launch`, `model.rviz`가 무엇을 하는 파일인지는 알았으나, 정확히 어떤 매커니즘인지 알아보기 위해 코드를 까보겠습니다.

```xml
<!-- in Turtlebot PC -->
<!-- turtlebot3_robot.launch -->
<launch>
  <arg name="multi_robot_name" default=""/>
  <arg name="set_lidar_frame_id" default="base_scan"/>

  <include file="$(find turtlebot3_bringup)/launch/turtlebot3_core.launch">
    <arg name="multi_robot_name" value="$(arg multi_robot_name)"/>
  </include>
  <include file="$(find turtlebot3_bringup)/launch/turtlebot3_lidar.launch">
    <arg name="set_frame_id" value="$(arg set_lidar_frame_id)"/>
  </include>

  <node pkg="turtlebot3_bringup" type="turtlebot3_diagnostics" name="turtlebot3_diagnostics" output="screen"/>
</launch>
```

launch 파일은 `turtlebot3_core.launch`, `turtlebot3_lidar.launch`를 실행하며, `turtlebot3_diagnostics`라는 상태 진단 노드를 실행하네요. 변수를 설정하고 그걸 다른 launch 파일에 넘겨주는 모습도 보입니다.

```xml
<!-- turtlebot3_core.launch -->
<launch>
  <arg name="multi_robot_name" default=""/>

  <node pkg="rosserial_python" type="serial_node.py" name="turtlebot3_core" output="screen">
    <param name="port" value="/dev/ttyACM0"/>
    <param name="baud" value="115200"/>
    <param name="tf_prefix" value="$(arg multi_robot_name)"/>
  </node>
</launch>
```

core에서는 `rosserial_python`을 이용하여 시리얼 통신을 진행합니다. 

```py
# rosserial_python/nodes/serial_node.py
# ...
client = SerialClient(port_name, baud, fix_pyserial_for_test=fix_pyserial_for_test)
client.run()
# ...
```

serial_node는 port_name 값에 따라 시리얼 서버, 클라이언트가 될 수 있지만, 여기서는 클라이언트가 선택됩니다.

시리얼 서버 장치는 `/dev/ttyACM0` = OpenCR입니다. `turtlebot3_core.launch`는 OpenCR로부터 정보를 수신하는 역할을 합니다.

LiDAR 장치의 포트는 `/dev/ttyUSB0`로, `turtlebot3_lidar.launch`를 확인하면 알 수 있습니다.

```xml
<!-- turtlebot3_lidar.launch -->
<launch>
  <arg name="set_frame_id" default="base_scan"/>

  <node pkg="hls_lfcd_lds_driver" type="hlds_laser_publisher" name="turtlebot3_lds" output="screen">
    <param name="port" value="/dev/ttyUSB0"/>
    <param name="frame_id" value="$(arg set_frame_id)"/>
  </node>
</launch>
```

```cpp
// hls_lfcd_lds_driver/src/hlds_laser_publisher.cpp
// ...
ros::Publisher laser_pub = n.advertise<sensor_msgs::LaserScan>("scan", 1000);
ros::Publisher motor_pub = n.advertise<std_msgs::UInt16>("rpms",1000);
// ...
```

`turtlebot3_lidar.launch`가 실행하는 `hlds_laser_publisher` 노드는 레이저 스캔 데이터와 uint16 형식의 모터 rpm을 퍼블리싱하고 있네요. 레이저 스캔 데이터는 무엇을 포함하고 있을까요?

```
# sensor_msgs/LaserScan.msg
Header header            # uint32  sequence_id
                         # time    stamp
                         # string  frame_id
              
float32 angle_min        # 측정 시작 각도 [rad]
float32 angle_max        # 측정 종료 각도 [rad]
float32 angle_increment  # 각도 증가량 [rad]

float32 time_increment   # 측정 간 시간 증가량 [seconds]
                         # 스캐너가 움직이고 있다면 3d 좌표를 보간하는 데에 사용됩니다.

float32 scan_time        # 측정 시간 [seconds]

float32 range_min        # 최소 범위 값 [m]
float32 range_max        # 최대 범위 값 [m]

float32[] ranges         # 범위 데이터 [m] (Note: range_min보다 작거나 range_max보다 큰 값은 무시됩니다.)
float32[] intensities    # 레이저 세기 데이터 [장치의 고유 단위]
                         # 장치가 레이저 세기를 지원하지 않는다면 빈 배열로 유지
```

LiDAR는 시리얼 통신을 통해 각도와 측정 시간, 범위를 출력하는 모양입니다. 다음으로 `turtlebot3_remote.launch`를 분석해보겠습니다.

```xml
<!-- turtlebot3_remote.launch -->
<launch>
  <arg name="model" default="$(env TURTLEBOT3_MODEL)" doc="model type [burger, waffle, waffle_pi]"/>
  <arg name="multi_robot_name" default=""/>

  <include file="$(find turtlebot3_bringup)/launch/includes/description.launch.xml">
    <arg name="model" value="$(arg model)" />
  </include>

  <node pkg="robot_state_publisher" type="robot_state_publisher" name="robot_state_publisher">
    <param name="publish_frequency" type="double" value="50.0" />
    <param name="tf_prefix" value="$(arg multi_robot_name)"/>
  </node>
</launch>
```

마찬가지로 인자를 설정하고 `description.launch.xml`과 `robot_state_publisher`를 실행합니다. 특이하게도 .launch가 아니라 .xml 형식을 사용하였지만, 사실 확장자는 상관없이 같은 동작을 합니다.

```xml
<launch>
  <arg name="model"/>
  <arg name="urdf_file" default="$(find xacro)/xacro --inorder '$(find turtlebot3_description)/urdf/turtlebot3_$(arg model).urdf.xacro'" />
  <param name="robot_description" command="$(arg urdf_file)" />
</launch>
```

xacro(XML Macro)로 실행할 urdf 파일을 설정하고, 이를 `robot_description` 파라미터의 커맨드로 설정하네요. xacro 파일은 로봇의 joint, visual, collision을 설정합니다. 이 과정에서 화면에 그려질 로봇의 모델링 파일(.stl)을 불러옵니다. 해당 파일은 너무 길기 때문에 아래 링크로 대체합니다. 다음 게시글에선 xacro 파일 형식에 대해 다뤄야겠네요.

https://github.com/ROBOTIS-GIT/turtlebot3/blob/master/turtlebot3_description/urdf/turtlebot3_burger.urdf.xacro

`robot_state_publisher` 노드는 `tf` 변환 라이브러리를 사용하여 로봇의 모든 관절 위치, 상태 등을 퍼블리시 해줍니다.

`model.rviz` 파일을 살펴보겠습니다. rviz 확장자이지만 yaml 형식을 사용하며, 아래 파일을 읽어 GUI로 시각화 해줍니다.

```yaml
Panels:
  - Class: rviz/Displays
    Help Height: 78
    Name: Displays
    Property Tree Widget:
      Expanded:
        - /Global Options1
        - /Status1
        - /Grid1
        - /Axes1
        - /LaserScan1
        - /RobotModel1
        - /TF1
      Splitter Ratio: 0.6029411554336548
    Tree Height: 787
  - Class: rviz/Selection
    Name: Selection
  - Class: rviz/Tool Properties
    Expanded:
      - /2D Pose Estimate1
      - /2D Nav Goal1
      - /Publish Point1
    Name: Tool Properties
    Splitter Ratio: 0.5886790156364441
  - Class: rviz/Views
    Expanded:
      - /Current View1
    Name: Views
    Splitter Ratio: 0.5
  - Class: rviz/Time
    Experimental: false
    Name: Time
    SyncMode: 0
    SyncSource: LaserScan
Visualization Manager:
  Class: ""
  Displays:
    - Alpha: 0.5
      Cell Size: 1
      Class: rviz/Grid
      Color: 160; 160; 164
      Enabled: true
      Line Style:
        Line Width: 0.029999999329447746
        Value: Lines
      Name: Grid
      Normal Cell Count: 0
      Offset:
        X: 0
        Y: 0
        Z: 0
      Plane: XY
      Plane Cell Count: 10
      Reference Frame: <Fixed Frame>
      Value: true
    - Class: rviz/Axes
      Enabled: true
      Length: 0.5
      Name: Axes
      Radius: 0.009999999776482582
      Reference Frame: <Fixed Frame>
      Value: true
    - Alpha: 1
      Autocompute Intensity Bounds: true
      Autocompute Value Bounds:
        Max Value: 10
        Min Value: -10
        Value: true
      Axis: Z
      Channel Name: intensity
      Class: rviz/LaserScan
      Color: 255; 0; 0
      Color Transformer: FlatColor
      Decay Time: 0
      Enabled: true
      Invert Rainbow: false
      Max Color: 255; 255; 255
      Max Intensity: 5932
      Min Color: 0; 0; 0
      Min Intensity: 106
      Name: LaserScan
      Position Transformer: XYZ
      Queue Size: 10
      Selectable: true
      Size (Pixels): 3
      Size (m): 0.009999999776482582
      Style: Flat Squares
      Topic: /scan
      Unreliable: false
      Use Fixed Frame: true
      Use rainbow: true
      Value: true
    - Alpha: 1
      Class: rviz/RobotModel
      Collision Enabled: false
      Enabled: true
      Links:
        All Links Enabled: true
        Expand Joint Details: false
        Expand Link Details: false
        Expand Tree: false
        Link Tree Style: Links in Alphabetic Order
        base_footprint:
          Alpha: 1
          Show Axes: false
          Show Trail: false
        base_link:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
        base_scan:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
        caster_back_link:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
        wheel_left_link:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
        wheel_right_link:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
      Name: RobotModel
      Robot Description: robot_description
      TF Prefix: ""
      Update Interval: 0
      Value: true
      Visual Enabled: true
    - Class: rviz/TF
      Enabled: true
      Frame Timeout: 15
      Frames:
        All Enabled: true
        base_footprint:
          Value: true
        base_link:
          Value: true
        base_scan:
          Value: true
        caster_back_link:
          Value: true
        imu_link:
          Value: true
        odom:
          Value: true
        wheel_left_link:
          Value: true
        wheel_right_link:
          Value: true
      Marker Scale: 0.4000000059604645
      Name: TF
      Show Arrows: true
      Show Axes: true
      Show Names: true
      Tree:
        odom:
          base_footprint:
            base_link:
              base_scan:
                {}
              caster_back_link:
                {}
              imu_link:
                {}
              wheel_left_link:
                {}
              wheel_right_link:
                {}
      Update Interval: 0
      Value: true
  Enabled: true
  Global Options:
    Background Color: 108; 108; 108
    Fixed Frame: base_footprint
    Frame Rate: 30
  Name: root
  Tools:
    - Class: rviz/Interact
      Hide Inactive Objects: true
    - Class: rviz/MoveCamera
    - Class: rviz/Select
    - Class: rviz/FocusCamera
    - Class: rviz/Measure
    - Class: rviz/SetInitialPose
      Topic: /initialpose
    - Class: rviz/SetGoal
      Topic: /move_base_simple/goal
    - Class: rviz/PublishPoint
      Single click: true
      Topic: /clicked_point
  Value: true
  Views:
    Current:
      Class: rviz/XYOrbit
      Distance: 1.910049319267273
      Enable Stereo Rendering:
        Stereo Eye Separation: 0.05999999865889549
        Stereo Focal Distance: 1
        Swap Stereo Eyes: false
        Value: false
      Focal Point:
        X: -0.02010004222393036
        Y: -0.467065691947937
        Z: -4.76837158203125e-7
      Focal Shape Fixed Size: false
      Focal Shape Size: 0.05000000074505806
      Name: Current View
      Near Clip Distance: 0.009999999776482582
      Pitch: 0.7953976392745972
      Target Frame: <Fixed Frame>
      Value: XYOrbit (rviz)
      Yaw: 4.793606281280518
    Saved: ~
Window Geometry:
  Displays:
    collapsed: false
  Height: 1056
  Hide Left Dock: false
  Hide Right Dock: true
  QMainWindow State: 000000ff00000000fd0000000400000000000001560000039bfc0200000008fb0000001200530065006c0065006300740069006f006e00000001e10000009b0000005c00fffffffb0000001e0054006f006f006c002000500072006f007000650072007400690065007302000001ed000001df00000185000000a3fb000000120056006900650077007300200054006f006f02000001df000002110000018500000122fb000000200054006f006f006c002000500072006f0070006500720074006900650073003203000002880000011d000002210000017afb000000100044006900730070006c00610079007301000000270000039b000000c600fffffffb0000002000730065006c0065006300740069006f006e00200062007500660066006500720200000138000000aa0000023a00000294fb00000014005700690064006500530074006500720065006f02000000e6000000d2000003ee0000030bfb0000000c004b0069006e0065006300740200000186000001060000030c00000261000000010000010f0000039bfc0200000003fb0000001e0054006f006f006c002000500072006f00700065007200740069006500730100000041000000780000000000000000fb0000000a0056006900650077007300000000270000039b0000009e00fffffffb0000001200530065006c0065006300740069006f006e010000025a000000b200000000000000000000000200000490000000a9fc0100000001fb0000000a00560069006500770073030000004e00000080000002e100000197000000030000073f0000003efc0100000002fb0000000800540069006d006501000000000000073f0000024400fffffffb0000000800540069006d00650100000000000004500000000000000000000005e30000039b00000004000000040000000800000008fc0000000100000002000000010000000a0054006f006f006c00730100000000ffffffff0000000000000000
  Selection:
    collapsed: false
  Time:
    collapsed: false
  Tool Properties:
    collapsed: false
  Views:
    collapsed: true
  Width: 1855
  X: 65
  Y: 24
```

.rviz 파일은 직접 작성할수도 있지만, RViz에서 GUI로 작업하는 방법이 더욱 편리하기 때문에 각 항목에 대해 특별히 설명이 필요하진 않을 것 같습니다.

해당 파일에서는 LaserScan, 로봇 모델, TF 등의 정보가 포함되어 있다는 걸 알 수 있습니다.