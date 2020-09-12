# zumi 와이파이 사용하지 않고 접속
주미를 켠 다음, 노트북에서 zumi0000 와이파이를 잡고 [zumidashboard.ai](zumidashboard.ai)에 접속합니다. 주미가 무선 인터넷에 연결이 되면, 터미널에서 ifconfig 등을 이용하여 주미의 아이피를 확인합니다.

```sh
pi@zumi3766:~/Dashboard $ ifconfig wlan0
```
```sh
wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.34  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::ba27:ebff:fea0:6e57  prefixlen 64  scopeid 0x20<link>
        ether b8:27:eb:a0:6e:57  txqueuelen 1000  (Ethernet)
        RX packets 19932  bytes 2167568 (2.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 33190  bytes 34118562 (32.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

노트북에서 주미가 접속한 와이파이와 같은 AP에 접속하여 확인한 inet으로 접속합니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbM9u6B%2FbtqIxXAiayP%2Fj9hLTheeLwmoDALGDVTOkk%2Fimg.png)

# 주미 터미널 접속
이전에 주미 대시보드에 접속하여 무선랜을 연결한 적이 있다면 주미는 부팅 시 알아서 와이파이를 잡습니다. 주미 AP에 접속하지 않아도, (2G, 5G 상관 없이) 주미와 같은 와이파이를 사용 중이라면 와이파이 내에서 주미의 아이피를 확인한 후, 해당 아이피로 ssh를 시도할 수 있습니다.

```sh
ssh pi@192.168.0.34
```

혹은 주미의 jupyter에서 터미널을 열 수 있습니다. [http://192.168.0.34:5555/terminals/1](http://192.168.0.34:5555/terminals/1)에 접속하면 웹셸 형태의 터미널이 보입니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FNIvxv%2FbtqIDEmci1f%2FDryjTxVLabPZuLkbQHmrLk%2Fimg.png)

# 주미 충전 중 사용
위 사항들을 응용하면 주미를 충전하면서도 일부 사용할 수 있습니다. 주미의 본체를 이루는 라즈베리파이는, 전원이 연결되면 별도의 부팅 과정 없이도 스스로 실행되기 때문입니다. 다만, 충전 중에는 (케이블을 인식하여) 대시보드가 실행되지 않습니다.

# zumi.forward()
`zumi.forward()` 명령어를 사용하여 직진하고, `zumi.reverse()` 명령어를 사용하여 후진할 수 있습니다. 

```sh
from zumi.zumi import Zumi

zumi = Zumi()
zumi.forward()
```

`zumi.forward()`는 어떤 코드로 이루어져 있을까요? 아마도 아두이노에 I2C로 값을 전달하여 두 DC 모터를 움직이고 있을 겁니다. 확인해보도록 할까요?

## __init__()
```py
# zumi.py
# ...
class Zumi:
  def __init__(self):
    # ...

    # drive PID values; used in forward(),go_straight() etc
    self.D_P = 2.9
    self.D_I = 0.01
    self.D_D = 0.05

    # turn PID values; used in turn()
    self.T_P = 0.6
    self.T_I = 0.001
    self.T_D = 0.001
    # ...
