# rosnodejs service server/client 만들기
이전 시간에 catkin doc에서 제공하고 있는 메시지, 서비스, 액션 패키지 생성 방법을 살펴봤습니다.

이를 이용하여 간단한 msg pub/sub, srv server/client를 만들어 보도록 하겠습니다.

delivery service robot의 간략화된 버전으로, nodejs에서 3개의 메뉴 중 하나를 클릭하면 turtlesim이 2차원 좌표를 이동해 물건 위치에 도달하고, 다시 제자리로 돌아오는 예제입니다.

## 환경 세팅
`rosnodejs`를 사용하기 위해선, 먼저 `nodejs`와 `npm`을 설치해야 합니다. `nodejs`는 12.18.2 LTS 버전을 설치하겠습니다. 이를 설치하면 `npm`이 자동으로 설치됩니다.
```sh
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```
프로젝트를 진행하기에 앞서, 폴더를 하나 생성합니다. 이 프로젝트는 다양한 패키지를 사용하기 때문에 다른 패키지와 혼동되는 것을 막기 위해 아래 폴더에 패키지를 생성하도록 하겠습니다.
```sh
cd ~/catkin_ws/src
mkdir delivery_service_robot
cd delivery_service_robot
```

## 독립된 토픽(메시지, 서비스, 액션) 패키지
이전 시간에 숙지한대로, 프로젝트 전체에서 사용할 토픽을 모아놓은 패키지를 만들겠습니다.
```sh
catkin_create_pkg delivery_topics
```
`delivery_topics`의 `package.xml`을 다음과 같이 작성합니다. `message_generation`은 빌드할 때만, `message_runtime`은 export와 실행 시에만 사용하는 의존성입니다.
```xml
<?xml version="1.0"?>
<package format="2">
  <name>delivery_topics</name>
  <version>0.0.0</version>
  <description>The delivery_topics package</description>
  <maintainer email="roomedia@naver.com">EngHyu</maintainer>
  <license>Apache 2.0</license>

  <buildtool_depend>catkin</buildtool_depend>

  <depend>std_msgs</depend>
  <build_depend>message_generation</build_depend>
  <build_export_depend>message_runtime</build_export_depend>
  <exec_depend>message_runtime</exec_depend>

</package>
```
`CMakeLists.txt`도 아래와 같이 수정합니다. 이후 사용할지 모르는 message, action 관련 함수는 주석 처리한 상태입니다.
```
cmake_minimum_required(VERSION 3.0.2)
project(delivery_topics)

find_package(
  catkin REQUIRED COMPONENTS
  message_generation
  std_msgs
)

# add_message_files(
#   FILES
#   Message1.msg
#   Message2.msg
# )

add_service_files(
  FILES
  MenuSelector.srv
)

# add_action_files(
#   FILES
#   Action1.action
#   Action2.action
# )

generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(
  CATKIN_DEPENDS
  message_runtime
  std_msgs
)
```
`srv` 폴더를 만든 뒤, `MenuSelector.srv` 파일을 만들어 아래 내용으로 채웁니다. 해당 파일은 유저가 웹사이트에서 메뉴를 선택할 때 서버에 메뉴 타입을 전달하며, 서버는 현재 상황을 되돌려줍니다.
```
uint8 menu
---
string message
```

