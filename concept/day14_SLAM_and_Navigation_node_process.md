# 저번 시간 요약
SLAM 설명 -> 논문 -> 으악! 너무너무 어렵다 -> 그렇다면 그대로 쓰시고 좀더 시간을 원하는 부분에 투자하세요..

# SLAM 노드의 처리 과정
gmapping을 사용한 SLAM이 실제 로봇에서 어떻게 적용되는지 알아보겠습니다.
![https://t1.daumcdn.net/cfile/tistory/99492E4E5B8E24DA24](https://t1.daumcdn.net/cfile/tistory/99492E4E5B8E24DA24)
## 노드
1. sensor_node
2. turtlebot3_teleop
3. turtlebot3_core
4. slam_gmapping
5. mapping_server
## 과정
1. 로봇을 구동시키면 먼저 센서에서 측정한 2D 평면 상의 거리 정보가 raw data가 메시지로 전달됩니다. 이때 토픽 이름을 scan이라고 해보겠습니다. (임의 지정 가능)
2. turtlebot3_teleop 노드는 실제 로봇(여기서는 터틀봇)의 core 노드에 병진 속도, 회전 속도 값을 전달합니다. 로봇은 이 정보를 바탕으로 이동합니다.
3. 구동시킨 뒤 로봇의 왼쪽, 오른쪽 바퀴 엔코더 값을 통해 바퀴의 위치를 측정할 수 있고, imu 값을 더해 보정할 수 있습니다. odometry 값을 이용해 이동 거리를 알 수 있습니다. 이러한 정보를 사용하여 상대 좌표로 이루어진 base_footprint(로봇의 최상단점), base_link(로봇의 중심점), base_scan(센서의 위치)를 구할 수 있고, 이는 다시 tf 메시지로 gmapping 노드에 전달됩니다.
4. gmapping이 지도를 만들어내고, 이를 map_server를 통해 저장할 수 있습니다. 저장된 맵은 pgm 형식의 portable gray map, yaml 문서로 표현됩니다.

# 위치 추정 (localization)
## Kalman filter
- 가장 많이 쓰이는 위치 추정 방법입니다. 다양한 발전된 알고리즘 중 확장 칼만 필터가 많이 사용됩니다.
- 가우시안 잡음이 포함되어 있는 선형 시스템에서 대상체의 상태를 추적하는 재귀 필터입니다.
- 베이즈 확률 기반으로 예측과 보정을 반복적으로 실행하면서 현재 위치를 추정해나가는 알고리즘입니다.
- 예측(prediction): 모델을 상정하고 이 모델을 이용하여 이전 상태로부터 현재 상태를 예측합니다.
- 보정(update): 앞 단계의 예측 값과 외부 계측기로 얻은 실제 측정 값 간의 오차를 이용하여 더욱 정확한 상태의 값을 추정하도록 보정합니다.
- 칼만 필터는 선형 시스템과 가우시안 잡음이 적용되는 시스템에서만 정확도가 보장됩니다.
## Particle filter
- 파티클 필터는 시행 착오 기법을 기반으로 한 시뮬레이션 예측 기술로 대상 시스템에 확률 분포로 임의 생성된 추정값을 파티클 형태로 나타낸다.
- 확률 초기화를 거친 후 예측, 보정을 통해 위치를 추정하고, 재추출하며 확률을 수렴해나가는 과정입니다.
- 파티크 필터는 샘플의 개수가 충분하다면 칼만 필터를 개선한 EKF나 UKF보다 위치 추정이 정확하지만, 샘플의 개수가 충분치 않으면 정확도가 떨어집니다.
- 이런 부분을 해결하기위해 파티클 필터와 칼만 필터를 동시에 사용하는 방법인 RBPF(Rao-Blackwellized Particle Filter) 기반의 SLAM이 일반적으로 사용됩니다. gmapping 또한 이러한 방법을 사용하고 있습니다.
- 더 자세히 알고 싶다면 세바스찬 스론의 Probabilistic Robotics를 읽어보자!고 하시네요...

# 내비게이션
## Dynamic Window Approach(DWA)
- local plan에서 주로 사용하는 기법으로, 로봇의 속도 탐색 영역(Velocity Search Space)에서 로봇과 충돌 가능한 장애물을 회피하면서 목표점까지 빠르게 다다를 수 있는 속도를 선택하는 방법입니다.
- 기존의 지도를 위치 기반으로 접근하던 방식에서 지도 자체를 속도 영역으로 접근합니다.
![Dynamic Window Approach](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FclExdw%2FbtqFDxE5M69%2FR4H1JIbdSCFWxfidRBuRrK%2Fimg.png)
- v(병진속도), w(회전속도)
- V_s: 가능 속도 영역
- V_a: 허용 속도 영역
- V_r: 다이나믹 윈도우 안의 속도 영역
- ![목적함수](https://latex.codecogs.com/png.latex?G(v,w)=\sigma(\alpha\cdot&space;heading(v,w)+\beta\cdot&space;dist(v,w)+\gamma\cdot&space;velocity(v,w)))
- 목적함수 G는 로봇의 방향, 속도, 충돌을 고려하여, 목적함수가 최대가 되는 속도 v,w를 구하게 됩니다.
- 로봇 이동에 실제 사용되는 병진속도, 회전속도를 이용하기 때문에 좌표를 전달하는 것보다 효율적입니다.

# 내비게이션 노드의 실행 과정
## 노드
1. /odom, nav_msgs/Odometry: 오도메트리 정보를 받아 local path를 만들거나 장애물 회피에 사용합니다.
1. /tf, tf/tfMessage: 이전에 보았던 odom -> base_footprint -> base_link -> base_scan의 변환을 거쳐 토픽을 퍼블리시 합니다. mobe_base 노드가 이 정보를 받아 로봇의 위치와 센서의 위치를 가지고 이동 경로를 계획합니다.
1. 거리 센서 /scan, sensor_msgs/LaserScan or sensor_msgs/PointCloud: LDS, Kinect, Xtion 등을 사용하여 측정한 거리 raw data이며 로봇 위치 추정 방법인 AMCL을 이용하여 로봇의 현재 위치 추정, 로봇의 모션 계획에 사용됩니다.
1. 지도 /map, nav_msgs/GetMap: 내비게이션에서는 occupancy grid map을 사용합니다. RViz 상에서 붉게 나타나는 부분은 로봇이 이 붉은 지점에 도달할 수록 충돌할 확률이 커진다는 의미입니다.
1. 목표 좌표 /move_base_simple/goal, geometry_msgs/PoseStamped: 사용자가 직접 지정하는 좌표로, RViz 상에서 지정할 수 있습니다. 2차원 좌표 x,y와 자세 i로 구성되어 있습니다. 로봇은 x, y 좌표로 이동한 뒤 자세를 맞추게 됩니다.
1. 속도 명령 /cmd_vel, geometry_msgs/Twist: 최종적으로 계획된 이동 궤적에 따라 로봇을 움직이는 속도 명령을 퍼블리시하여 로봇을 목적지까지 이동시킵니다.