```

먼저 주미 클래스는 이것저것 초깃값을 설정하고 있습니다. 그 중 `zumi.forward()`에 사용되는 값은 위와 같습니다. PID 값은 무엇일까요? PID는 PID 제어에 사용되는 파라미터입니다. PID 제어(비례-적분-미분 제어기, Proportional-Integral-Differential controller)는 출력값과 설정값을 비교하여 오차를 계산한 뒤, 오차의 P배, 오차를 적분한 값의 I배, 오차를 미분한 값의 D배를 하는 구조로, PID의 각 값이 달라지면 그래프는 다음과 같이 변화합니다.

![PID Effect](https://upload.wikimedia.org/wikipedia/commons/3/33/PID_Compensation_Animated.gif)

오차를 e(t)로 나타내고, PID를 각각 K_p, K_i, K_d로 나타낸 공식은 다음과 같습니다.

![PID](https://wikimedia.org/api/rest_v1/media/math/render/svg/82e28e130699d4c7e795046f5b44b729a8e48e86)

세 가지 항이 가지는 의미는 다음과 같습니다.

- 비례항: 현재 상태에서 오차값의 크기에 비례한 제어 작용을 합니다.
- 적분항: 정상 상태(steady-state)에서 오차를 없애는 작용을 합니다.
- 미분항: 출력값의 급격한 변화에 제동을 걸어 오버슛을 줄이고 안정성을 향상시키는 작용을 합니다.

이를 알고 다시 한 번 그래프를 봅시다. P가 증가할수록 그래프의 전체적인 높이가 커지고, I가 증가하면 굴곡이 심해지고, D가 증가하면 그래프가 다시 평평해집니다.

![PID Effect](https://upload.wikimedia.org/wikipedia/commons/3/33/PID_Compensation_Animated.gif)

출력값은 제어에 필요한 제어값이라고 하는데, 우리의 경우에는 속도를 얼마나 증가/감속할지 결정하므로 가속도를 나타내겠지요? PID 값들은 실험적/경험적 방법을 통해 계산한다고 합니다. (계산 방법으로는 지글러-니콜스 방법 등이 있습니다.) PID에 대해서는 이렇게 대략적으로만 알아보고 `zumi.forward()` 함수를 알아보도록 할까요?

## forward()
```py
  # ...
  def forward(self, speed=40, duration=1.0, desired_angle=123456, accuracy=1.0):
    '''
    #:param speed: the forward speed you want Zumi to drive at. Should only input a positive number
    #:param duration: number of seconds you want Zumi to try to drive forward
    #:param desired_angle: the desired angle
    #:return: nothing
    '''
    k_p = self.D_P
    k_i = self.D_I
    k_d = self.D_D

    if (desired_angle == 123456):
      # if no input find the z angle and go in that direction
      desired_Angle = self.read_z_angle()

    max_speed = 127

    self.reset_PID()
    start_time = time.time()
    speed_now = 0
    accel_period = 0.7
    # acceleration to start motors
    # -------------------------------------
    if duration >= accel_period:
      while time.time() - start_time < aceel_period:
        if (speed_now < max_speed):
          speed_now = speed_now + 1
        self.drive_at_angle(60, abs(speed_now), desired_angle, k_p, k_d, k_i, accuracy)

    elif duration < accel_period:
      while time.time() - start_time < abs(duration * 0.6):
        if (speed_now < max_speed):
          speed_now = speed_now + 1
        self.drive_at_angle(60, abs(speed_now), desired_angle, k_p, k_d, k_i, accuracy)
    # -------------------------------------

    # stay with constant max speed
    while (time.time() - start_time) < abs(duration):
      self.drive_at_angle(max_speed, abs(speed), desired_angle, k_p, k_d, k_i, accuracy)
    self.stop()
```

모르는 내용이 참 많네요^^; 자체적인 함수는 나중에 알아보기로 하고 로직부터 설명해볼까요? 먼저 주행을 위한 PID 값을 받아옵니다. desired_angle이 초깃값이면 현재 향하고 있는 방향을 읽어오고, 아니라면 해당 방향으로 향하는 듯 합니다. 최대 속도는 127이며 초기 속도는 0입니다. 현재 시간부터 accel_period까지 속도를 하나씩 올려가며 가속을 하며, 이후에는 속도를 유지하며 달립니다. duration 동안 주행한 이후에는 멈춰섭니다. duration이 accel_period보다 작은 경우에는 duration의 0.6배만큼 가속하고 이후 해당 속도로 주행합니다.

## etc.
각각의 함수들을 살펴보면 다음과 같습니다. `zumi.read_z_angle()` 함수는 자이로 센서에서 Z 값을 읽어옵니다.

```py
def read_z_angle(self):
  """
  updates and reads the angle from the gyroscope
  and returns only the z angle
  :return: z axis angle
  """
  # updates and reads only the Z angle
  angle_list = self.update_angles()
  return angle_list[2]
```

`zumi.reset_PID()`는 적분 에러와 누적 에러를 초기화하고 PID와 gyro 시간을 초기화합니다.

```py
def reset_PID(self):
  """
  The error values for error integral and error past
  will need to be reset every time the
  PID needs to be used for driving straight or turning
  """
  self.PID_time_past = time.time()
  self.gyro_time_past = time.time() $ for the angle readings
  self.error_sum = 0
  self.error_past = 0
