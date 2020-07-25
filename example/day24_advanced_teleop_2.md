[지난글](./day23_advanced_teleop_1.md)

# 라이브러리 선정
가장 중요한 멀티 키 입력을 위해 사용할 라이브러리를 선정하도록 하겠습니다. 파이썬에서 키보드 입력을 제어하는 모듈을 찾아본 바 다음과 같습니다.

모듈 | 특징
---|---
`pygame` | gui 윈도우 필요
`keyboard` | sudo 권한 필요
`pynput` | ssh 사용 시 추가 작업 필요

`pynput`을 사용하였습니다.

## 설치
파이썬 모듈은 다음과 같이 설치할 수 있습니다.
```bash
python -m pip install pynput
```
`pip`가 안 깔려 있으신 분들은 [다음](https://pip.pypa.io/en/stable/installing/)을 참고하세요.

# 코드 분석
전체 코드는 다음에서 확인하실 수 있습니다.

https://github.com/EngHyu/turtle_teleop

https://github.com/EngHyu/turtle_teleop/blob/master/scripts/turtle_teleop_multi_key

파이썬 패키지 구성을 위한 코드는 건너뛰고, 핵심적인 로직만 확인하도록 하겠습니다.

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

from __future__ import print_function
from pynput.keyboard import Key, Listener

import rospy
import sys, select, os
if os.name == 'nt':
  import msvcrt
else:
  import tty, termios
from geometry_msgs.msg import Twist

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
w/s : linear movement (Burger : ~ 0.22, Waffle and Waffle Pi : ~ 0.26)
a/d : angular movement (Burger : ~ 2.84, Waffle and Waffle Pi : ~ 1.82)
stop when key released
ESC to quit
"""

e = """
Communications Failed
"""
# 입력에 필요한 변수, 함수와 그 외의 코드가 구분이 어려워 클래스로 만들어보았습니다.
class Teleop:
    def __init__(self):
        print(msg)
        # 추가된 부분. 현재 입력 중인 키를 집합 형태로 가지고 있습니다.
        self.keys = set()
        self.status = 0
        self.target_linear_vel   = 0.0
        self.target_angular_vel  = 0.0

        # getKey 대신 pynput.keyboard.Listener를 사용하여 입력값을 처리합니다.
        with Listener(
            on_press=self.on_press,
            on_release=self.on_release
        ) as listener:
            listener.join()
    # ...
    def move(self):
        global pub
        try:
            # set의 index 사용 시 값이 없으면 에러가 발생하여
            # string으로 변환 후 find 함수 사용
            # w, s 값이 입력 중이 아니면 병진 속도를 0으로 설정합니다.
            keys = ''.join(self.keys)
            if keys.find('w') == -1 and keys.find('s') == -1 :
                self.target_linear_vel = 0.0
            else :
                # 더 나중에 입력한 값을 우선으로 취급합니다.
                sign = 1 if keys.find('w') > keys.find('s') else -1
                self.target_linear_vel = self.checkLinearLimitVelocity(self.target_linear_vel + sign * LIN_VEL_STEP_SIZE)

            if keys.find('a') == -1 and keys.find('d') == -1 :
                self.target_angular_vel = 0.0
            else :
                sign = 1 if keys.find('a') > keys.find('d') else -1
                self.target_angular_vel = self.checkAngularLimitVelocity(self.target_angular_vel + sign * ANG_VEL_STEP_SIZE)

            self.status += 1
            print(self.vels(self.target_linear_vel, self.target_angular_vel))

            if self.status == 100 :
                print(msg)
                self.status = 0
        
            twist = Twist()
            # 모든 키 release 시 force stop과 같은 효과를 내기 위해
            # smooth velocity를 사용하지 않았습니다.
            twist.linear.x = self.target_linear_vel; twist.linear.y = 0.0; twist.linear.z = 0.0
            twist.angular.x = 0.0; twist.angular.y = 0.0; twist.angular.z = self.target_angular_vel
            pub.publish(twist)

        except Exception as err:
            print(e)
            print(err)

    # 키가 눌렸을 때 호출되는 콜백 함수
    def on_press(self, key):
        # to remove input buffer
        if os.name == 'nt':
            msvcrt.getch()
        else:
            tty.setraw(sys.stdin.fileno())
            termios.tcsetattr(sys.stdin, termios.TCSADRAIN, settings)

        # esc 키를 입력했을 경우 Listener 종료
        if key == Key.esc:
            return False

        # wasd가 아닌, ctrl, alt 등의
        # 특수 키를 입력했을 경우 함수를 조기 종료합니다.
        if not hasattr(key, 'char'):
            return True

        # wasd가 아닐 때에도 조기 종료합니다.
        key = key.char
        if key not in 'wasd':
            return True

        # 상반되는 두 키가 함께 눌렸을 경우,
        # 먼저 눌린 키를 제거합니다.
        if key == 'w' and 's' in self.keys:
            self.keys.remove('s')
        if key == 'a' and 'd' in self.keys:
            self.keys.remove('d')
        if key == 's' and 'w' in self.keys:
            self.keys.remove('w')
        if key == 'd' and 'a' in self.keys:
            self.keys.remove('a')
        
        self.keys.add(key)
        self.move()

    # 키에서 손을 뗐을 때 호출되는 콜백 함수
    def on_release(self, key):
        # 입력 값이 특수 키인지 확인
        # wasd만 처리하기 때문에 특수 키일 경우 조기 종료합니다.
        if not hasattr(key, 'char'):
            return True # False면 Listener 종료

        # 키 코드 값을 가져옵니다.
        key = key.char
        # 해당 키 코드 값이 집합에 있으면 제거합니다.
        if key in self.keys:
            self.keys.remove(key)
            self.move()

if __name__=="__main__":
    if os.name != 'nt':
        settings = termios.tcgetattr(sys.stdin)

    rospy.init_node('turtle_teleop')
    turtlebot3_model = rospy.get_param("model", "burger")
    # topic_type으로부터 topic_name을 도출하여 turtlesim/turtlebot 동시 적용 가능합니다.
    topic_type = rospy.get_param("topic_type", "turtlebot")

    if topic_type == 'turtlebot':
        topic_name = 'cmd_vel'
    else:
        topic_name = 'turtle1/cmd_vel'
    pub = rospy.Publisher(topic_name, Twist, queue_size=10)

    Teleop()

    twist = Twist()
    twist.linear.x = 0.0; twist.linear.y = 0.0; twist.linear.z = 0.0
    twist.angular.x = 0.0; twist.angular.y = 0.0; twist.angular.z = 0.0
    pub.publish(twist)

    # 표준 입력 버퍼 정리
    termios.tcsetattr(sys.stdin, termios.TCSADRAIN, settings)
```

다음과 같이 완성되었습니다. 실제 작동 영상은 아래에서 확인할 수 있습니다.

[![turtle_teleop_multi_key movement](https://img.youtube.com/vi/bDvn6tUCVmk/0.jpg)](https://youtu.be/bDvn6tUCVmk)
