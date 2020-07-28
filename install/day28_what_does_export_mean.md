# 문제 상황 1
터틀봇이 Remote PC가 보내는 `/cmd_vel`을 비롯한 모든 메시지를 받아올 수 없음.

# 해결 방법
Remote PC의 방화벽 문제였습니다. 다음을 실행하여 방화벽에서 터틀봇 IP를 허용해줍니다.

```sh
sudo ufw allow from {TURTLEBOT_IP} # 실제 아이피로 변경합니다.
```

# 문제 상황 2
터틀봇이 Remote PC로부터 `/cmd_vel`을 받아오지만, 반대로 센서값은 받아올 수 없음.

# 해결 방법
`~/.bashrc`에서 해당 구문이 잘못 설정되어 있었습니다.

```sh
# 이러면 오류
# ROS_MASTER_URI=http://{REMOTE_PC_IP}:11311
# ROS_HOSTNAME={TURTLEBOT_IP}

# 이래야 정상
export ROS_MASTER_URI=http://{REMOTE_PC_IP}:11311
export ROS_HOSTNAME={TURTLEBOT_IP}
```

# `export`의 역할
`export`는 환경변수를 설정하는 역할을 합니다. 이는 그냥 `변수_이름=변수_값`와 구분되는데, 바로 해당 구문 이후 생성되는 모든 자식 프로세스에 환경변수가 설정되기 때문입니다. `export` 키워드를 붙이지 않으면, 아무리 `~/.bashrc`에서 변수를 설정한다고 해도, 생성되는 프로세스가 `bash`가 아니라면 해당 환경변수를 볼 수 없죠.

## 자주 사용되는 옵션
- `-p`: 옵션 뒤에 현재 쉘에 설정된 여러 개의 변수를 나열하여 export 할 수 있습니다.
- `-n`: 옵션 뒤에 변수 이름을 붙여 해당 변수의 export를 해제할 수 있습니다.

```sh
a=1
b=2
c=3
export -p a b c
bash               # 새로운 자식 프로세스를 생성하여 확인
echo $a $b $c      # 1 2 3 출력
exit
export -n a b c
sh
echo $a $b $c      # 공백 출력
```