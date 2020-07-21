[원문](https://emanual.robotis.com/docs/en/platform/turtlebot3/opencr_setup/#opencr-setup)

# 6.3. OpenCR Setup

![https://emanual.robotis.com/assets/images/platform/turtlebot3/software/remote_pc_and_turtlebot.png](https://emanual.robotis.com/assets/images/platform/turtlebot3/software/remote_pc_and_turtlebot.png)

> NOTE: OpenCR에 대해 더 알고 싶다면, [OpenCR Wiki](https://emanual.robotis.com/docs/en/parts/controller/opencr10/)를 참고하세요.

## 6.3.1. OpenCR Firmware Upload for TB3

> NOTE: 펌웨어를 업로드하기 위해 두 가지 방법을 사용할 수 있지만, **shell script**를 사용하는 걸 추천합니다. TurtleBot3의 펌웨어를 수정해야 한다면, 두 번째 방법을 사용할 수 있습니다.

### 6.3.1.1. (Recommended) 쉘 스크립트

> NOTE: 윈도우를 사용하는 개발자라면 아래 아두이노 IDE를 사용하는 방법을 통해 펌웨어를 업로드 하세요.

이 명령어는 `Ubuntu 16.04`, `Ubuntu Mate`, `Linux Mint`, `Raspbian` (그리고 번역자가 사용한 `Ubuntu 20.04`)에서 테스트 되었습니다. OpenCR을 remote PC나(데스크톱이나 노트북) 터틀봇 PC(`Intel Joule`, `Raspberry Pi 3`)에 연결한 뒤 다음 커맨드를 사용하세요.

-   터틀봇3 버거
    
    ```
    export OPENCR_PORT=/dev/ttyACM0
    export OPENCR_MODEL=burger
    rm -rf ./opencr_update.tar.bz2 # 이전에 사용한 업데이트 파일이 있다면 삭제합니다.
    wget https://github.com/ROBOTIS-GIT/OpenCR-Binaries/raw/master/turtlebot3/ROS1/latest/opencr_update.tar.bz2 && tar -xvf opencr_update.tar.bz2 && cd ./opencr_update && ./update.sh $OPENCR_PORT $OPENCR_MODEL.opencr && cd .. # 최신 펌웨어를 다운로드하여 업데이트를 진행합니다. ROS2를 사용한다면 git 레포지토리 주소에서 ROS1을 ROS2로 변경합니다.
    ```
    
    ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/opencr/shell01.png)
    
      
    펌웨어 업로드가 끝나면 터미널에서 `jump_to_fw` 텍스트가 출력됩니다.
    
-   터틀봇3 와플, 와플 파이
    
    ```
    export OPENCR_PORT=/dev/ttyACM0
    export OPENCR_MODEL=waffle
    rm -rf ./opencr_update.tar.bz2
    wget https://github.com/ROBOTIS-GIT/OpenCR-Binaries/raw/master/turtlebot3/ROS1/latest/opencr_update.tar.bz2 && tar -xvf opencr_update.tar.bz2 && cd ./opencr_update && ./update.sh $OPENCR_PORT $OPENCR_MODEL.opencr && cd ..
    ```
    
    ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/opencr/shell02.png)
    
      
    펌웨어 업로드가 끝나면 터미널에서 `jump_to_fw` 텍스트가 출력됩니다.
    

### 6.3.1.2. (Alternative) Arduino IDE

> WARNING: 이 항목은 `Remote PC`에서 터틀봇3을 제어하는 데스크톱, 노트북 등에서 진행하여야 합니다. 이 명령어를 터틀봇3에 적용**하지 마십시오.**

이 단계를 시작하기 전에, 아두이노 IDE를 Remote PC에 설치합니다.

-   [OpenCR을 위한 아두이노 IDE 설치하기](https://emanual.robotis.com/docs/en/parts/controller/opencr10/#arduino-ide)

터틀봇3의 OpenCR 펌웨어는 다이나믹셀을 제어하거나, 센서 데이터를 전송하는 작업을 합니다. 펌웨어는 보드 매니저가 다운로드한 OpenCR 예제 폴더에 안에 있습니다.

터틀봇3 버거를 가지고 있다면,

**\[Remote PC\]** 파일 > 예제 > turtlebot3 > turtlebot3\_burger > turtlebot3\_core

터틀봇3 와플이나 와플 파이를 가지고 있다면,

**\[Remote PC\]** 파일 > 예제 > turtlebot3 > turtlebot3\_waffle > turtlebot3\_core

![](https://emanual.robotis.com/assets/images/platform/turtlebot3/opencr/o1.png)

**\[Remote PC\]** OpenCR에 펌웨어를 올리려면 `Upload` 버튼을 클릭하세요.

![](https://emanual.robotis.com/assets/images/platform/turtlebot3/opencr/o2.png)![](https://emanual.robotis.com/assets/images/platform/turtlebot3/opencr/o3.png)

> NOTE: 펌웨어를 업로드하는 과정에 에러가 발생한다면 도구 > 포트에서 올바른 포트가 선택되었는지 확인하세요. OpenCR에서 `Reset` 버튼을 누르고 펌웨어를 다시 업로드 합니다.

**\[Remote PC\]** 펌웨어가 업로드되면, `jump_to_fw` 텍스트가 화면에 출력됩니다.

## 6.3.2. 기본 동작

[##_Image|kage@E3Lyv/btqFQwTRRF1/EzPksPy6qeXZe0sKVbsUR0/img.png|alignCenter|data-origin-width="0" data-origin-height="0" data-ke-mobilestyle="widthContent"|||_##]

`PUSH SW 1`과 `PUSH SW 2` 버튼을 눌러 로봇이 올바르게 동작하는지 확인할 수 있습니다. 이 과정은 양쪽 다이나믹셀과 OpenCR 보드를 테스트합니다.

1.  터틀봇3을 조립하고나서, OpenCR에 배터리를 연결하고 스위치를 켭니다. OpenCR의 `Power LED`가 켜지는 걸 볼 수 있습니다.
2.  로봇을 바닥에 두세요. 테스트를 위해 1미터의 안전 거리를 두는 걸 추천합니다.
3.  `PUSH SW 1`를 몇 초간 꾹 누르고 있으세요. 로봇이 30cm 앞으로 움직입니다.
4.  `PUSH SW 2`를 몇 초간 꾹 누르고 있으세요. 로봇이 180도 회전합니다.