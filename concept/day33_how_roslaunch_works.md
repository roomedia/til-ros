# 서론
이전에 `rosnodejs`로 `nodemon`을 실행하기 위해 `rosrun`의 소스코드를 살펴봤었습니다. 쉬뱅(She-bang) 혹은 샤뱅이라 불리는 `#!/usr/bin/env nodemon` 구문을 추가하여 해결했었죠. 이러한 방법으로 `rosrun`을 통해 자바스크립트 파일을 실행할 때 `nodemon`을 이용할 수 있었습니다. 그런데... `roslaunch`에는 안 통합니다... 오늘은 `roslaunch`를 통해 노드를 실행할 때 `nodemon`을 통해 실행할 수 있도록 만들어보겠습니다.

# which roslaunch
`roslaunch`의 정체를 알기 위해 해당 파일이 위치한 경로로 이동하여 어떤 파일 형식과 내용을 가지고 있는지 파악해보도록 하겠습니다. 다음 명령어를 통해 `roslaunch`의 경로를 획득할 수 있습니다.

```sh
which roslaunch
```

추가로 다음 명령어를 사용하여 해당 파일의 내용을 바로 알 수 있습니다.

```sh
cat $(which roslaunch)
```

에엥~? `rosrun`과는 다르게 파이썬 파일이었네요.. 내용은 간단하게 `roslaunch` 패키지를 불러와 `main()` 함수를 실행하는 것입니다.

```sh
#!/usr/bin/python
# 기타 라이센스 구문
import roslaunch
roslaunch.main()
```

`roslaunch`의 소스코드 또한 공개되어 있습니다. `main()` 함수 이외의 소스코드는 아래 링크에서 확인하실 수 있습니다.

http://docs.ros.org/hydro/api/roslaunch/html/roslaunch-module.html

`main()` 함수에서 주어진 `.launch` 파일을 실행하는 부분은 다음과 같습니다.

```py
# __init__.py
# ...
def main(argv=sys.argv):
    # ...
    from . import parent as roslaunch_parent 
    try: 
        # force a port binding spec if we are running a core 
        if options.core: 
            options.port = options.port or DEFAULT_MASTER_PORT 
        p = roslaunch_parent.ROSLaunchParent(uuid, args, is_core=options.core, port=options.port, local_only=options.local_only, verbose=options.verbose, force_screen=options.force_screen) 
        p.start() 
        p.spin() 
    # ...
```

`roslaunch_parent.ROSLaunchParent`는 매개변수로 주어진 내용을 모두 `self`의 멤버변수로 설정하며, `p.start()`를 통해 노드를 시작, `p.spin()`을 통해 반복적으로 내용을 수행합니다. `p.start()`의 내용은 다음과 같습니다.

```py
# parent.py
    # ...
    def start(self, auto_terminate=True): 
        """ 
        Run the parent roslaunch. 

        @param auto_terminate: stop process monitor once there are no 
        more processes to monitor (default True). This defaults to 
        True, which is the command-line behavior of roslaunch. Scripts 
        may wish to set this to False if they wish to keep the 
        roslauch infrastructure up regardless of processes being 
        monitored. 
        """ 
        self.logger.info("starting roslaunch parent run") 

        # load config, start XMLRPC servers and process monitor 
        try: 
            self._start_infrastructure() 
        except: 
            # infrastructure did not initialize, do teardown on whatever did come up 
            self._stop_infrastructure() 
            raise 

        # Initialize the actual runner.  
        # Requires config, pm, server and remote_runner 
        self._init_runner() 

        # Start the launch 
        self.runner.launch() 

        # inform process monitor that we are done with process registration 
        if auto_terminate: 
            self.pm.registrations_complete() 

        self.logger.info("... roslaunch parent running, waiting for process exit") 
        if self.process_listeners: 
            for l in self.process_listeners: 
                self.runner.pm.add_process_listener(l) 
```