## rosnodejs 패키지 만들기
이제 메뉴판을 띄울 서비스 서버 패키지를 만들겠습니다. `delivery_service_robot` 폴더 아래에 `delivery_nodejs` 패키지를 생성합니다.
```sh
cd ~/catkin_ws/src/delivery_service_robot
catkin_create_pkg delivery_nodejs
cd delivery_nodejs
npm init -y
npm install rosnodejs ejs express
```
node 패키지 `rosnodejs`의 의존성은 node의 `package.json`가 관리하기 때문에, 따로 `package.xml`이나 `CMakeLists.txt`에 포함시킬 필요가 없습니다. 각각의 파일 내용을 다음과 같이 변경합니다.
#### `package.json`
```json
{
  "name": "delivery_nodejs",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "nodemon scripts/app.js",
    "server": "nodemon scripts/server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "ejs": "^3.1.3",
    "express": "^4.17.1",
    "rosnodejs": "^3.0.2"
  }
}
```
#### `package.xml`
```xml
<?xml version="1.0"?>
<package format="2">
  <name>delivery_nodejs</name>
  <version>0.0.0</version>
  <description>delivery service robot web menu plate package</description>
  <maintainer email="roomedia@naver.com">EngHyu</maintainer>
  <license>Apache 2.0</license>

  <buildtool_depend>catkin</buildtool_depend>

</package>
```
#### `CMakeLists.txt`
```
cmake_minimum_required(VERSION 3.0.2)
project(delivery_nodejs)

find_package(catkin REQUIRED)
catkin_package()
```
이제 해당 패키지에 `scripts/` 폴더를 만들고 해당 폴더에 `app.js`, `index.ejs`, `server.js` 파일을 생성합니다. 해당 파일들이 실행 가능하도록 권한을 설정합니다.
```sh
roscd delivery_nodejs
mkdir scripts
cd scripts
touch app.js index.ejs server.js
# 권한 설정 필수!!!
chmod +x *
```
실행 권한이 없으면 실행 시 다음과 같은 에러가 발생합니다.
```
# 에러!!!
root@84af844ca3ea:~/catkin_ws# rosrun delivery_nodejs talker.js
[rosrun] Couldn't find executable named talker.js below /root/catkin_ws/src/delivery_nodejs
[rosrun] Found the following, but they're either not files,
[rosrun] or not executable:
[rosrun]   /root/catkin_ws/src/delivery_nodejs/scripts/talker.js
```
각각의 파일 내용은 아래와 같습니다.
#### `app.js`
```js
#!/usr/bin/env nodemon
// prepare for ROS
const rosnodejs = require('rosnodejs');
const { MenuSelector } = rosnodejs.require('delivery_topics').srv;

// init node and get node handle
rosnodejs.initNode('/menu_selector_client');
const nh = rosnodejs.nh;

// set menu file path
const template = __dirname + '/index.ejs';
const num = [1, 10, 100];

// make express app
const express = require('express');
const app = express();

// to show service response.message
app.set('view engine', 'ejs');
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// default page
app.get('/', (req, res) => {
  res.render(template, { num });
})

// to solve "Unhandled Promise Rejections"
const doAsync = fn => async (req, res, next) => await fn(req, res, next).catch(next);

// menu requested, srv requested
app.post('/', doAsync(async(req, res) => {
  // set request message
  const srvReq = new MenuSelector.Request();
  srvReq.menu = req.body.menu;

  // check number
  if (num[srvReq.menu] == 0) {
    res.render(template, {
      num,
      message: "I'm sorry, but it's sold out...",
    });
    return;
  }

  // reduce number of selected menu
  num[srvReq.menu]--;

  // create the client and get return message
  const client = nh.serviceClient('/menu_selector', MenuSelector)
  const { message } = (await client.call(srvReq))

  // response message will be displayed
  res.render(template, {
    num,
    message,
  })
}));

// host web
app.listen(8080, (req, res) => {
  console.log('now runnig!');
})
```
#### `index.ejs`
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Select Menu</title>
  <style>
    body {
      height: 80vh;
    }

    form {
      height: 100%;
    }

    button {
      width: 100%;
      height: 33.3%;;
    }
  </style>
</head>
<body>
  <form action="/" method="post">
    <button name="menu" value="0"><h2>Salad: <%= num[0] %></h2></button><br>
    <button name="menu" value="1"><h2>Fried Potatoes: <%= num[1] %></h2></button><br>
    <button name="menu" value="2"><h2>Drinks: <%= num[2] %></h2></button>
  </form>
  <% if (hasOwnProperty('message')) { %>
    <h2><%= message %></h2>
  <% } %>
