# SLAM 패키지
SLAM 관련 패키지로는 gmapping, cartographer, rtabmap이 많이 사용됩니다.

## Gmapping
- Gmapping은 OpenSLAM에 공개된 SLAM의 한 종류로, ROS 패키지로 제공되고 있습니다.
- Rao-Blackwellized 파티클 필터를 사용하고 있으며, 파티클 수가 감소되었고, 그리드 맵을 제공하고 있습니다.
- X, Y, Theta 속도 이동 명령을 받도록 되어있어 두 축의 구동 모터로 이루어져 좌/우측 바퀴를 따로 구동하는 로봇이나 3개 이상의 구동 휠을 가지고 있어야 합니다.
- 주행기록계(Odometry)가 반드시 있어 자신이 이동한 거리, 회전량을 추측 항법(dead reckoning)으로 계산할 수 있거나, 이에 준하는 자세 보상, 혹은 위치 계측 및 추정 수단이 있어야 합니다.
- 또한 XY 평면상의 장애물을 계측할 수 있도록 LDS(Laser Distance Sensor), LRF(Laser Range Finder), LiDAR(Light Detection And Ranging) 등이 있어야 합니다. 3차원 Depth Camera를 사용할 수도 있지만, 해당 데이터를 다시 2차원 평면 정보로 변환하기 때문에 적합하지 않습니다.
- 로봇 형태에도 제약 조건이 있습니다. 정다각형, 직사각형, 원형 로봇이 적합합니다. 기타 한쪽 축이 길게 나온 변형 로봇, 문 사이로 나가지 못할 정도로 큰 로봇, 휴머노이드, 다관절 이동 로봇, 비행 로봇 등은 제외합니다. (휴머노이드, 다관절의 경우 자세 계측이 어려워 그러지 않을까)
https://index.ros.org/p/openslam_gmapping/github-ros-perception-openslam_gmapping/#noetic

### 계측 환경
GMapping은 센서를 이용해 맵을 생성하기 때문에 다음과 같은 환경에 적합하지 않습니다.
- 장애물 하나 없는 정사각 형태의 환경
- 장애물 없는 평행한 복도
- 레이저 및 적외선이 반사되지 못하는 유리창
- 레이저 및 적외선이 산란하는 거울
- 호수, 바닷가 기타 물가

### Algorithm
GMapping은 측정값 z_1:t = z_1, ..., z_t와 Odometry 측정값 u_2:t = u_1, ..., u_t가 주어졌을 때 p(x_1:t, m | z_1:t, u_2:t)를 찾는 알고리즘입니다. (m은 그리드 맵을 의미합니다.)

