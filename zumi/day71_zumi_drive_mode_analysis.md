# drive mode 파일 위치 파악하기
먼저 drive mode가 정확히 어떤 파일을 실행하는지 알아내기 위해 다음 명령어를 통해 프로세스 목록을 저장한 뒤 비교하겠습니다. 먼저 주미 대시보드에 접속한 뒤, 주미에 ssh 접속하여 다음 명령어를 입력합니다.

```sh
ps -ef | grep python | grep -v grep
```

`ps -ef`는 현재 실행되고 있는 모든 프로세스(프로그램)을 표시하고, `grep python`은 그 중 `python`이라는 단어가 들어가는 항목만 추려냅니다. `grep -v grep`은 `grep`이란 단어가 들어간 항목을 제외합니다. 내용을 확인해볼까요?

```
root       481     1  4 15:44 ?        00:00:33 python3 /home/pi/Dashboard/dashboard.py
root      1592     1  0 15:45 ?        00:00:00 sudo python3 /usr/local/lib/python3.5/dist-packages/zumidashboard/gesture.py
root      1596  1592  4 15:45 ?        00:00:31 python3 /usr/local/lib/python3.5/dist-packages/zumidashboard/gesture.py
pi        2461     1 11 15:53 ?        00:00:21 /usr/bin/python3 /usr/local/bin/jupyter-notebook --notebook-dir=/home/pi/Dashboard/user/EngHyu/
```

`dasyboard.py`, 관리자 권한으로 실행된 `gesture.py`, `gesture.py` 하나 더, `jupyter-notebook`이 실행되고 있는 게 보이시나요?

이제 `주행 모드` 버튼을 눌러 카메라가 켜질 때까지 기다린 뒤 아까의 명령어를 다시 실행합니다.

```sh
ps -ef | grep python | grep -v grep
```

```sh
root       481     1  3 15:44 ?        00:00:42 python3 /home/pi/Dashboard/dashboard.py
root      1592     1  0 15:45 ?        00:00:00 sudo python3 /usr/local/lib/python3.5/dist-packages/zumidashboard/gesture.py
root      1596  1592  4 15:45 ?        00:00:46 python3 /usr/local/lib/python3.5/dist-packages/zumidashboard/gesture.py
pi        2461     1  3 15:53 ?        00:00:21 /usr/bin/python3 /usr/local/bin/jupyter-notebook --notebook-dir=/home/pi/Dashboard/user/EngHyu/
root      2673   481  0 16:02 ?        00:00:00 sudo sh /usr/local/lib/python3.5/dist-packages/zumidashboard/shell_scripts/drivemode.sh .
root      2678  2673  0 16:02 ?        00:00:00 sh /usr/local/lib/python3.5/dist-packages/zumidashboard/shell_scripts/drivemode.sh .
root      2679  2678  0 16:02 ?        00:00:00 sudo python3 /usr/local/lib/python3.5/dist-packages/zumidashboard/drive_mode.py
root      2690  2679 23 16:02 ?        00:00:13 python3 /usr/local/lib/python3.5/dist-packages/zumidashboard/drive_mode.py
root      2704  2690 63 16:03 ?        00:00:23 /usr/bin/python3 /usr/local/lib/python3.5/dist-packages/zumidashboard/drive_mode.py
```

`/usr/local/lib/python3.5/dist-packages/zumidashboard/` 폴더 아래의 `drive_mode.py`와 `shell_scripts/drivemode.sh`가 실행되는 게 보입니다. 위치도 알아냈으니 두 파일은 어떤 내용일지 확인해봅시다.

쉘 파일을 먼저 확인하겠습니다.

```sh
cd /usr/local/lib/python3.5/dist-packages/zumidashboard
cat shell_scripts/drivemode.sh 
```

```sh
sudo python3 /usr/local/lib/python3.5/dist-packages/zumidashboard/drive_mode.py
```

쉘 파일은 그저 `drive_mode.py`를 실행하기 위한 장치였습니다. 아마 웹사이트에서 버튼을 누르면 sh 파일이 실행되는 구조이려나요..?

다음은 `drive_mode.py` 파일입니다.

