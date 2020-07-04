ROS 설치만 몇 번째인지 모르겠습니다.. 이전에 올렸던 글에서는 gui는 띄울 수 있었지만, rviz나 gazebo를 구동하는 데에 실패했습니다. rviz나 gazebo는 hardware acceleration을 이용하는데, 이는 그래픽 카드가 필수입니다. 아래와 비슷한 에러가 발생한다면, 그래픽 드라이버가 잡히지 않아서라고 합니다.
```
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-yeomko'
[ INFO] [1593833014.567870114]: rviz version 1.14.0
[ INFO] [1593833014.567980964]: compiled against Qt version 5.11.3
[ INFO] [1593833014.568003887]: compiled against OGRE version 1.9.0 (Ghadamon)
[ INFO] [1593833014.579515645]: Forcing OpenGl version 0.
libGL error: No matching fbConfigs or visuals found
libGL error: failed to load driver: swrast
libGL error: No matching fbConfigs or visuals found
libGL error: failed to load driver: swrast
libGL error: No matching fbConfigs or visuals found
libGL error: failed to load driver: swrast
```
호스트의 그래픽 드라이버와 같은 드라이버를 docker container 내부에서도 잡아주라고 하는데, 맥은 그래픽 드라이버가 OS 내장이고, 이것저것 시도해봐도 도저히 갈피를 잡을 수 없었습니다. 결국 Ubuntu로 실행 환경을 변경했습니다....
하지만 여전히 docker의 간결함을 포기할 수 없어서... 다시 docker로 이것저것 해봤습니다... 먼저 Dockerfile을 만듭니다.
```
FROM osrf/ros:noetic-desktop-full-buster

# Arguments picked from the command line!
ARG user
ARG uid
ARG gid

# nvidia-docker container
ENV NVIDIA_VISIBLE_DEVICES \
    ${NVIDIA_VISIBLE_DEVICES:-all}
ENV NVIDIA_DRIVER_CAPABILITIES \
    ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics

#Add new user with our credentials
ENV USERNAME ${user}
RUN useradd -m $USERNAME && \
        echo "$USERNAME:$USERNAME" | chpasswd && \
        usermod --shell /bin/bash $USERNAME && \
        usermod  --uid ${uid} $USERNAME && \
        groupmod --gid ${gid} $USERNAME

USER ${user}
WORKDIR /home/${user}
```
저는 그래픽 카드가 nvidia라 이렇게 진행하였습니다. AMD나 Intel을 사용하시는 분들은 레퍼런스를 참고하세요. 아래 명령어를 사용하여 Dockerfile을 빌드하시면 my\_ros라는 이미지가 생성됩니다.
```
docker build --build-arg user=$USER --build-arg uid=$(id -u) --build-arg gid=$(id -g) -t my_ros .
```
이제 새 컨테이너를 생성하기 전에 gui 액세스 권한을 부여해야 합니다.
```
xauth nlist $DISPLAY | sed -e 's/^..../ffff/' | xauth -f /tmp/.docker.xauth nmerge -
```
다음과 같은 명령어로 컨테이너를 생성하면 완성~!! 컨테이너 이름은 중복되면 안 되며, 저장 공간 공유가 필요 없다면 해당 라인을 지우시면 됩니다.
```
nvidia-docker run -it \
    --name={컨테이너 이름} \
    -e DISPLAY=unix$DISPLAY \
    -e QT_X11_NO_MITSHM=1 \
    -e XAUTHORITY=/tmp/.docker.xauth \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v /tmp/.docker.xauth:/tmp/.docker.xauth:rw \
    -v {저장 공간이 컨테이너와 공유되길 원하는 경로}:/home/$USER/ros_ws \
    my_ros:latest
```
이제 환경변수를 설정하고, 마스터를 구동한 뒤 rviz를 실행해봅니다.
```
# 환경변수 설정
. /opt/ros/noetic/setup.bash
# 백그라운드에서 마스터 구동
roscore & # ctrl + c를 눌러 탈출해줍니다.
# rviz 구동
rosrun rviz rviz # rviz 명령어로 대체 가능
```

드디어 모습을 드러낸 rviz....
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FyUg3O%2FbtqFnPrJ1Mx%2Fhur7RRJy1PattyUSFYiokK%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FyUg3O%2FbtqFnPrJ1Mx%2Fhur7RRJy1PattyUSFYiokK%2Fimg.png)

## 레퍼런스:
[http://wiki.ros.org/docker/Tutorials/Hardware%20Acceleration](http://wiki.ros.org/docker/Tutorials/Hardware%20Acceleration)

[https://riptutorial.com/ko/docker/example/21831/linux-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EC%97%90%EC%84%9C-gui-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EC%8B%A4%ED%96%89%ED%95%98%EA%B8%B0](https://riptutorial.com/ko/docker/example/21831/linux-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EC%97%90%EC%84%9C-gui-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EC%8B%A4%ED%96%89%ED%95%98%EA%B8%B0)