### Rao-Blackwellized Particle Filter
각각의 파티클 = 샘플 포즈 기록 + 주어진 샘플 포즈 기록에서 지도에 대한 사후 확률로 구성됩니다.
자세한 내용은 이해가 가질 않네요..
[https://people.eecs.berkeley.edu/~pabbeel/cs287-fa11/slides/gmapping.pdf](https://people.eecs.berkeley.edu/~pabbeel/cs287-fa11/slides/gmapping.pdf)

### 결과물
GMapping의 결과물은 2차원 점유 격자 지도(OGM, Occupancy Grid Map) 형태로 생성됩니다. 흰색은 로봇이 이동 가능한 free area, 흑색은 occupied area, 회색은 unknown area를 뜻합니다. 색상을 추가하여 충돌 위험 지역 같은 추가적인 정보를 표현할 수 있습니다.

## Cartographer
Cartographer는 구글이 제작한 SLAM 기법으로, 2D 및 3D 매핑을 지원합니다. 그리드 방식의 매핑과 Ceres 기반 Scan Matcher를 이용하여 환경을 재구성 합니다. Cartographer는 ROS2에서 동작하며, 세 패키지 중 유일하게 ROS2를 지원하는 패키지입니다. GMapping이 CCBL 라이센스를 사용하고 있어 상업용으로 사용하기 어려운만큼, Google Cartographer는 Apache 2.0 라이센스를 사용하고 있고, 지속적인 유지 보수가 되고 있는만큼 현재의 대세라고 볼 수 있습니다.

### Scan Matcher
Raw Odometry만을 이용하여 지도를 제작할 수 없는 이유는, 벽이나 장애물에 가로막혀 동작에 노이즈가 있을 수 있고, 센서값이 부정확할 수 있으며, 포즈 추정이 잘못될 수 있기 때문입니다.

이런 이유로 Scan-matching 기법이 발달하게 되었습니다. Scan-matching 기법은 스캔 데이터와 맵, 아니면 두 종류의 센서를 이용한 스캔 데이터나 두 개의 맵 데이터가 주어졌을 때 데이터의 모양을 변형하지 않고, 이동, 회전만을 사용하여 가장 잘 정렬할 수 있는 방법을 찾는 기법입니다. 

![https://miro.medium.com/max/700/1*BbjhHKV6q1ThXDUG2MXZ1w.png](https://miro.medium.com/max/700/1*BbjhHKV6q1ThXDUG2MXZ1w.png)

Scan Matcher는 접근법에 따라 세 가지로 분류될 수 있습니다.
- Beam Sensor Model: sensor beam full reading과 맵을 매칭합니다.
- Likelihood Field Model: sensor beam endpoints와 likelihood field를 매칭합니다. 
- Map Matching Model: local map과 global map을 매칭합니다.
- Iterative Closest Points: sensor beam endpoints 간 비교합니다.

### Beam Sensor Model
측정 노이즈는 정규 분포를 따르며, 예측되지 않은 장애물을 만날 확률은 다음과 같습니다. z_t는 t 시점의 센서 측정값, x는 00, m은 map을 의미합니다. 

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FBUYkQ%2FbtqFAPSO9l9%2FhIvBizhqnkKzuzU9oD8YbK%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FBUYkQ%2FbtqFAPSO9l9%2FhIvBizhqnkKzuzU9oD8YbK%2Fimg.png)

랜덤 측정하였을 때 모든 값은 동일하게 분포하며, Max Range는 다음과 같습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Ft12oi%2FbtqFz2SQU4C%2FPqsi6qR3krKHEDha71Oo40%2Fimg.png)

모든 확률을 합치면 아래처럼 됩니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbsvzZc%2FbtqFAfEp4kF%2F4wUOvHd7JBfJkkTMRk4kkK%2Fimg.png)

Beam Sensor Model은 x_t 값이 부드럽지 않고, 모든 센서 값을 읽은 뒤 ray-cast를 연산하여야 하기 때문에 연산량이 큽니다.

### Likelihood Field Model
Likelihood Field Model은 Beam Endpoint Model 또는 Scan-based Model이라 불리며, Beam Sensor Model의 smoothness와 연산량 문제를 해결하기 위해 개발되었습니다. 센서 구조의 제너레티브 모델에 따라 조건부 확률을 고려하지 않습니다. 모든 센서 값을 고려하는 대신 end-point만 고려하자는 아이디어에서 시작되었으며, 가능도 p(z | x_t, m)은 아래와 같이 주어집니다.

![likelihood](https://latex.codecogs.com/png.latex?{1\over\sqrt{2\pi}\sigma}\exp(-{d^2\over2\sigma^2}))

d는 end-point와 가장 가까운 장애물의 거리입니다. 아래와 같은 알고리즘을 통해 2D 그리드에 대한 likelihood field를 미리 계산할 수 있습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fc8SBXH%2FbtqFz17vSS5%2FJSqnpCt1Q5D1rNu1xFZ8ZK%2Fimg.png)

Likelihood Field Model의 경우 움직이는 물체에 대한 모델링이 되지 않으며, 말단 값만을 사용하여 센서가 벽을 뚫고 보는 것처럼 취급합니다. 탐사되지 않은 지역에 대해 적용이 불가능하다는 단점이 있습니다. 이런 상황에는 p(z_t | x_t, m) = I / z_max로 두어 통제합니다.

두 개의 스캔을 매칭할 수도 있습니다. 첫 번째 스캔에서 likelihood field를 추출한 뒤 이를 다음 스캔에서 사용하는 방식입니다. 이런 방식을 사용할 경우, 2D 테이블만을 사용하여 매칭을 진행하기 때문에 효율이 높고, smooth 문제가 해결됩니다. gradient descent를 적용할 수 있으나 센서의 물리적인 특성은 무시한다는 단점이 있습니다.

### Map Matching Model
Map Matching Model은 센서 데이터로부터 작은 local map을 생성하여 global map에 매칭하는 방식입니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fc7xs1m%2FbtqFAPFfmJw%2FLicsjuJjcUPKYBSfFLGlKK%2Fimg.png)
포기