</body>
</html>
```
#### `server.js`
```js
#!/usr/bin/env nodemon
// prepare for ROS and get custom message
const rosnodejs = require('rosnodejs');
const { MenuSelector } = rosnodejs.require('delivery_topics').srv;

// init node and get node handle
rosnodejs.initNode('/menu_selector_server');
const nh = rosnodejs.nh;

// define callback
const cbSelectMenuRtnMessage = (req, res) => {
  const menu = ['salad', 'fried potatoes', 'drinks'];
  res.message = 'You just selected ' + menu[req.menu] + '!';
  rosnodejs.log.info('menu type: ', req.menu);
  rosnodejs.log.info(res.message);
  return true;
}

// create the service
const service = nh.advertiseService('/menu_selector', MenuSelector, cbSelectMenuRtnMessage);
```

## 실행하기
먼저 환경변수(`setup.bash`)와 `roscore`가 실행되고 있는지 확인합니다. `delivery_topics`를 수정하였다면 수정사항 반영을 위해 `catkin_make`를 실행해야 합니다.
```
pushd ~/catkin_ws
catkin_make
popd
```
이후 두 개의 터미널 창에서 각각 서버와 클라이언트를 띄워볼 수 있습니다. 먼저 서비스 서버를 띄우겠습니다.
```
rosrun delivery_nodejs server.js
```
다음과 같은 출력이 표시되어야 합니다.
```
[nodemon] 2.0.4
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node /root/catkin_ws/src/delivery_service_robot/delivery_nodejs/scripts/server.js __name:=menu_service_server __log:=/root/.ros/log/db5c6f24-c965-11ea-bcc7-0242ac110002/ns1-menu_service_server-2.log`
[INFO] [1595142112.117] (ros): Connected to master at http://localhost:11311!
```
서비스 클라이언트이자 웹 서버를 띄워보겠습니다.
```
rosrun delivery_nodejs app.js
```
다음과 같은 출력이 표시되어야 합니다.
```
[nodemon] 2.0.4
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node /root/catkin_ws/src/delivery_service_robot/delivery_nodejs/scripts/server.js __name:=menu_service_server __log:=/root/.ros/log/db5c6f24-c965-11ea-bcc7-0242ac110002/ns1-menu_service_server-2.log`
now runnig!
[INFO] [1595142112.117] (ros): Connected to master at http://localhost:11311!
```
[127.0.0.1:8080](http://127.0.0.1:8080)에 접속하면 다음과 같은 화면이 표시되어야 합니다. 버튼을 누르면 response.message가 출력되며, 서버에서는 아래와 같은 출력이 생성됩니다.
```
[INFO] [1595143417.537] (ros): menu type:  1
[INFO] [1595143417.541] (ros): You just selected fried potatoes!
```

## roslaunch로 한 번에 실행하기
`roslaunch`를 이용하여 여러 개의 노드를 한 번에 띄울 수 있습니다. `delivery_nodejs` 패키지 아래에 `launch` 폴더를 생성하고, `union.launch` 파일을 생성합니다. `union.launch` 파일의 내용은 다음과 같습니다.
```xml
<launch>
  <group>
    <node pkg="delivery_nodejs" type="app.js" name="menu_service_app"/>
    <node pkg="delivery_nodejs" type="server.js" name="menu_service_server"/>
  </group>
</launch>
```
이제 아래와 같은 코드로 서버/클라이언트를 한 번에 실행할 수 있습니다.
```
roslaunch delivery_nodejs union.launch --screen
```

## github
위에서 설명한 코드는 깃허브에 공개되어 있습니다.

https://github.com/EngHyu/ros_delivery_service_robot

## 출처
https://github.com/ROBOTIS-GIT/turtlebot3_deliver

https://github.com/RethinkRobotics-opensource/rosnodejs_examples

https://roscon.ros.org/2017/presentations/ROSCon%202017%20rosnodejs.pdf