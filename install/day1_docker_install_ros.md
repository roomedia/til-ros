**ros2에 대한 학습 자료를 찾기 어려워 부득이하게 ros1으로 변경되었습니다.**
# 1일차 - docker를 이용한 ros 설치

ros2를 맛보기 위해 설치를 하려고 했는데,,, 설치부터 난관의 연속입니다. 먼저 ros2 foxy의 경우 macOS 설치법 또한 제공은 한다고 하여, macOS에 먼저 진행 해보았습니다. 듀얼 부팅은 귀찮잖아요..? 결과부터 말씀드리면.... 장렬히 실패하고 docker를 이용하여 컨테이너를 띄웠습니다. 중간에 문제가 생겨서 구글링해도 아무 도움이 되지 않으니 추천 드리지 않습니다. 저의 경우 `colcon build` 명령어를 이용하여 ros2를 빌드하던 중, qt\_gui\_cpp에 문제가 생겨 중도 드랍했습니다.

# docker

docker는 리눅스 컨테이너를 기반으로한 가상화 서비스 머신입니다. docker가 없으시다면 아래 문서를 참고하여 docker를 설치해주세요.

[https://hub.docker.com/editions/community/docker-ce-desktop-mac/](https://hub.docker.com/editions/community/docker-ce-desktop-mac/)

dockerhub에 가시면 osrf에서 만든 도커 이미지가 있습니다.

[https://hub.docker.com/r/osrf/ros](https://hub.docker.com/r/osrf/ros)

학습하는 입장에서 어떤 기능을 사용할 지 모르기 때문에, 가장 최신이며 용량이 큰 `noetic-desktop-full-buster` 이미지를 다운로드 했습니다. docker가 실행된 상태로 아래 명령어를 터미널에 입력하여 docker image를 다운로드 할 수 있습니다.

```
# 도커 이미지 다운로드
docker pull osrf/ros:noetic-desktop-full-buster
```

docker 실행 여부는 오른쪽 위 상태바를 보면 알 수 있습니다. 고래 아이콘이 떠있으면 도커가 실행되고 있다는 표식입니다.

![https://k.kakaocdn.net/dn/Yad0n/btqFjczRnj6/kf1O65x3M2YCPWkkEj8MOk/img.png](https://k.kakaocdn.net/dn/Yad0n/btqFjczRnj6/kf1O65x3M2YCPWkkEj8MOk/img.png)

mac에서 docker를 통해 gui 프로그램을 실행하기 위해선, container를 생성하기 전에 socat과 xquartz를 설치합니다. 설치 후에는 xquartz를 실행합니다.

```
brew install socat
brew cask install xquartz
open -a xquart
```

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FdS2ZdE%2FbtqFjdlqFky%2FCmUflauc1LocKVS88J14Ak%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FdS2ZdE%2FbtqFjdlqFky%2FCmUflauc1LocKVS88J14Ak%2Fimg.png)

해당 창이 표시되면 다음과 같이 환경설정에 진입하여 보안 탭을 다음과 같이 설정합니다. 환경설정은 cmd + , 키로도 진입 가능합니다.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Flj99x%2FbtqFhldE6QZ%2FpYgrJqsCxKoTOz6l201j61%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Flj99x%2FbtqFhldE6QZ%2FpYgrJqsCxKoTOz6l201j61%2Fimg.png) ![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbDDuKE%2FbtqFgHO1Gna%2FDVAJER2PkYoun3Br1iGTc1%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbDDuKE%2FbtqFgHO1Gna%2FDVAJER2PkYoun3Br1iGTc1%2Fimg.png)

터미널에서 아래 명령어를 입력합니다. 명령어는 해당 터미널에서 계속 실행 중이므로 새 터미널 창을 띄웁니다. 띄어쓰기 없습니다.

```
socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\\"$DISPLAY\\"
```

아래 명령어로 docker image로부터 container를 생성합니다. container 이름은 ros로 설정하였으나, 임의로 변경하셔도 됩니다. 해당 명령어는 컨테이너 생성 시에만 사용합니다.

```
# ip 확인 후 xhost에 추가
ip=$(ifconfig en0 | grep inet | awk '$1=="inet" {print $2}')
xhost + $ip

# 컨테이너 생성
docker run -it -e DISPLAY=$ip:0 --name ros osrf/ros:noetic-desktop-full-buster
```

다음과 같은 화면으로 진입합니다. 현재 보이는 화면은 기존의 터미널과 분리된, 가상 환경입니다. root@ 뒤에 붙은 `fe3d5ba6bc80`은 다를 수 있습니다.

![https://k.kakaocdn.net/dn/tDWgJ/btqFfYDDzdm/xeZGQW493BOF0bmSwk3Ok0/img.png](https://k.kakaocdn.net/dn/tDWgJ/btqFfYDDzdm/xeZGQW493BOF0bmSwk3Ok0/img.png)

가상 환경에서 나가기 위해선 `exit`을 사용하고, 컨테이너를 종료하려면 다음 명령어를 사용합니다.

```
# 컨테이너 종료
docker stop ros
```

다음 번 실행 시에는 아래 명령어를 사용합니다.

```
# 컨테이너가 종료된 경우, 컨테이너 실행
docker start ros

# 컨테이너가 실행 중이면 다음 명령어를 통해 컨테이너 접속
docker exec -it ros bash
```
다음으로 gui, ros 작동 여부 등을 테스트 해보겠습니다.

먼저 ros 환경변수를 설정해줍니다. 매번 터미널을 열때마다 설정해야하며, ~/.bashrc 파일에 해당 내용을 적어주면 반복 작업을 피할 수 있습니다.

```
. /opt/ros/foxy/setup.bash
```

master를 실행합니다. 해당 명령어를 실행하면 계속 master가 실행된 상태이므로 새로운 터미널 창을 세 개 열어 각각 turtlesim_node와 turtle_teleop을 실행합니다.
```
# ros master 실행
roscore
```
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FxbAcT%2FbtqFiy5Zky9%2Fahv4fbXIty7R0HEvElOkjk%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FxbAcT%2FbtqFiy5Zky9%2Fahv4fbXIty7R0HEvElOkjk%2Fimg.png)
```
# turtlesim gui 노드 표시
rosrun turtlesim turtlesim_node
```
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FcoAJqs%2FbtqFjFiLtlu%2FCPkhZyL2TVEHzpwIhTTlF0%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FcoAJqs%2FbtqFjFiLtlu%2FCPkhZyL2TVEHzpwIhTTlF0%2Fimg.png)
```
# turtlesim 이동 노드 표시
rosrun turtlesim turtle_teleop_key
```
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fyq9NW%2FbtqFkmXBIWk%2Frb27tMDcXMnI78BbcrRnrk%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fyq9NW%2FbtqFkmXBIWk%2Frb27tMDcXMnI78BbcrRnrk%2Fimg.png)
```
# 노드 간 관계 표현
rqt_graph
```
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FEhA8o%2FbtqFkTOaNvQ%2F0KDbt0CWeI29VN5YQ2Jr30%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FEhA8o%2FbtqFkTOaNvQ%2F0KDbt0CWeI29VN5YQ2Jr30%2Fimg.png)
다음과 같이 표시되며, `turtle_teleop_key`에서 방향키를 입력했을 때 turtlesim_node에서 거북이가 움직이면 정상입니다.

ros 뉴비가 저와 같은 문제를 겪으며 의지가 꺾이지 않았으면 합니다.

From 뉴비.
