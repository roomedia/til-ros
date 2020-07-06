# ROS 도구
1. RViz: 3차원 시각화 도구
2. rqt: Qt 기반의 ROS GUI 개발 도구
3. rqt_image_view: 이미지 표시 도구
4. rqt_graph: 노드 및 메시지 간 상관 관계를 그래프로 나타내는 도구
5. rqt_plot: 2차원 데이터 플롯 도구
6. rqt_bag: GUI 기반 bag 데이터 분석 도구

## RViz
3차원 모델을 표현하고, 인터렉티브 마커를 이용하여 상호작용할 수 있습니다. 이를 통해 각각의 모델을 이동 시키거나 시뮬레이션, 제어할 수 있으며, 이외에도 키넥트, LRF, RealSense 등 다양한 센서 데이터를 3차원으로 시각화하는 역할을 합니다.
RViz는 아래 명령어로 설치할 수 있으며, `roscore`가 실행된 상태에서 다음 명령어를 이용하여 실행할 수 있습니다.
```
# install
sudo apt-get install ros-noetic-rviz
```
```
# run
rviz
# or
# rosrun rviz rviz
```

## rqt
rqt 관련 도구는 아래 명령어로 설치할 수 있습니다.
```
sudo apt-get install ros-noetic-rqt*
```
실행은 아래와 같이 하면 됩니다.
```
# run
rqt
```
`rqt_image_view`, `rqt_graph`와 `rqt_plot`은 각각 해당 이름을 터미널에 입력하여 실행할 수 있지만, 아래와 같이 `rqt`를 실행한 뒤 플러그인 메뉴에서 실행할 수도 있습니다.
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FlCu5z%2FbtqFo6BbSA7%2F88Izm5cY6sIHUTAeiwgC3K%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FlCu5z%2FbtqFo6BbSA7%2F88Izm5cY6sIHUTAeiwgC3K%2Fimg.png)
### rqt_plot
`rqt_plot`을 통해 좌표 데이터를 표기해봅시다. 먼저 `turtlesim` 노드를 실행합니다.
```
# shell 1
rosrun turtlesim turtlesim_node
```
```
# shell 2
rosrun turtlesim turtle_teleop_key
```
다음 명령어를 통해 토픽을 지정하여 `rqt_plot`을 실행할 수 있습니다.
```
# shell 3
rqt_plot /turtle1/pose/
```
혹은 `rqt_plot`을 먼저 실행한 뒤, Topic을 입력하는 방법이 있습니다.
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fk3n3a%2FbtqFo5CgAVm%2FrGtTk4qykp02gumjiPG7Vk%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fk3n3a%2FbtqFo5CgAVm%2FrGtTk4qykp02gumjiPG7Vk%2Fimg.png)
다음과 같이 거북이를 이동할 때마다 이동 정보가 실시간으로 표시됩니다.
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fc55qeX%2FbtqFo6HXvYN%2FTu1beDbhQmPWlWlmzQvkkk%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fc55qeX%2FbtqFo6HXvYN%2FTu1beDbhQmPWlWlmzQvkkk%2Fimg.png)
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2F89YwW%2FbtqFraJaN2S%2FA0J6XALTaaQ1JLuPGYWFT1%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2F89YwW%2FbtqFraJaN2S%2FA0J6XALTaaQ1JLuPGYWFT1%2Fimg.png)