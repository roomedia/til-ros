# SLAM and Navigation Theory
## What is Navigation
모바일 로봇에서 Navigation은 로봇이 정해진 목적지까지 이동하는 것을 말합니다. 이를 위해 SLAM을 통한 위치 추정, 지도가 필요한 것이며, 다양한 경로 중 최적의 경로를 찾아내고 장애물을 회피할 수 있어야 합니다. 내비게이션의 필수 요소는 다음과 같습니다.
- 지도
- 로봇 자세 계측/추정 기능
- 벽, 물체 등 장애물 계측 기능
- 최적 경로를 계산하고 주행하는 기능
## 지도 = SLAM
모바일 로봇의 지도는 인간이 제작하여 넣어주는 방법도 있지만, 로봇 스스로 센서 데이터를 이용하여 지도를 제작할 수도 있습니다. 이렇게 최소한의 노력으로 지도를 얻기 위해 SLAM을 사용합니다. SLAM은 Simultaneous Localization And Mapping, 즉 동시적 위치 추정 및 지도 작성을 말합니다. 쉽게 말하자면 로봇이 미지의 공간으로 이동하며 주변을 감지, 현재 자신의 위치를 추정하여 지도를 작성하는 방법입니다.
## 자세 계측/추정 = dead reckoning
로봇 스스로 자신의 위치, 방향을 계측하고 추정할 수 있어야 합니다. 실외 로봇의 경우에는 GPS, 그보다 더 정밀한 DGPS를 이용할 수 있지만, GPS를 사용할 수 없는 실내용 로봇의 경우 마커 인식 방식 및 실내 위치 추정 시스템 등이 도입되고 있습니다. 가장 널리 사용되는 방법은 추측 항법(dead reckoning)으로, 바퀴의 회전량을 이용하여 상대적인 위치를 추정하는 방법입니다. 이는 오차가 많이 발생하며, IMU 센서 등을 통해 관성 정보를 취득하여 오차를 줄이곤 합니다.

로봇의 위치(position)와 방향(orientation)을 통틀어 자세(pose)라고 정의합니다. 위치는 x, y, z의 3축 벡터로, 방향은 x, y, z, w의 4축 쿼터니언 방식을 사용합니다. 쿼터니언 형태의 방향을 사용하면 x, y, z 3축을 사용할 때 발생하는 짐벌락 현상을 막을 수 있습니다. 짐벌락은 특정 축이 회전했을 때 두 축이 겹쳐진다면, 한 축이 쓸모없어지는 현상을 말합니다.

로봇이 Te라는 짧은 시간 이동하였을 때, 바퀴의 회전 속도를 v, 엔코더 값을 E라고 했을 때, 모터 회전량을 통해 바퀴 회전속도를 구하는 식은 다음과 같습니다.

![left](https://latex.codecogs.com/png.latex?v_{left}&space;=&space;{(E_{left\\_current}&space;-&space;E_{left\\_past})&space;\over&space;T_e}&space;\cdot&space;{\pi&space;\over&space;180}&space;(radian&space;/&space;sec))

![right](https://latex.codecogs.com/png.latex?v_{right}&space;=&space;{(E_{right\\_current}&space;-&space;E_{right\\_past})&space;\over&space;T_e}&space;\cdot&space;{\pi&space;\over&space;180}&space;(radian&space;/&space;sec))

바퀴의 이동 속도를 V, 바퀴의 반지름을 r이라 했을 때 V는 다음과 같이 구할 수 있습니다.

![velocity](https://latex.codecogs.com/png.latex?V&space;=v\cdot&space;r(meter/sec))

양 바퀴의 이동 속도를 이용하면 병진속도와 회전속도를 구할 수 있습니다. 바퀴 간 거리는 D로 정의합니다.

![linear velocity](https://latex.codecogs.com/png.latex?v_{k}={V_{right}+V_{left}\over2}(meter/sec))

![angular velocity](https://latex.codecogs.com/png.latex?w_{k}={V_{right}-V_{left}\over&space;D}(radian/sec))

마지막으로 이들 값을 이용하여 로봇의 이동 후 위치 및 방향을 구할 수 있습니다.

![delta s](https://latex.codecogs.com/png.latex?\Delta{s}=v_kT_e)
![delta theta](https://latex.codecogs.com/png.latex?\Delta\theta=w_kT_e)

![x k+1](https://latex.codecogs.com/png.latex?x_{(k+1)}=x_k+\Delta{s}\cos(\theta_k+{\Delta\theta\over2}))

![y k+1](https://latex.codecogs.com/png.latex?y_{(k+1)}=y_k+\Delta{s}\sin(\theta_k+{\Delta\theta\over2}))

![theta k+1](https://latex.codecogs.com/png.latex?\theta_{(k+1)}=\theta_k+\Delta\theta)

이 연산을 매 단위 시간 반복합니다...
## 장애물 계측 = LiDAR, Kinect, ...
벽, 물체 등 장애물을 파악하기 위해 레이저 기반 거리 센서, 초음파 센서, 적외선 거리 센서 등이 사용되며 비전을 위해 다양한 카메라 및 RealSense, Kinect 등 Depth camera를 사용합니다.
## 최적 경로 계산 및 주행
최적 경로를 계산하기 위한 알고리즘으로는 A*, 포텐셜 필드, 파티클 필터, RRT(Rapidly-exploring Random Tree) 등이 있습니다.

RRT 알고리즘이 A* 알고리즘보다 실행 속도는 훨씬 빠르지만, 계산된 경로가 A* 알고리즘과 비교했을 때 더 길다는 단점이 있습니다. 또한, 이러한 알고리즘은 거리 측정 방법(Manhattan distance, Euclidean distance)에 따라 영향을 받기 때문에 지속적으로 연구되고 있습니다.
[https://www.researchgate.net/publication/280094486_A_Comparison_Between_RRT_and_A_Algorithms_for_Motion_Planning_in_Complex_Environments](https://www.researchgate.net/publication/280094486_A_Comparison_Between_RRT_and_A_Algorithms_for_Motion_Planning_in_Complex_Environments)
### Manhattan Distance
Manhattan Distance는 L1 Distance라고도 불리며, 공식은 아래와 같습니다.

![manhattan distance](https://latex.codecogs.com/png.latex?|x_1-x_2|+|y1-y2|)

### Euclidean Distance
Euclidean Distance는 L2 Distance라고 불리며, 피타고라스 정리를 이용하여 구할 수 있습니다.

![euclidean distance](https://latex.codecogs.com/png.latex?\sqrt{(x_2-x_1)^2+(y2-y1)^2}) 

아래 그림에서 빨강, 파랑, 노랑 경로는 모두 Manhattan Distance로 표현된 거리이며, 초록색 경로는 Euclidean Distance를 통해 표현된 거리입니다.
 
![Manhattan and Euclidean Distance](https://upload.wikimedia.org/wikipedia/commons/thumb/0/08/Manhattan_distance.svg/200px-Manhattan_distance.svg.png)