코드를 더 파봐도 실행 파일 타입을 지정하는 문구가 없길래, 혹시나 하는 마음에 `--screen` 옵션을 붙여 노드가 실행되며 발생하는 출력을 모두 화면에 표시해봤습니다.

```sh
roslaunch delivery_nodejs union.launch --screen
```

`nodemon`은 정상적으로 실행되고 있었습니다. 그렇다면 왜 소스코드를 수정했을 때 노드가 자동 재실행되는, `nodemon`의 기본 기능이 실행되지 않는 걸까요? 반면 `rosrun`으로 실행할 때는 변경 사항에 따라 자동으로 재실행되는 모습을 볼 수 있습니다.

![변경 사항을 감지하여 nodemon이 자동 재실행 되는 모습](https://blog.kakaocdn.net/dn/b2x4i2/btqGbOGsqMi/0IOkFjH7mk0BN3YpOoMkLk/img.png)

두 명령어의 차이는 명령어 실행 시 실행되는 경로가 다르기 때문입니다. 이를 테스트 하기 위해 다음과 같은 노드를 만들겠습니다. 저는 기존 파일의 첫 두 줄을 수정하여 테스트 했습니다.

```sh
#!/bin/bash
pwd
```

해당 노드를 `rosrun`을 통해 실행할 경우, 실행 시 출력되는 경로는 현재 경로와 같습니다.

![rosrun의 경로는 현재 경로와 같다](https://blog.kakaocdn.net/dn/dtZREl/btqGewRW7rM/GlYSiL9oU1SkWhZcY2HVK1/img.png)

`roslaunch`를 통해 실행할 경우 경로가 `$HOME/.ros`로 변경됩니다.

![roslaunch의 경로로는 ~/.ros가 나온다](https://blog.kakaocdn.net/dn/bl8i1B/btqGbqeLZJp/5WP57ipK5HZ80MQ7EDA6y1/img.png)

정확한 검증을 위해 `$HOME/.ros`로 이동하여 `rosrun`을 실행한 뒤, 소스코드를 변경하여 보겠습니다.

![~/.ros에서 실행한 경우](https://blog.kakaocdn.net/dn/cvXQcy/btqGdDjVSgG/QLgpOV37yVxLfd2AQIszw1/img.png)

보시는 것처럼 변경 사항을 캐치하지 못합니다. 사실 이외에도 소스코드가 현재 경로의 하위 폴더로 존재하지 않으면, 변경 사항을 캐치하지 못하는 모습을 볼 수 있습니다.

![~/snap에서 실행한 경우](https://blog.kakaocdn.net/dn/vDXRt/btqGfttInYD/czViDVon2kFTaptAkrMikk/img.png)

이는 `nodemon`에 별도의 옵션을 주지 않을 경우, 현재 작업 경로의 변경 사항을 감지하기 때문입니다. 따라서 이를 특정해주면 `roslaunch`를 사용하여도 변경 사항을 감지하여 재실행 할 수 있겠죠?

...

아쉽게도 `#!` 구문을 사용하면서 `nodemon`에 `--watch` 파라미터를 주는 방법은 없는 것 같습니다... 대신 `roslaunch`에서 `cwd`(current working directory) 구문을 이용하여 노드를 실행할 때의 작업 경로를 설정할 수 있었는데요! `cwd`를 설정하고 `--screen` 옵션을 붙이지 않아도 내용이 출력되도록 변경한 `union.launch` 파일은 다음과 같습니다.

```xml
<launch>
  <group>
    <node pkg="delivery_nodejs" type="app.js" name="menu_service_app" output="screen" cwd="node"/>
    <node pkg="delivery_nodejs" type="server.js" name="menu_service_server" output="screen" cwd="node"/>
  </group>
</launch>
```

더욱 자세한 옵션은 아래 위키 페이지에서 확인할 수 있습니다.

[http://wiki.ros.org/roslaunch/XML/node](http://wiki.ros.org/roslaunch/XML/node)