# 서론
어제 얘기했던대로 오늘은 웹 메뉴판을 누르면 터틀봇이 지정된 위치로 움직이도록 만들어보겠습니다.

# 토픽
지난 시간에 터틀봇을 움직이게 만드는 토픽을 확인했습니다. 해당 토픽은 `geometry_msgs/PoseStamped` 메시지를 사용하는 `/move_base_simple/goal`입니다. 내용은 다음과 같습니다.

```py
# geometry_msgs/PoseStamped.msg
# A Pose with reference coordinate frame and timestamp
Header header
Pose pose
```

헤더와 포즈의 내용은 각각 다음과 같습니다.

```py
# std_msgs/Header.msg
# Standard metadata for higher-level stamped data types.
# This is generally used to communicate timestamped data 
# in a particular coordinate frame.
# 
# sequence ID: consecutively increasing ID 
uint32 seq
#Two-integer timestamp that is expressed as:
# * stamp.sec: seconds (stamp_secs) since epoch (in Python the variable is called 'secs')
# * stamp.nsec: nanoseconds since stamp_secs (in Python the variable is called 'nsecs')
# time-handling sugar is provided by the client library
time stamp
#Frame this data is associated with
string frame_id
```

```py
# geometry_msgs/Pose.msg
# A representation of pose in free space, composed of position and orientation. 
Point position
Quaternion orientation
```

포즈는 다시 Point(x, y, z)와 Quaternion(x, y, z, w)으로 나뉩니다.

# Header
헤더의 구성 요소가 `sequence ID`, `stamp`, `frame_id`로 이루어진다는 건 알게 되었지만, `rosnodejs`에서 헤더를 어떻게 만들어야 할까요? 실제 시간을 저장하는 `stamp`의 경우 `rosnodejs.Time.now()`를 통해 얻을 수 있으며, `pose`의 경우, 다음과 같이 json으로 직접 설정할 수 있습니다.

```js
const { PoseStamped } = rosnodejs.require('geometry_msgs').msg;
const poseStamped = new PoseStamped();
poseStamped.header.stamp = rosnodejs.Time.now();
poseStamped.pose = {
  position: {
    x: 0.1,
    y: 0.1,
    z: 0.0,
  }, 
  orientation: {
    x: 0.0,
    y: 0.0,
    z: 0.0,
    w: 1.0,
  }
};
```

필요에 따라 `seq`와 `frame_id`도 직접 수정하면 되겠네요^^;

이제 publisher를 만들고, 메뉴가 클릭될 때마다 메시지를 설정한 뒤 publish 합니다.

```js
const robot_service = nh.advertise('/move_base_simple/goal', PoseStamped);
const moveRobotByMenuType = (menu) => {
  // set message
  poseStamped.header.stamp = rosnodejs.Time.now();
  poseStamped.pose = posAndRotate[menu];
  // logging
  rosnodejs.log.info('\n', poseStamped.header.stamp);
  rosnodejs.log.info('\n', poseStamped.pose);
  // publish
  robot_service.publish(poseStamped);
}
```

![rostopic echo /move_base_simple/goal](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcoT4Jo%2FbtqGetUW9vg%2FSAghHwp13iX8daC2KlZDjk%2Fimg.png)

`rostopic`을 통해 확인해보았습니다. 오늘은 시간이 너무 늦어 로봇을 테스트해볼 순 없고, 다음에 해보면서 직접 좌표를 수정해야 할 듯^^*;