```

# zumi.drive_at_angle(...)
대망의 `zumi.drive_at_angle(...)`은 과연 어떤 모양일까요?

```py
def drive_at_angle(self, max_speed, base_speed, desired_angle, k_p, k_d, k_i, min_error):
  """
  #Use this method if you wish to drive Zumi in a straight line or turn.

  #:param max_speed: This number will cap the max speed for the motors.
  #:param base_speed: This is the "forward/reverse" speed Zumi will drive at.
  # set to 0 if you wish not to move forward or reverse.
  #:param desired_angle: This will be the angle Zumi will try to drive towards. Can be a positive or negative number
  #:param k_p: constant for proportional controller.
  #:param k_d: constant for derivative controller.
  #:param k_i: constant for integral controller.
  #:param min_error: the minimum error that the motors will stop trying to correct for
  #:return: error
  """
  # update the angles and grab the Z angle
  angles_now_list = self.update_angles()
  angle_z = angles_now_list[2]

  # calculate error values
  error = desired_angle - angle_z
  self.error_sum = self.error_sum + error

  error_change = error - self.error_past
  self.error_past = error

  # find change in time
  dt = time.time() - self.PID_time_past
  self.PID_time_past = time.time()

  # PID components
  e_p = (k_p * error)
  e_d = (k_d * error_change / dt)
  e_i = (k_i * self.error_sum * dt)
  speed_offset = int(e_p + e_d + e_i)

  # if within the error threshold set motors to 0 speed
  if abs(error) < min_error:
    speed_offset = 0

  # input speed for all functions that use drive_at_angle are capped at +-MAX_USER_SPEED=80
  base_speed = self.clamp(int(base_speed), -1 * self.MAX_USER_SPEED, self.MAX_USER_SPEED)

  # make sure motor speeds never go above 127 or below -127
  final_left_speed = self.clamp(base_speed - speed_offset, -max_speed, max_speed)
  final_right_speed = self.clamp(base_speed + speed_offset, -max_speed, max_speed)

  # send I2C msg to set motors to these values, with no acceleration
  self.control_motors(final_right_speed, final_left_speed)
  return error
```

자이로 센서에서 Z 값을 받아와 desired_angle과의 에러를 측정합니다. 또한, PID 제어 파트가 다시 나왔네요. 미분은 단위 시간 당 에러의 변화량으로, 적분은 누적 에러 * 단위 시간으로 해결할 수 있었습니다.

`zumi.forward()`에선 min_error로 항상 accuracy를 넣어주는데, `zumi.drive_at_angle()`에서는 에러가 accuracy보다 작을 경우 가속도를 0으로 만들어버립니다.

base_speed와 final_left_speed, final_right_speed를 일정 범위 안에 매핑하기 위해 clamp 함수를 사용합니다. 이후 조정된 값을 I2C 메시지로 아두이노에 전달합니다.

```py
def control_motors(self, right, left, acceleration=0):
  right = self.clamp(right, -126, 127)
  left = self.clamp(left, -126, 127)
  right = _convert_to_arduino_speed(right)
  left = _convert_to_arduino_speed(left)
  try:
    self.bus.write_i2c_block_data(Device.Arduino, _create_command(CommandType.Motor, MotorCommand.SetMotorstoSpeed, acceleration), [right, left])
    time.sleep(self.MIN_I2C_DELAY)
  except IOError:
    time.sleep(self.MIN_I2C_DELAY)
    self.stop()
```

I2C 통신은 데이터를 주고 받는 선(SDA, Serial DAta)과 송수신 타이밍 동기화를 위한 선(SCL, Serial CLock) 하나로 이루어져 있습니다. 하나의 마스터에 최대 127개의 슬레이브로 구성될 수 있으며, 주미의 경우 라즈베리파이 제로가 마스터이며 아두이노가 슬레이브가 되는 형식입니다. 라즈베리파이 제로의 3번, 5번 핀이 각각 SDA, SCL이며, 이는 아두이노에 연결이 되어 있네요.

![라즈베리파이 핀 구성](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcRoHzn%2FbtqIDET7rQM%2FDapUZWK8VAekgQ29x4tmIk%2Fimg.png)

# Reference
- zumi: https://pypi.org/project/zumi/
- PID: https://ko.wikipedia.org/wiki/PID_%EC%A0%9C%EC%96%B4%EA%B8%B0
- I2C: https://m.blog.naver.com/yuyyulee/220323559541
