# Docker에서 Arduino(/dev/tty.usbserial) 접근하기
다음 링크를 참고하여 진행하였습니다.
[https://gist.github.com/stonehippo/e33750f185806924f1254349ea1a4e68](https://gist.github.com/stonehippo/e33750f185806924f1254349ea1a4e68)
[https://docs.docker.com/docker-for-mac/docker-toolbox/](https://docs.docker.com/docker-for-mac/docker-toolbox/)
먼저 Docker의 기본 구조를 알 필요가 있습니다. macOS 상에서 Docker는 Docker Desktop, Docker Toolbox로 구분할 수 있습니다. Docker for macOS를 설치하셨다면, `eval $(docker-machine env default)`를 하기 전에는 Docker Desktop, 이후에는 Docker Toolbox를 사용하는 개념입니다. Docker Desktop은 가상 환경 구성을 위해 HyperKit을 사용합니다. 아래 글에 따르면 HyperKit은 2019년까지도 USB forwarding을 할 수 없다고 합니다.
[https://christopherjmcclellan.wordpress.com/2019/04/21/using-usb-with-docker-for-mac/](https://christopherjmcclellan.wordpress.com/2019/04/21/using-usb-with-docker-for-mac/)
Docker Toolbox는 Virtualbox VM 내부에 생성된 Ubuntu에서 작동합니다. 해당 가상머신에는 `docker-machine` 명령어로 접근할 수 있습니다. Docker Toolbox는 Docker Desktop과 별도의 이미지를 가지고 있기 때문에, 기존에 Docker Desktop을 사용하고 있었다면 이를 Export, Import 하여야 합니다.
```
# image
docker save -o <name>.tar <image_name>:<tag>
# container
docker export <current_container> > <name>.tar 
```
다음으로 현재 작동하고 있는 vm을 중지시킵니다.
```
docker-machine stop default
```
다음으로 virtualbox를 실행합니다. 아래와 같이 표시됩니다.
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2F2HT2C%2FbtqFAPKXX6o%2FWYQhN3PP6bWDuVuxvT62tk%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2F2HT2C%2FbtqFAPKXX6o%2FWYQhN3PP6bWDuVuxvT62tk%2Fimg.png)
[setting] > [Ports] > [USB] 메뉴에 진입합니다. USB 2.0/3.0 메뉴를 선택하기 위해선 virtualbox extension을 설치해야 합니다. 이를 위해선 최신 virtualbox가 설치되어 있어야 합니다. (현재 6.1.10 버전이 필요합니다.)
[https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)
오른쪽 버튼 중 두 번째 버튼을 클릭하면 다음과 같이 USB 장치의 목록이 표시됩니다. 여기서 Arduino가 사용하는 USB Serial 제품을 선택하면 됩니다. 저는 하나는 블루투스, 하나는 무선 키보드/마우스임을 알고 있기에 쉽게 고를 수 있었습니다.
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fea4S3w%2FbtqFzzoZgII%2F463FODJSjqZ4HtRmWCSSMK%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fea4S3w%2FbtqFzzoZgII%2F463FODJSjqZ4HtRmWCSSMK%2Fimg.png)
확인을 누르고 docker-machine을 다시 실행합니다. 환경변수도 다시 세팅해줍니다.
```
docker-machine start default
eval $(docker-machine env default)
```
이미지/컨테이너를 Import 합니다.
```
# image
docker load -i <tar_file_name>
# container
docker import <tar_file_name or url or -(dash means stdin)> <storage_name>/<image_name>:<tag> 
```
아래 명령어로 컨테이너를 실행하면 Arduino에 접근하는 모습을 확인할 수 있습니다.
```
docker run -it --name <container_name> --device /dev/ttyUSB0 <image_name>
```