```py
from zumi.util.camera import Camera

from flask import Flask, render_template, Response
import cv2
import time
import os
from threading import Thread

class DriveMode:
    def __init__(self, _zumi):
        self.zumi = _zumi
        self.camera = Camera()
        self.current_key = ''
        self.drive_thread = ''

    def __move_zumi(self):
        desired_angle = self.zumi.read_z_angle()
        self.zumi.paly_note(0,0)

        while self.current_key != '':
            if self.current_key == "ArrowUp":
                k_p = 2.9
                k_i = 0.0
                k_d = 0.0
                accuracy = 5
                self.zumi.drive_at_angle(80, 40, desired_angle, k_p, k_d, k_i, accuracy)
            elif self.current_key == "ArrowDown":
                k_p = 2.9
                k_i = 0.0
                k_d = 0.0
                accuracy = 5
                self.zumi.drive_at_angle(80, -40, desired_angle, k_p, k_d, k_i, accuracy)
            elif self.current_key == "ArrowLeft":
                k_p = 0.6
                k_i = 0.000
                k_d = 0.0
                accuracy = 3
                self.zumi.drive_at_angle(10, 0, desired_angle + 360, k_p, k_d, k_i, accuracy)
            elif self.current_key == "ArrowRight":
                k_p = 0.6
                k_i = 0.0
                k_d = 0.0
                accuracy = 3
                self.zumi.drive_at_angle(10, 0, desired_angle - 360, k_p, k_d, k_i, accuracy)
            elif self.current_key == "q":
                self.send_image_thread.join()
            else:
                break

        self.zumi.stop()

    def zumi_direction(self, input_key):
        if input_key != self.current_key:
            self.current_key = input_key
            self.drive_thread = Thread(target=self.__move_zumi)
            self.drive_thread.start()

    def zumi_stop(self):
        self.current_key = ''
        if type(self.drive_thread) != str and self.drive_thread.isAlive():
            self.drive_thread.join()
        self.zumi.stop()

app = Flask(__name__)

@app.route('/')
def index():
   """Video streaming ."""
   return render_template('drivescreen.html')

def gen():
    camera = Camera(320, 240, auto_start=True)

    count = 0
    timee = 0
    start = time.time()

    if not os.path.isdir('/home/pi/Dashboard/DriveImg'):
        os.makedirs('/home/pi/Dashboard/DriveImg')

    """Video streaming generator function."""
    while True:
        frame = camera.capture()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        (flag, encodedImage) = cv2.imencode(".jpg", frame)
        if not flag:
            continue
        yield (b'--frame\r\n' 
              b'Content-Type: image/jpeg\r\n\r\n' + bytearray(encodedImage) + b'\r\n')


@app.route('/video_feed')
def video_feed():
   """Video streaming route. Put this in the src attribute of an img tag."""
   return Response(gen(),
                   mimetype='multipart/x-mixed-replace; boundary=frame')


if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True, threaded=True, port=3456)
```

여러분은 파이썬 코드를 어디부터 읽으시나요? 저는 `import`부터 본 뒤 바로 `if __name__ == '__main__':` 부분으로 넘어가는 편입니다. 그 편이 무엇이 실행되는지 파악하기 쉬운 것 같습니다. 이 파일은 주미의 Camera 모듈과 flask, opencv, 시스템 모듈을 사용하네요.

메인으로 넘어와서 바로 보이는 건, 3456 포트에서 `flask` 서버를 실행하는 모습입니다. `192.168.0.34:3456/`에서는 아무 화면이 뜨지 않지만, `192.168.0.34:3456/video_feed`에 접속하면 카메라 화면을 볼 수 있습니다.

![camera]()

`iframe`을 사용하여 해당 화면을 주행 모드에서 보여주며, `gen()` 함수를 사용하여 카메라 화면을 지속적으로 리프레싱 하는 것 같네요.

![iframe]()

`DriveMode` 클래스는 존재하지만 해당 파일 내에서 사용되고 있지 않습니다. 그렇다면 화살표 버튼을 눌러 주미를 이동하는 부분의 코드는 어디에 있을까요? python 프로세스를 하나씩 종료해보니, `dashboard.py`를 종료했을 때 주미를 조종할 수 없었습니다.

```sh
cat ~/Dashboard/dashboard.py
```

```python
import zumidashboard.app as app
app.run()
```

