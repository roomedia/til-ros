# turtlebot3_teleop_key
Turtlebot PC에서 Bringup을 실행한 상태에서 터틀봇을 움직일 수 있습니다.
```bash
# Remote PC
roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```
다음과 같은 화면이 표시되며, waxd로 이동, 스페이스바나 s키로 멈출 수 있습니다. 한 번 누르면 그 속도 그대로 계속 이동하기 때문에 조종하기 어렵고 벽에 부딪힐 수 있어 조심해야 합니다.
```bash
 Control Your Turtlebot3!
  ---------------------------
  Moving around:
          w
     a    s    d
          x

  w/x : increase/decrease linear velocity
  a/d : increase/decrease angular velocity
  space key, s : force stop

  CTRL-C to quit
```

# turtle_teleop_key?
그런데 어째 생김새가 `turtlesim turtle_teleop_key`와 비슷합니다.

```sh
# Remote PC Terminal 1
rosrun turtlesim turtle_teleop_key
```
```sh
# Remote PC Terminal 2
rostopic list
```
```sh
# ...
/cmd_vel          # for turtlebot3_teleop_key
/turtle1/cmd_vel  # for turtle_teleop_key
# ...
```

`turtlebot3_teleop_key` 코드를 다음과 같이 고치면 `turtlesim_node`를 움직일 수 있습니다.
```py
# ...
rospy.init_node('turtlebot3_teleop')
# pub = rospy.Publisher('cmd_vel', Twist, queue_size=10)
pub = rospy.Publisher('turtle1/cmd_vel', Twist, queue_size=10)
# ...
```

![modified turtlebot3_teleop moves turtlesim]()

