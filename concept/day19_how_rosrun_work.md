# 서론
`rosrun`을 통해 노드 스크립트를 구동할 때, 가 아닌 `nodemon`을 사용하고 싶었습니다. `apt-cache search`를 이용하여 패키지를 살펴보고, 검색을 해보았더니 어떤 도움도 얻을 수 없었습니다.. 애초에 `rosnodejs`에 관한 자료가 많이 없더라고요. 파이썬 못지 않게 편하고 웹도 사용할 수 있고 `catkin_make`도 안 해도 되고 확장 가능성도 무궁무진한데 왜 안 쓰이는지 잘 모르겠습니다.

아무튼! `rosrun`에서 몇 가지 설정만 고치면 될 듯하여, 오픈소스에 기여하여 명성을 얻어볼 생각에, `rosrun` 명령어는 어디에 존재하며, 어떻게 작동하는가에 대해 알아보았습니다.

# 결론
`app.js` 등과 같은 js 파일 최상단에 다음 줄을 추가합니다. 이후 `rosrun` 구동 시 해당 파일이 `nodemon`을 통해 실행됩니다.
```
#!/usr/bin/env nodemon
```

# 과정
## rosrun 위치
`rosrun` 등 우리가 사용하는 모든 명령어의 위치는 다음 명령어를 통해 얻을 수 있습니다.
```sh
which rosrun
```
소스코드를 통해 직접 빌드한 게 아니라면, 보통의 경로는 다음과 같습니다.
```
root@869b5a874518:~# which rosrun
sh: 0: getcwd() failed: No such file or directory
/opt/ros/noetic/bin/rosrun
```
`rosrun`은 bin 폴더에 존재하지만 사실 컴파일 된 프로그램은 아닙니다. 쉘 스크립트인데 확장자만 떼어놓은 파일이기 때문에 에디터를 통해 내용을 확인할 수 있습니다.
```sh
vi /opt/ros/noetic/bin/rosrun
```
다음과 같은 코드를 확인할 수 있습니다.
```sh
#!/usr/bin/env bash
# 사용법을 출력하는 함수
function usage() {
  echo "Usage: rosrun [--prefix cmd] [--debug] PACKAGE EXECUTABLE [ARGS]"
  echo "  rosrun will locate PACKAGE and try to find"
  echo "  an executable named EXECUTABLE in the PACKAGE tree."
  echo "  If it finds it, it will run it with ARGS."
}

# 변수 초기화
args=1
rosrun_prefix=""

# 디버그 전용 함수, 디버그 플래그가 설정되면 내용이 변경됩니다.
function debug() {
  return
}

# 파라미터가 남아있는 동안 반복
while [ $args == 1 ]
do
  case "$1" in
    # help 플래그가 존재할 경우 사용법 출력 후 종료
    "--help" | "-h")
      usage
      exit 0
      ;;
    # prefix 플래그가 존재할 경우
    "--prefix" | "-p")
      # --prefix 다음 오는 인자를 rosrun_prefix로 설정
      rosrun_prefix="$2"
      # 파라미터를 앞에서부터 2개 제거
      shift
      shift
      ;;
    # debug 플래그가 존재할 경우
    "--debug" | "-d")
      # 해당 인자 제거 후 debug 함수가 파라미터를 모두 출력하도록 재선언
      shift
      function debug() { echo "[rosrun]" "$@" ; }
      ;;
    *) # default
      args=0
  esac
done

# 파라미터가 2개 미만인 경우, 사용법 출력 후 종료
if [ $# -lt 2 ]; then
  usage
  exit 0
fi

# 파일 이름이 폴더 형식 '/'으로 끝날 경우 종료
case $2 in
  */) echo "Invalid filename: " $2
    exit 1
    ;;
esac

# 변수 초기화
pkg_name="$1"
file_name="$2"

# 파일이 존재하는지 확인
function inonedir() {
  exe=$1
  shift
  # 모든 파라미터를 각각 배열의 요소로
  list_of_dirs=("$@")
  # 배열을 이터레이트
  for location in "${list_of_dirs[@]}";
  do
    # exe path가 $location으로 시작할 시
    if [[ "$exe" = "$location"* ]] ; then
      echo "yes"
      return
    fi
  done
  echo "no"
}

# 문자열 변수 $CMAKE_PREFIX_PATH가 비어있지 않다면
if [[ -n $CMAKE_PREFIX_PATH ]]; then
  # 임시 변수 _rosrun_IFS
  _rosrun_IFS="$IFS"
  IFS=$'\n'
  # catkin_find 스크립트를 통해 실행 파일 경로를 찾습니다
  # catkin_find 스크립트의 경로는 다음과 같습니다:
  # /opt/ros/noetic/bin/catkin_find
  # rosrun과 마찬가지로 에디터로 내용을 확인할 수 있습니다.
  catkin_package_libexec_dirs=($(catkin_find --without-underlays --libexec --share "$pkg_name" 2> /dev/null))
  # debug 플래그 설정 시 출력
  debug "Looking in catkin libexec dirs: $IFS$(catkin_find --without-underlays --libexec --share "$pkg_name" 2>&1)"
  # IFS를 복구합니다.
  IFS="$_rosrun_IFS"
  unset _rosrun_IFS
fi
# rospack으로 패키지 경로를 찾습니다
pkgdir=$(rospack find "$pkg_name")
debug "Looking in rospack dir: $pkgdir"
# 실행 파일 경로가 하나도 없고, 패키지 경로가 비어있다면 rosrun 종료
if [[ ${#catkin_package_libexec_dirs[@]} -eq 0 && -z $pkgdir ]]; then
  exit 2
fi
# 파일 이름이 폴더를 포함한 경로가 아닐 때
if [[ ! $file_name == */* ]]; then
  # The -perm /mode usage is not available in find on the Mac or FreeBSD
  #exepathlist=(`find $pkgdir -name $file_name -type f -perm /u+x,g+x,o+x`)
  # -L: #3475
  if [[ $(uname) == Darwin || $(uname) == FreeBSD ]]; then
    _perm="+111"
  else
    _perm="/111"
  fi
  debug "Searching for $file_name with permissions $_perm"
  # 실행 권한이 있는 파일 탐색
  exepathlist="$(find -L "${catkin_package_libexec_dirs[@]}" "$pkgdir" -name "$file_name" -type f  -perm "$_perm" ! -regex ".*$pkgdir\/build\/.*" | uniq)"
  _rosrun_IFS="$IFS"
  IFS=$'\n'
  exepathlist=($exepathlist)
  IFS="$_rosrun_IFS"
  unset _rosrun_IFS
  unset _perm
  # 실행 권한 있는 파일이 없을 때 에러 출력 후 종료
  if [[ ${#exepathlist[@]} == 0 ]]; then
    echo "[rosrun] Couldn't find executable named $file_name below $pkgdir"
    # 실행 권한 없는 파일 탐색
    nonexepathlist=($(find -H "$pkgdir" -name "$file_name"))
    # 실행 권한 없는 파일은 있을 때
    if [[ ${#nonexepathlist[@]} != 0 ]]; then
      echo "[rosrun] Found the following, but they're either not files,"
      echo "[rosrun] or not executable:"
      # 각 파일 이름 출력
      for p in "${nonexepathlist[@]}"; do
        echo "[rosrun]   ${p}"
      done
    fi
    exit 3
  # 실행 권한 있는 파일이 2개일 때 (실행 파일을 특정할 수 없을 때)
  elif [[ ${#exepathlist[@]} -eq 2 ]]; then
    # 한 실행 파일이 share와 libexec에 있을 때는 libexec에 있는 실행 파일을 사용합니다.
    # libexec에 있는 파일은 devel 폴더 아래 있는, catkin에 의해 생성된 파일이기 때문입니다.

    # Share 폴더는 devel, install, src 경로를 포함합니다.
    catkin_package_share_dirs=($(catkin_find --without-underlays --share "$pkg_name" 2> /dev/null))
    debug "Looking in catkin share dirs: $IFS$(catkin_find --without-underlays --share "$pkg_name" 2>&1)"

    # Share 폴더 배열 안에 두 경로가 포함되는지 확인
    first_share=$(inonedir "${exepathlist[0]}" "${catkin_package_share_dirs[@]}")
    second_share=$(inonedir "${exepathlist[1]}" "${catkin_package_share_dirs[@]}")

    # no인 경로 = libexec 경로를 exepathlist로 설정합니다.
    if [[ $first_share == "no" && $second_share == "yes" ]]; then
      debug "Assuming ${exepathlist[0]} is a devel-space relay for ${exepathlist[1]}"
      exepathlist=(${exepathlist[0]})
    elif [[ $second_share == "no" && $first_share == "yes" ]]; then
      debug "Assuming ${exepathlist[1]} is a devel-space relay for ${exepathlist[0]}"
      exepathlist=(${exepathlist[1]})
    fi
  fi

  # 여전히 경로가 여러 개인 경우 배열을 출력하고 사용자 선택을 기다립니다.
  if [[ ${#exepathlist[@]} -gt 1 ]]; then
    echo "[rosrun] You have chosen a non-unique executable, please pick one of the following:"
    select opt in "${exepathlist[@]}"; do
      exepath="$opt"
      break
    done
  # 요소가 하나면 해당 요소를 경로로 설정합니다.
  else
    exepath="${exepathlist[0]}"
  fi

# 파일 이름이 폴더를 포함할 때
else
  absname="$pkgdir/$file_name"
  debug "Path given. Looing for $absname"
  # 파일을 찾을 수 없거나 경로를 찾을 수 없을 때
  if [[ ! -f $absname || ! -x $absname ]]; then
    echo "[rosrun] Couldn't find executable named $absname"
    exit 3
  fi
  exepath="$absname"
fi
# 인자 두 개를 제거합니다.
shift
shift
debug "Running $rosrun_prefix $exepath" "$@"
# 경로의 파일을 남은 매개변수와 함께 실행합니다.
# --prefix를 설정하지 않은 이상 $rosrun_prefix는 공백입니다.
exec $rosrun_prefix "$exepath" "$@"
```
..네 결국 어떤 매커니즘으로 실행되는지 확인하려다 경로 확인만 왕창 하는 걸 알아낼 수 있었습니다. 실제 실행은 다음 줄에서 실시됩니다.
```sh
exec $rosrun_prefix "$exepath" "$@"
```
확장자를 `.py`, `.js` 붙여도 결국 실행되는 건 쉘 스크립트 파일 형식이었네요. 파일 첫 두 글자인 `#!`는 2 Byte 매직넘버로, 이 스크립트를 실행시켜줄 프로그램의 경로를 지정하는 역할입니다. 프로그램의 경로는 시스템 환경에 따라 달라질 수 있기 때문에 `#!/usr/bin/env` + `실행 프로그램 이름`의 조합으로 사용합니다.

결국 파일 확장자에 따라 `rosrun`이 실행 프로그램을 달리 한다는 추측은 완전히 틀린거죠.. 첫 줄은 그저 무시하고 넘어갔는데, 사소한 문법의 중요성을 깨닫는 시간이었습니다.. 덤으로 bash의 `shift`나, `$@`, `$*`의 차이 등 재미있는 사실도 알게 되었네요.