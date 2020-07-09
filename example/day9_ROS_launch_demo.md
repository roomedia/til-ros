# ROS Launch
`rosrun` 대신 `roslaunch`를 이용하여 많은 양의 노드를 한 번에 구동할 수 있습니다. 이때 주의할 점에 대해 알아보도록 하겠습니다. 먼저 대상이 될 패키지 폴더 안에 `launch` 폴더를 생성한 뒤 `*.launch` 형식의 xml 파일을 만듭니다. 대상 패키지는 `ros_tutorials_topic`으로, 이름은 `union.launch`로 하겠습니다.
```xml
<launch>
  <node pkg="ros_tutorials_topic" type="topic_publisher" name="publisher1"/>
  <node pkg="ros_tutorials_topic" type="topic_subscriber" name="subscriber1"/>
  <node pkg="ros_tutorials_topic" type="topic_publisher" name="publisher2"/>
  <node pkg="ros_tutorials_topic" type="topic_subscriber" name="subscriber2"/>
</launch>
```
아래 명령어를 이용하여 구동합니다.
```bash
roslaunch ros_tutorials_topic union.launch --screen
```
`--screen` 옵션을 통해 모든 노드의 출력이 한 터미널에 표시됩니다. `rosnode list`를 통해 확인해보면 아래 이름으로 노드들이 실행되고 있음을 알 수 있습니다.
```bash
/topic_publisher1
/topic_subscriber1
/topic_publisher2
/topic_subscriber2
```
`rqt_graph`를 통해 확인해보면 각 subscriber가 모든 publisher의 topic을 수신하고 있는 걸 알 수 있습니다. 이를 방지하기 위해 `union.launch`를 수정합니다. `<group>` 태그를 사용하여 노드를 그룹화할 수 있으며, `<group>` 태그의 속성인 `ns`를 이용하면 topic 이름 앞에 prefix가 붙어 어떤 topic을 수신해야 할 지 구분할 수 있습니다.

```xml
<launch>
  <group ns="ns1">
    <node pkg="ros_tutorials_topic" type="topic_publisher" name="topic_publisher1"/>
    <node pkg="ros_tutorials_topic" type="topic_subscriber" name="topic_subscriber1"/>
  </group>
  <group ns="ns2">
    <node pkg="ros_tutorials_topic" type="topic_publisher" name="topic_publisher2"/>
    <node pkg="ros_tutorials_topic" type="topic_subscriber" name="topic_subscriber2"/>
  </group>
</launch>
```
다시 `roslaunch`를 통해 구동한 뒤, `rqt_graph`를 사용해보면 각각 하나의 topic만 수신하는 걸 볼 수 있습니다.

## Launch 태그
launch에서 사용되는 태그는 다음과 같습니다.
`<launch>`: roslaunch 구문의 시작과 끝을 가리킵니다.
`<node>`: 실행할 노드를 설정합니다. pkg, type, name 속성이 있으며 각각 패키지, 노드명, 실행명을 설정할 수 있습니다.
`<machine>`: 실행 기기를 설정할 수 있습니다. name, address, env-loader, default, user, password, timeout 속성이 있으며 각각 기기 이름, 주소, 환경 파일(.sh) 경로, 기본 머신 지정 여부, 사용자, 비밀번호, timeout 시간(seconds)을 설정할 수 있습니다.
`<include>`: 다른 패키지나 같은 패키지의 다른 launch를 불러와 하나의 launch로 실행할 수 있습니다. file, ns, clear_params, pass_all_args 속성이 있으며, 각각 파일 이름, 네임스페이스, include 되는 namespace의 파라미터 삭제 여부, 모든 argument 전달 여부를 나타냅니다. ns는 옵션이며, clear_params와 pass_all_args의 초기값은 false입니다.
`<remap>`: 노드 이름, 토픽 이름 등 노드에서 사용 중인 ROS 변수의 이름을 변경할 수 있습니다. from 속성에 원래 변수 이름을, to 속성에 변경될 이름을 넣어주면 됩니다.
`<env>`: 경로, IP 등 환경변수를 설정합니다. name, value 속성이 있습니다.
`<param>`, `<rosparam>`: 파라미터 이름, 타입, 값을 설정합니다. name, value, type, textfile, binfile, command 속성이 있으며, 다음과 같이 YAML file을 불러오는 등 사용할 수 있습니다.
```xml
<param name="params_a" type="yaml" command="cat &quot;$(find roslaunch)/test/params.yaml&quot;" />
<rosparam command="load" file="FILENAME" />
```
`<group>`: ns, clear_params 속성을 가지고 있습니다. namespace는 global, relative 모두 지원하지만 global namespace는 권장되지 않습니다.
`<test>`: 노드를 테스트할 때 사용합니다. `<node>`와 비슷하지만 테스트에 사용하는 옵션이 추가되어 있습니다. pkg, test-name, type, name, args, clear_params, cwd, launch-prefix, ns, retry, time-limit을 속성으로 가지고 있습니다.
`<arg>`: launch 파일 내에서 변수를 정의할 수 있으며, 아래와 같이 실행 시 값을 변경시킬 수 있습니다.
```xml
<launch>
  <arg name="update_period" default="10" />
  <param name="timing" value="$(arg update_period)"/>
</launch>
```
```bash
roslaunch my_package my_package.launch update_period:=30
```