# turtlebot3 안전하게 움직이기
[![teleop thumbnail to youtube](https://i.ytimg.com/vi/Z4s18hlazb4/maxresdefault.jpg)](https://youtu.be/Z4s18hlazb4)

시뮬레이션에서도 확인할 수 있고, 영상에서도 볼 수 있듯 `turtlebot3_teleop_key`는 게임 캐릭터와는 달리 컨트롤러에서 손을 떼도 움직입니다. 이 부분이 터틀봇을 조종할 때 위험하다고 생각되어 게임 캐릭터처럼 움직이도록 변경하고자 합니다. `turtlesim_node`와도 호환되었으면 좋겠습니다.

## 코드 분석
다음은 ROBOTIS에서 제공하는 `turtlebot3_teleop_key` 코드입니다. 원본 코드는 [여기](https://github.com/ROBOTIS-GIT/turtlebot3/blob/master/turtlebot3_teleop/nodes/turtlebot3_teleop_key)에서 확인하실 수 있습니다.
```py
#!/usr/bin/env python

# Copyright (c) 2011, Willow Garage, Inc.
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
#    * Redistributions of source code must retain the above copyright
#      notice, this list of conditions and the following disclaimer.
#    * Redistributions in binary form must reproduce the above copyright
#      notice, this list of conditions and the following disclaimer in the
#      documentation and/or other materials provided with the distribution.
#    * Neither the name of the Willow Garage, Inc. nor the names of its
#      contributors may be used to endorse or promote products derived from
#       this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.

import rospy
from geometry_msgs.msg import Twist
import sys, select, os
# OS가 윈도우인지 구분하여 모듈 임포트
if os.name == 'nt':
  import msvcrt
else:
  import tty, termios

BURGER_MAX_LIN_VEL = 0.22
BURGER_MAX_ANG_VEL = 2.84

WAFFLE_MAX_LIN_VEL = 0.26
WAFFLE_MAX_ANG_VEL = 1.82

LIN_VEL_STEP_SIZE = 0.01
ANG_VEL_STEP_SIZE = 0.1

msg = """
Control Your TurtleBot3!
---------------------------
Moving around:
        w
   a    s    d
        x
w/x : increase/decrease linear velocity (Burger : ~ 0.22, Waffle and Waffle Pi : ~ 0.26)
a/d : increase/decrease angular velocity (Burger : ~ 2.84, Waffle and Waffle Pi : ~ 1.82)
space key, s : force stop
CTRL-C to quit
"""

e = """
Communications Failed
"""

# stdin에서 글자를 읽어옵니다.
# release 되는 키를 확인할 수 없습니다.
def getKey():
    # 윈도우
    if os.name == 'nt':
      return msvcrt.getch()
    # 유닉스 계열
    tty.setraw(sys.stdin.fileno())
    rlist, _, _ = select.select([sys.stdin], [], [], 0.1)
    if rlist:
        key = sys.stdin.read(1)
    else:
        key = ''
    # 입력 버퍼 정상화? 초기화?
    termios.tcsetattr(sys.stdin, termios.TCSADRAIN, settings)
    return key

# 현재 속도 출력
def vels(target_linear_vel, target_angular_vel):
    return "currently:\tlinear vel %s\t angular vel %s " % (target_linear_vel,target_angular_vel)

# 속도를 완만하게 증가시킵니다.
def makeSimpleProfile(output, input, slop):
    if input > output:
        output = min( input, output + slop )
    elif input < output:
        output = max( input, output - slop )
    else:
        output = input

    return output

# 일정 범위 clamp 함수
# checkLinearLimitVelocity/checkAngularLimitVelocity에 사용되어
# 속도가 일정 범위를 넘어가지 못하게 합니다.
def constrain(input, low, high):
    if input < low:
      input = low
    elif input > high:
      input = high
    else:
      input = input

    return input

# 병진 속도 범위 체크 함수
def checkLinearLimitVelocity(vel):
    if turtlebot3_model == "burger":
      vel = constrain(vel, -BURGER_MAX_LIN_VEL, BURGER_MAX_LIN_VEL)
    elif turtlebot3_model == "waffle" or turtlebot3_model == "waffle_pi":
      vel = constrain(vel, -WAFFLE_MAX_LIN_VEL, WAFFLE_MAX_LIN_VEL)
    else:
      vel = constrain(vel, -BURGER_MAX_LIN_VEL, BURGER_MAX_LIN_VEL)

    return vel

# 회전 속도 범위 체크 함수
def checkAngularLimitVelocity(vel):
    if turtlebot3_model == "burger":
      vel = constrain(vel, -BURGER_MAX_ANG_VEL, BURGER_MAX_ANG_VEL)
    elif turtlebot3_model == "waffle" or turtlebot3_model == "waffle_pi":
      vel = constrain(vel, -WAFFLE_MAX_ANG_VEL, WAFFLE_MAX_ANG_VEL)
    else:
      vel = constrain(vel, -BURGER_MAX_ANG_VEL, BURGER_MAX_ANG_VEL)

    return vel

if __name__=="__main__":
    # 윈도우가 아닐 경우 표준 입력 버퍼의 초기값을 저장합니다.
    if os.name != 'nt':
        settings = termios.tcgetattr(sys.stdin)

    rospy.init_node('turtlebot3_teleop')
    # 토픽 이름이 cmd_vel로 고정된 모습입니다.
    pub = rospy.Publisher('cmd_vel', Twist, queue_size=10)

    turtlebot3_model = rospy.get_param("model", "burger")

    status = 0
    target_linear_vel   = 0.0
    target_angular_vel  = 0.0
    control_linear_vel  = 0.0
    control_angular_vel = 0.0

    try:
        print(msg)
        while(1):
            # 키 입력 하나를 받아옵니다.
            key = getKey()
            # if/elif/else 구문까지 사용하여 멀티 키 입력을 받을 수 없습니다.
            # 입력에 따라 속도를 증가/감소시키고 출력합니다.
            # 상태를 1 증가시킵니다.
            if key == 'w' :
                target_linear_vel = checkLinearLimitVelocity(target_linear_vel + LIN_VEL_STEP_SIZE)
                status = status + 1
                print(vels(target_linear_vel,target_angular_vel))
            elif key == 'x' :
                target_linear_vel = checkLinearLimitVelocity(target_linear_vel - LIN_VEL_STEP_SIZE)
                status = status + 1
                print(vels(target_linear_vel,target_angular_vel))
            elif key == 'a' :
                target_angular_vel = checkAngularLimitVelocity(target_angular_vel + ANG_VEL_STEP_SIZE)
                status = status + 1
                print(vels(target_linear_vel,target_angular_vel))
            elif key == 'd' :
                target_angular_vel = checkAngularLimitVelocity(target_angular_vel - ANG_VEL_STEP_SIZE)
                status = status + 1
                print(vels(target_linear_vel,target_angular_vel))
            # 스페이스바나 s가 입력될 때 속도를 0으로 설정합니다.
            elif key == ' ' or key == 's' :
                target_linear_vel   = 0.0
                control_linear_vel  = 0.0
                target_angular_vel  = 0.0
                control_angular_vel = 0.0
                print(vels(target_linear_vel, target_angular_vel))
            # 그 외 입력 처리
            else:
                # ctrl + c를 처리합니다.
                if (key == '\x03'):
                    break

            if status == 20 :
                print(msg)
                status = 0

            twist = Twist()

            # 속도를 완만하게 증가시킨 뒤, 이를 토대로 메시지를 만듭니다.
            control_linear_vel = makeSimpleProfile(control_linear_vel, target_linear_vel, (LIN_VEL_STEP_SIZE/2.0))
            twist.linear.x = control_linear_vel; twist.linear.y = 0.0; twist.linear.z = 0.0

            control_angular_vel = makeSimpleProfile(control_angular_vel, target_angular_vel, (ANG_VEL_STEP_SIZE/2.0))
            twist.angular.x = 0.0; twist.angular.y = 0.0; twist.angular.z = control_angular_vel

            # /cmd_vel 메시지가 퍼블리시 됩니다.
            pub.publish(twist)

    except:
        print(e)

    finally:
        # ctrl + c를 누르면 터틀봇이 더이상 이동하지 않도록 속도를 0으로 만들어줍니다.
        twist = Twist()
        twist.linear.x = 0.0; twist.linear.y = 0.0; twist.linear.z = 0.0
        twist.angular.x = 0.0; twist.angular.y = 0.0; twist.angular.z = 0.0
        pub.publish(twist)

    # 윈도우가 아닐 경우, 표준 입력 버퍼를 원상태로 회복합니다.
    if os.name != 'nt':
        termios.tcsetattr(sys.stdin, termios.TCSADRAIN, settings)
```

몇 가지 유의할 점과 문제점이 보입니다.
먼저 현재 상태로는 멀티 키 입력을 받을 수 없으며, 직접 코드를 수정하지 않는 한 `turtlesim`에 적용할 수 없습니다. 또한, `turtle_teleop` 실행 종료 이후 터미널이 이상해지는 현상을 막기 위해 표준 입력 버퍼를 조작하는 코드는 반드시 사용해야 합니다. 다음 시간에는 이를 개선하여 터틀봇을 조작하는 패키지를 만들어보겠습니다.