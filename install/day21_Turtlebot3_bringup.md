# 7. [ROS 1] Bringup
![](https://emanual.robotis.com/assets/images/platform/turtlebot3/software/remote_pc_and_turtlebot.png)
> **WARNING**:
>
> 1. 이 지시사항은 remote PC에서 실행하도록 의도되었습니다. 만약 지시사항을 터틀봇에서 따라하고 있다면, roscore 명령어를 터틀봇 PC에서 실행하지 마세요.
> 1. IP 주소가 알맞게 설정되어 있는지 확인하세요.
> 1. 배터리 전압이 11V보다 낮으면, 부저 알람이 지속적으로 울리며 액츄에이터가 비활성화 됩니다. 부저가 울리면 배터리를 충전하세요.

> NOTE: 이 지시사항은 `Ubuntu 16.04`, `ROS Kinetic Kame`와 `Windows 10`, `ROS Melodic Morenia`에서 테스트 되었습니다.
(번역자의 환경은 `Ubuntu 20.04`, `ROS Noetic Ninjemys`입니다.)

## 7.1. roscore 구동

**\[Remote PC\]**에서 roscore를 구동합니다.

```sh
roscore
```


## 7.2. 터틀봇3 가져오기
**\[터틀봇\]** 터틀봇3을 실행하기 위한 기본 패키지를 가져옵니다.
```
roslaunch turtlebot3_bringup turtlebot3_robot.launch
```
터틀봇 모델이 버거라면, 터미널은 아래 메시지를 표시합니다.
```
SUMMARY
========

PARAMETERS
 * /rosdistro: kinetic
 * /rosversion: 1.12.13
 * /turtlebot3_core/baud: 115200
 * /turtlebot3_core/port: /dev/ttyACM0
 * /turtlebot3_core/tf_prefix: 
 * /turtlebot3_lds/frame_id: base_scan
 * /turtlebot3_lds/port: /dev/ttyUSB0

NODES
  /
    turtlebot3_core (rosserial_python/serial_node.py)
    turtlebot3_diagnostics (turtlebot3_bringup/turtlebot3_diagnostics)
    turtlebot3_lds (hls_lfcd_lds_driver/hlds_laser_publisher)

ROS_MASTER_URI=http://192.168.1.2:11311

process[turtlebot3_core-1]: started with pid [14198]
process[turtlebot3_lds-2]: started with pid [14199]
process[turtlebot3_diagnostics-3]: started with pid [14200]
[INFO] [1531306690.947198]: ROS Serial Python Node
[INFO] [1531306691.000143]: Connecting to /dev/ttyACM0 at 115200 baud
[INFO] [1531306693.522019]: Note: publish buffer size is 1024 bytes
[INFO] [1531306693.525615]: Setup publisher on sensor_state [turtlebot3_msgs/SensorState]
[INFO] [1531306693.544159]: Setup publisher on version_info [turtlebot3_msgs/VersionInfo]
[INFO] [1531306693.620722]: Setup publisher on imu [sensor_msgs/Imu]
[INFO] [1531306693.642319]: Setup publisher on cmd_vel_rc100 [geometry_msgs/Twist]
[INFO] [1531306693.687786]: Setup publisher on odom [nav_msgs/Odometry]
[INFO] [1531306693.706260]: Setup publisher on joint_states [sensor_msgs/JointState]
[INFO] [1531306693.722754]: Setup publisher on battery_state [sensor_msgs/BatteryState]
[INFO] [1531306693.759059]: Setup publisher on magnetic_field [sensor_msgs/MagneticField]
[INFO] [1531306695.979057]: Setup publisher on /tf [tf/tfMessage]
[INFO] [1531306696.007135]: Note: subscribe buffer size is 1024 bytes
[INFO] [1531306696.009083]: Setup subscriber on cmd_vel [geometry_msgs/Twist]
[INFO] [1531306696.040047]: Setup subscriber on sound [turtlebot3_msgs/Sound]
[INFO] [1531306696.069571]: Setup subscriber on motor_power [std_msgs/Bool]
[INFO] [1531306696.096364]: Setup subscriber on reset [std_msgs/Empty]
[INFO] [1531306696.390979]: Setup TF on Odometry [odom]
[INFO] [1531306696.394314]: Setup TF on IMU [imu_link]
[INFO] [1531306696.397498]: Setup TF on MagneticField [mag_link]
[INFO] [1531306696.400537]: Setup TF on JointState [base_link]
[INFO] [1531306696.407813]: --------------------------
[INFO] [1531306696.411412]: Connected to OpenCR board!
[INFO] [1531306696.415140]: This core(v1.2.1) is compatible with TB3 Burger
[INFO] [1531306696.418398]: --------------------------
[INFO] [1531306696.421749]: Start Calibration of Gyro
[INFO] [1531306698.953226]: Calibration End
```

> **TIP**: 만약 Lidar 센서, 라즈베리파이 카메라, 인텔 리얼센스 R200이나 코어를 따로 실행하고 싶다면, 아래 명령어를 사용하세요.
> - roslaunch turtlebot3_bringup turtlebot3_lidar.launch
> - roslaunch turtlebot3_bringup turtlebot3_rpicamera.launch
> - roslaunch turtlebot3_bringup turtlebot3_realsense.launch
> - roslaunch turtlebot3_bringup turtlebot3_core.launch

> **NOTE**: 만약 `lost sync with device` 에러 메시지가 터미널 화면에 표시된다면, 센서 장치가 느슨하게 연결된 상태일 수 있습니다.

## 7.3. RViz에서 터틀봇3 불러오기
**[Remote PC]** robot state publisher를 실행하고 RViz를 구동합니다.
> **TIP**: 이 명령어를 실행하기 전에, 터틀봇의 모델 이름을 명시해야 합니다. `${TB3_MODEL}`은 모델 이름이며 `burger`, `waffle`, `waffle_pi` 중 하나입니다. export 세팅을 항상 설정하고 싶다면, [Export TURTLEBOT3_MODEL](https://emanual.robotis.com/docs/en/platform/turtlebot3/export_turtlebot3_model) 페이지를 참고하세요.

### 7.3.1. 우분투 환경
```sh
export TURTLEBOT3_MODEL=${TB3_MODEL}
roslaunch turtlebot3_bringup turtlebot3_remote.launch
```
새로운 터미널 화면을 열고 아래 명령어를 입력하세요.
```sh
rosrun rviz rviz -d `rospack find turtlebot3_description`/rviz/model.rviz
```

### 7.3.2 윈도우 환경
```sh
set TURTLEBOT3_MODEL=${TB3_MODEL}
roslaunch turtlebot3_bringup turtlebot3_remote.launch
```
새로운 터미널 화면을 열고 아래 명령어를 입력하세요.
```sh
rosrun rviz rviz -d "<full path to turtlebot3_description>/rviz/model.rviz"
```

![](https://emanual.robotis.com/assets/images/platform/turtlebot3/bringup/run_rviz.jpg)

### 7.3.3 Turtlebot3에서 ROS\_MASTER\_URI에 접근하기

Remote PC에서 11311 포트를 개방하지 않고 위 예제를 구동하면 다음과 같은 에러가 발생합니다.

```
ERROR: unable to contact ROS master at [http://192.168.0.8:11311]
The traceback for the exception was written to the log file
```

우분투 환경을 사용하고 있다면, Remote PC에서 아래 명령어를 입력하여 포트 11311을 개방해줍니다.

```
sudo iptables -I INPUT 1 -p tcp --dport 11311 -j ACCEPT
```

윈도우 환경이라면, 아래 게시글을 따라하시되 포트 80이 아닌, 포트 11311을 대상으로 진행하시면 됩니다.

[https://teachertri.tistory.com/51](https://teachertri.tistory.com/51)