결국 돌고돌아 `zumidashboard` 폴더의 `app.py`였습니다.

```python
from flask import Flask
from flask_socketio import SocketIO
from flask import send_from_directory
from zumi.zumi import Zumi
from zumi.util.screen import Screen
from zumi.protocol import Note
import zumidashboard.scripts as scripts
import zumidashboard.sounds as sound
import zumidashboard.updater as updater
from zumidashboard.drive_mode import DriveMode
import time, subprocess, os, re, base64
from threading import Thread
import logging, json
from logging.handlers import RotatingFileHandler
# ...
```

이번에도 역시 백엔드로 `flask`를 사용하며, `DriveMode` 클래스 또한 여기서 사용되는 걸 알 수 있습니다. `flask` 웹 서버라는 걸 알았으니 곧장 `/drive`를 라우팅하는 부분으로 가볼까요?

```python
# ...
app = Flask(__name__, static_url_path="", static_folder='dashboard')
# ...
@app.route('/drive')
def drive():
    return app.send_static_file('index.html')
# ...
@socketio.on('zumi_direction')
def zumi_direction(input_key):
    app.drive_mode.zumi_direction(input_key)


@socketio.on('zumi_stop')
def zumi_stop():
    app.drive_mode.zumi_stop()
# ...
```

`/drive`에 접속하면 정적 파일인 `index.html`을 표시하며, 아마도 해당 파일에서 프론트엔드를 담당하는 엔진? 부분을 호출하겠죠. `socket.io`를 통해 `zumi_direction`과 `zumi_stop` 라는 이벤트가 발생했을 때 각각 해당하는 함수가 실행됩니다. 저게 다인걸 보면 제어는 어렵지 않아 보이네요.

프론트엔드를 살펴보겠습니다. 주미를 제어하는 버튼과 이벤트를 발생시키는 부분입니다. 개발자 모드에서 디버거를 열면 해당 부분의 react 컴포넌트를 확인할 수 있는데요. 다음과 같습니다.

![개발자 도구-디버거-DriveMode.js]()

```js
  // ...
  handleKeyDown(e) {
    let handleKey = e.key;
    global.socket.emit("zumi_direction", handleKey);

    if(handleKey === "ArrowUp") {
      this.setState({clickArrowUp: true});
    } else if(handleKey === "ArrowDown") {
      this.setState({clickArrowDown: true});
    } else if(handleKey === "ArrowLeft") {
      this.setState({clickArrowLeft: true});
    } else if(handleKey === "ArrowRight") {
      this.setState({clickArrowRight: true});
    }
  }

  handleKeyUp(e) {
    global.socket.emit("zumi_stop");
    this.setState({clickArrowUp: false});
    this.setState({clickArrowDown: false});
    this.setState({clickArrowLeft: false});
    this.setState({clickArrowRight: false});
  }

  directionMouseUp(e) {
    e.stopPropagation();
    this.setState({clickArrowUp: true});
    global.socket.emit("zumi_direction", "ArrowUp");
  }

  directionMouseDown(e) {
    e.stopPropagation();
    this.setState({clickArrowDown: true});
    global.socket.emit("zumi_direction", "ArrowDown");
  }

  directionMouseLeft(e) {
    e.stopPropagation();
    this.setState({clickArrowLeft: true});
    global.socket.emit("zumi_direction", "ArrowLeft");
  }

  directionMouseRight(e) {
    e.stopPropagation();
    this.setState({clickArrowRight: true});
    global.socket.emit("zumi_direction", "ArrowRight");
  }

  directionMouseStop() {
    global.socket.emit("zumi_stop");
    this.setState({clickArrowUp: false});
    this.setState({clickArrowDown: false});
    this.setState({clickArrowLeft: false});
    this.setState({clickArrowRight: false});
  }
  // ...
```

`handleKeyDown`, `handleKeyUp`은 키보드 이벤트에 연결되고, `directionMouse` 시리즈는 버튼과 연결됩니다. 각각 이벤트 발생 시 주미에서 실행할 함수명을 소켓으로 전달하고 자신의 state를 변경하여 버튼 색상 등을 변경합니다.

다음 시간에는 키보드 이벤트가 발생할 때 현재 영상과 키 입력을 저장하는 코드를 작성해보겠습니다.