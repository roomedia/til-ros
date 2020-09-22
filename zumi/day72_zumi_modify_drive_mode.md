오늘은 주미 주행 모드를 수정하여 카메라 프레임과 주행 방향 데이터를 모아보도록 하겠습니다.

먼저 주미에 ssh로 접속하여 이전에 살펴보았던 `app.py` 파일을 열어줍니다.

```sh
# 주미 와이파이를 잡으신 뒤, ssh 접속
ssh pi@192.168.10.1
# 기존 wifi에 등록된 주미의 와이파이를 통해 접속하는 방법
ssh pi@192.168.0.34 # 주미에서 ifconfig를 통해 알아낸 ip, 사람마다 다릅니다.
# 초기 비밀번호는 pi입니다.
```

```sh
pushd /usr/local/lib/python3.5/dist-packages/zumidashboard
sudo vi app.py
```

`/zumi_direction`을 입력하여 키패드 입력 시 방향을 전달하는 부분을 찾아줍니다. 전달된 방향은 주행에 사용됩니다. 이미지와 방향을 저장할 수 있도록 코드를 추가하도록 하겠습니다.

```python
@socketio.on('zumi_direction')
def zumi_direction(input_key):
    app.drive_mode.zumi_direction(input_key)
```

파이 카메라로부터 이미지를 받아오는 코드는 `drive_mode.py`에 있고, 입력 방향을 받아오는 부분은 `app.py`에 있어 구현이 까다롭습니다. 그래서 `app.py`에서는 `drive_mode.py`로 방향 데이터를 전달하는 역할만 하고, 실제 저장은 `drive_mode.py`에서 진행하겠습니다. 코드를 다음과 같이 변경합니다.

```python
import requests
# ...
@socketio.on('zumi_direction')
def zumi_direction(input_key):
    app.drive_mode.zumi_direction(input_key)
    response = requests.get('http://0.0.0.0:3456/save_video?direction=%s' % input_key)
```

이제 같은 폴더의 `drive_mode.py`를 열고 다음 코드를 추가합니다. `save_video()` 부분이 `app.py`에서 우리가 방금 추가한 요청을 처리하는 부분이며, `__save_video(direction)`가 실제 데이터 저장을 수행하는, 쓰레드에서 사용되는 함수입니다.

```python
def __save_video(direction):
    # load frame number
    if os.path.isfile(save_txt):
       f = open(save_txt, 'r')
       number = int(f.read()) + 1
       f.close()
    else:
       number = 1

    # frame 변수에 접근
    global frame
    if frame is None:
        return False

    # grayscale and crop
    frame = cv2.cvtColor(frame, cv2.COLOR_RGB2GRAY)[120:]
    # set name
    filename = os.path.join(save_dir, '%d.jpg' % number)
    # save frame
    cv2.imwrite(filename, frame)

    # save frame number
    f = open(save_txt, 'w')
    f.write(str(number))
    f.close()

    # save frame number and direction
    f = open(direction_txt, 'a')
    f.write(' '.join([str(number), direction]))
    f.write('\n')
    f.close()

    return True

@app.route('/save_video')
def save_video():
    save_video_thread = Thread(target=__save_video, args=(request.args.get('direction'),))
    save_video_thread.start()
    return json.dumps(True)
```

frame을 받아오기 위해서는 `gen()` 함수에서 사용되는 frame 변수가 글로벌 변수가 되도록 설정해주어야 하는데요. (생각해보니 굳이 안 해도 될 것 같기도 하고..🤔)

```python
frame = None
# ...
def gen():
    global frame
    # ...
```

초깃값이 None이기 때문에 카메라가 실행되지 않았을 때 이미지를 저장하려하면 에러가 발생할 수 있습니다. 이를 방지하는 부분이 `__save_video(direction)`의 다음 부분입니다.

```python
    # 글로벌 변수 frame
    global frame
    # 카메라가 실행되지 않은 경우
    if frame is None:
        # 조기 종료
        return False
```

파일이 저장되는 경로는 다음과 같이 설정하였습니다.

```python
save_dir = '/home/pi/Dashboard/DriveImg'
save_txt = os.path.join(save_dir, 'number.txt')
direction_txt = os.path.join(save_dir, 'direction.txt')
```

해당 위치는 쓰기 권한이 부여되어 있어야 하는데요. 다음 쉘 명령어를 입력하여 부여할 수 있습니다.

```sh
pushd /home/pi/Dashboard/DriveImg/
sudo chmod a+w .
```

`ls -al`로 확인했을 때 다음과 같이 표시되어야 합니다.

```sh
# 시간은 상관 없습니다...
drwxrwxrwx 2 root root 4096 Sep 22 16:04 .
```

이제 방향키가 눌릴 때마다 비디오 프레임과 이동 방향이 각각 저장됩니다. 다음 시간에는 이를 가공하여 머신러닝 데이터로 사용할 수 있을지 시험해봐야겠습니다. 라벨링을 용이하게 하려면 ArrowRight 등을 0, 1, 2, 3 등의 enum으로 변경하여야 할 듯 보이네요^^

```sh
# Direction.txt
1 ArrowRight
2 ArrowLeft
3 ArrowDown
```

```sh
# number.txt
3
```

![ex](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fp2Gox%2FbtqI9p5etE8%2FL0XhKFj3Gki8rTcwYjls41%2Fimg.jpg)

![ex](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fci9ZlU%2FbtqJhI33iDn%2FIX2v0FFAkks0eKKfHUSORk%2Fimg.jpg)

![ex](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FsAo1M%2FbtqJhIJGRfj%2F3vYnywiyxZOMXqku5TJ26k%2Fimg.jpg)