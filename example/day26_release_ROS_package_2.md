# 패키지 최초 배포하기
이 가이드는 **bloom**과 **catkin**을 사용하여 새로운 ROS 패키지를 배포하도록 도와줍니다. bloom은 패키지를 공개 ROS build farm에 배포하며, 이 튜토리얼은 해당 방법을 다룹니다. 배포 패키지 정보와 정책에 대해 더 알고 싶다면 http://build.ros.org/를 참고하세요.

# 1. 배포 준비
1. [bloom을 설치합니다](http://ros-infrastructure.github.com/bloom/). 패키지를 배포하기 위해 사용합니다.
1. 코드가 최신 상태인지, 모든 테스트를 통과하는지, 모든 변경 사항을 upstream 레포지토리에 반영했는지 확인하세요. upstream 레포지토리는 기능을 개발하고 패키지 소스 코드를 제공하는 레포지토리입니다. 이 레포지토리는 어디서든 (로컬에서도) 호스팅될 수 있고, git, hg, 또는 svn 레포지토리나 아카이브 파일이 될 수도 있습니다. (현재 tar.gz만 지원하지만 tar.bz와 zip 파일 또한 지원 예정입니다).
<br><br>
Bloom은 패키지를 배포하기위한 필수 도구입니다. upstream 레포지토리가 vcs (git, hg, 또는 svn)이라면, 배포할 버전과 일치하는 태그가 있어야 합니다. 예를 들어, 0.1.0 버전의 패키지를 배포하고 싶다면 bloom은 upstream 레포지토리에서 0.1.0 태그를 찾습니다. **나머지 튜토리얼을 통해 이러한 태깅 작업을 자동으로 수행할 수 있으니, 지금 직접 할 필요는 없습니다.**
<br><br>
만약 커스텀 버전 태깅 체계를 사용하고 싶다면, bloom에서 배포 트랙을 구성할 때 'Release Tag' 구성을 이용해 처리할 수 있습니다.

1. (ROS 1 배포 시) [배포 전 지시사항](http://wiki.ros.org/bloom/Tutorials/PrereleaseTest)을 따라 배포 전 테스트를 수행하세요. ROS 1에서만 동작하고, 부수적인 과정이지만, 매우 추천됩니다. 배포 전 테스트는 패키지가 다른 컴퓨터에 (실제로는 Docker 컨테이너에) 설치되었을 때 패키지가 빌드되고 테스트를 통과하는지 확인합니다. 배포 전 테스트를 통해 배포판에 생길 이슈를 줄일 수 있습니다.

# 2. Upstream 레포지토리 준비
이 페이지는 Bloom 배포를 위해 레포지토리에 설정해야 할 것을 소개합니다.
> Note: 배포 절차의 해당 부분은 새로운 빌드 시스템을 사용하는, ROS 2 패키지에선 동작하지 않습니다. ROS 2를 사용하고 있다면, changelog와 release 태그를 직접 만들어야 합니다. 더 알고 싶다면 [ROS 2 Bloom 페이지](https://index.ros.org/doc/ros2/Tutorials/Releasing-a-ROS-2-package-with-bloom/)를 참고하세요.

## 2.1 Changelog 업데이트
선택 사항이지만 권장됩니다. (출처: [REP-132](http://ros.org/reps/rep-0132.html))
### 2.1.1 Changelog 생성
```
catkin_generate_changelog
```
`catkin_generate_changelog`를 실행하여 `CHANGELOG.rst` 파일을 생성하거나 업데이트 합니다. 하나 이상의 패키지가 `CHANGELOG.rst` 파일을 가지고 있지 않다면, `--all` 옵션을 추가하여 각각의 패키지에 대한 모든 커밋을 이전합니다.

### 2.1.2 Changelog 정리
`catkin_generate_changelog`는 단순히 커밋 로그를 옮겨주는 명령어이기 때문에 changelog에 적합하지 않을 수 있습니다. `CHANGELOG.rst`를 열고 원하는대로 수정합니다. 여기 [잘 정리된 CHANGELOG.rst의 예시](https://github.com/ros/catkin/blob/groovy-devel/CHANGELOG.rst)가 있습니다.

다음 과정을 잊지마세요!
> Note: 형식이 알맞지 않은 CHANGELOG.rst는 패키지 내에서 문제를 일으킬 수 있습니다.

> Note: 언더스코어(\_)로 끝나는 커밋 메시지가 있다면 (e.g. name\_) 에러가 발생합니다. RST Changelog 포맷에서 사용하는 멤버변수와 같아 RST가 [link targets](http://docutils.sourceforge.net/docs/user/rst/quickstart.html#sections)로 취급하기 때문이며, 에러 메시지는 다음과 같습니다.
<br><br>
`
<string>:21: (ERROR/3) Unknown target name: "name".
`
<br><br>
이러한 문제를 해결하려면, escape를 사용하여 다음과 같이 고쳐야 합니다.
<br><br>
`* fix for checking the ``name_`` `

### 2.1.3 Changelog 커밋
**중요한 부분이니 절대 빼먹으면 안 됩니다!** 새로운/업데이트 된 changelog를 커밋하세요.
> Note: 커맨드 라인 플래그와 같은 catkin_generate_changelog에 대한 추가 정보는 원본 [논의 스레드](https://groups.google.com/forum/?hl=en#!msg/ros-sig-buildsystem/EQ4fzwvwYw0/245SJSFGqPMJ)에서 찾아볼 수 있습니다. (이 레퍼런스는 이메일 논의 스레드가 아니라 더 공식적인 문서로 변경되어야 합니다.)

## 2.2 package.xml 버전 업데이트
반드시 `package.xml`의 버전을 올려야 하며 upstream 레포지토리에 버전과 일치하는 태그를 만들어야 합니다. [catkin](http://wiki.ros.org/catkin)이 이를 위한 도구를 제공하며, 이름은 `catkin_prepare_release`입니다.
```sh
cd /path/to/your/upstream/repository
catkin_prepare_release
```
이 명령어는 upstream 레포지토리에서 모든 다른 패키지를 찾아 changelog가 있는지 검사합니다. (커밋되지 않은 로컬 변경 사항도 검사합니다.) `package.xml` 버전을 올리고 변경 사항과 bloom 호환 플래그를 커밋/태그합니다. 해당 명령어를 사용하여 패키지를 배포하는 방법이 가장 권장되고 일관성 있는 방법입니다.

기본적으로 이 명령어는 패치 버전을 0.1.1 -> 0.1.2와 같은 식으로 증가시키나, `--bump` 옵션을 통해 마이너, 메이저 버전을 선택할 수 있습니다.

`catkin_prepare_release`를 사용하지 않더라도, upstream 레포지토리의 태그와 매칭되는 하나 이상의 유효한 `package.xml`을 가지고 있어야 합니다.

# 3. 배포 레포지토리 생성
다음 단계는 배포 레포지토리를 만드는 것입니다. Bloom을 이용하기 위해선 "배포" 레포지토리를 분리하여야 합니다. 이 레포지토리는 반드시 git 레포지토리여야 하며, 보통 GitHub 등의 서비스를 통해 호스팅됩니다. 예를 들어 모든 ROS 배포 레포지토리는 [ros-gbp github organization](https://github.com/ros-gbp)에 있습니다. 이러한 레포지토리는 하나 이상의 catkin 패키지를 포함하는 레포지토리에서 bloom을 실행한 결과물입니다.

배포 레포지토리를 [GitHub](https://github.com/)에 두기를 강력히 추천합니다. 이 튜토리얼은 GitHub를 사용하나, 로컬 레포지토리를 이용하거나 다른 서비스에서 호스팅하는 것도 가능합니다.

## 3.1 github.com에서 배포 레포지토리 생성
github organization이나 github user에서 새로운 레포지토리를 생성합니다. 패키지 이름에 `-release` 접미사를 붙여 이름 붙여야합니다. 예를 들어, `ros_comm` 레포지토리에 대응되는 배포 레포지토리의 이름은 `ros_comm-release`입니다. 주의: github.com 레포지토리를 만들 때, **Initialize this repository with a README.md** 옵션을 체크하세요. 해당 파일은 이후 bloom이 배포 버전에 대한 내용으로 채웁니다.

새로운 배포 레포지토리를 생성했다면 패키지를 구성하고 배포할 준비가 된 것입니다. 이후 사용을 위해 배포 레포지토리 url을 기억해놓으세요.

# 4. 패키지 배포
> Note: github에서 두 개의 인증 정보가 허용되어 있다면, 다음 튜토리얼을 먼저 따라하세요: [GitHubManualAuthorization](http://wiki.ros.org/bloom/Tutorials/GithubManualAuthorization)

일반적으로 다음과 같이 사용합니다:
```sh
# 이건 예시입니다, 해당 명령어를 실행하지 마세요, 다음 명령어를 실행하세요
bloom-release --rosdistro <ros_distro> --track <ros_distro> <your_repository_name>
```

레포지토리에서 패키지를 처음으로 배포한다면 (혹은 언제든 새로운 '트랙'을 구성하고 싶다면) `--edit` 옵션을 추가합니다:
```sh
# <ros_distro>를 실제 ROS 배포판 이름으로 변경하세요, e.g. indigo
bloom-release --rosdistro <ros_distro> --track <ros_distro> <your_repository_name> --edit
```

해당 옵션으로 배포 전에 트랙을 수정할 수 있습니다. 처음으로 배포한다면, 트랙이 아직 생성되지 않았으니 bloom이 트랙을 먼저 생성한 후 수정할 수 있게 합니다. `repository_name`이 url이 아니라 `distribution.yaml`에 적혀있는 레퍼런스라는 점에 유의하세요.

위 명령어를 실행하면, 명시된 ROS distro 파일에서 레포지토리 정보를 찾습니다. 배포가 처음이라면, 정보를 찾지 않고 다음과 같이 배포 레포지토리 url을 요청합니다:

```
No reasonable default release repository url could be determined from previous releases.
Release repository url [press enter to abort]:
```

해당 정보는 처음 배포할 때만 필요하며, 반드시 배포 레포지토리 url을 입력해야 합니다. 방금 전 생성한 레포지토리가 배포 레포지토리입니다.

다음으로 bloom은 새로운 레포지토리를 생성할지 물어봅니다.

```
Freshly initialized git repository detected.
An initial empty commit is going to be made.
Continue [Y/n]?
```

계속하려면 `enter`를 누르거나 `y`를 입력하고 `enter`를 누르세요.

이제 bloom은 master 브랜치 (여기에 구성 정보가 저장됩니다.)를 설정하고 배포 정보를 표시합니다.

## 4.1 배포 트랙 구성
bloom은 하나의 배포 레포지토리에서 여러 ROS 배포판에 패키지를 배포할 수 있도록 디자인 되어있습니다. 이를 위해 bloom은 배포 "트랙"을 사용하여 다른 배포 작업에 대한 구성 정보를 유지합니다. 일반적인 catkin 기반 ROS 패키지에는 default 배포 트랙이 추천됩니다.

아까 실행한 `bloom-release` 명령어에서, `--track`을 명시했습니다. 보통 배포할 ROS distro와 같은 이름의 트랙을 생성하여 관리하지만, 다른 이름을 사용해도 상관없습니다.

처음에는 레포지토리 이름을 묻습니다:
```
Repository Name:
  upstream
    Default value, leave this as upstream if you are unsure
  <name>
    Name of the repository (used in the archive name)
  ['upstream']:
```
이름은 사소하지만 추가적인 태그를 제공하고 더 멋진 아카이브 이름을 만드는데 사용될 수 있습니다. 예제가 `foo`라는 이름의 패키지 하나를 포함한 레포지토리이기 때문에, `foo`라고 입력하겠습니다.

다음 구성은 upstream 레포지토리 uri입니다:
```
Upstream Repository URI:
  <uri>
    Any valid URI. This variable can be templated, for example an svn url
    can be templated as such: "https://svn.foo.com/foo/tags/foo-:{version}"
    where the :{version} token will be replaced with the version for this release.
  [None]:
```
이는 매우 중요한 설정 사항입니다; 실제 개발한 내용을 업로드하는 uri를 입력해야 합니다. 배포 레포지토리의 uri를 입력하는 공간이 아닙니다. 이 경우, 우리 코드가 `bar`라는 github organization에 올라간다고 가정한 뒤 `https://github.com/bar/foo.git`을 입력하겠습니다.

다음으로 bloom은 upstream 레포지토리 타입을 보여줍니다.
```
Upstream VCS Type:
  svn
    Upstream URI is a svn repository
  git
    Upstream URI is a git repository
  hg
    Upstream URI is a hg repository
  tar
    Upstream URI is a tarball
  ['git']:
```

예제에서 사용하는 upstream 레포지토리는 git이며, 이외에도 svg, hg 또는 tar 아카이브 파일을 지원합니다.

다음 몇몇 옵션(버전이나 배포 태그)은 기본 설정 그대로 놓아도 괜찮으며, catkin 기반이 아닌 패키지를 배포하지 않는 이상 변경하지 않습니다. `enter`를 눌러 기본 설정 그대로 설정합니다.

다음으로 수정할 옵션은 upstream 개발 브랜치입니다:
```
Upstream Devel Branch:
  <vcs reference>
    Branch in upstream repository on which to search for the version.
    This is used only when version is set to ':{auto}'.
  [None]:
```
이 옵션은 배포 태그할 upstream 레포지토리 브랜치입니다. None으로 설정 시 기본 브랜치가 배포 버전 브랜치로 사용됩니다. 기본 브랜치 이외의 것을 사용하고 싶다면, 해당 브랜치를 선택하세요. 예를 들어, 이번 배포 트랙에서 `indigo-devel` 브랜치를 사용하고 싶다면 `indigo-devel`을 사용합니다. 

다음으로 ROS distro가 요구됩니다:
```
ROS Distro:
  <ROS distro>
    This can be any valid ROS distro, e.g. groovy, hydro
  ['indigo']:
```

해당 트랙이 기반을 둔 ROS distro 이름을 입력하세요.

나머지 구성 사항은 (패치 디렉토리와 배포 레포지토리 푸시 URL) 대부분의 경우 기본에서 변경하지 않습니다.

축하합니다, 성공적으로 배포 트랙을 구성하셨네요.

`bloom-release`만 실행하게 될 것 같지만, bloom에는 다양한 명령어가 있습니다. 많은 bloom 명령어는 접두사 `git-`이 붙어있으며, 이는 해당 명령어가 반드시 git 레포지토리 안에서 실행되어야 함을 의미합니다. 배포 패키지를 수동으로 복제하면, `git-` 접두사가 붙은 명령어를 실행하여 배포 레포지토리를 수동 조작할 수 있습니다. 이러한 명령어 중 하나인 `git-bloom-config`를 사용하여 트랙을 관리할 수 있습니다. `git-bloom-config -h`를 사용하여 배포 트랙을 관리하는 방법에 대해 더 알아보세요.

## 4.2 배포 마치기
레포지토리 구성을 끝내면 `bloom-release`가 많은 일을 합니다. 배포 레포지토리를 복제하여 배포 트랙의 `actions` 섹션에 정의된 배포 작업을 수행하고, 이를 push하고, pull request를 접수합니다. 레포지토리를 올바르게 구성했다면 github 인증을 마친 뒤 bloom 배포가 종료됩니다. 종료되고나면, 새롭게 생성된 pull request url을 제공합니다.

## 4.3 Build Farm에 알리기
`bloom-release` 호출은 pull request를 열지만, 문제가 생겼거나 이를 원치 않을 때 수동으로 pull request를 열 수도 있습니다. **자동 생성 pull request가 성공적으로 열리면, 아래 기술한 수동 방법을 사용할 필요가 없습니다.**

각각의 ROS 배포판에는 GitHub에 호스트 된 distro 파일이 있습니다. hydro 버전은 다음과 같습니다:
https://github.com/ros/rosdistro/blob/master/hydro/distribution.yaml

해당 URL에 방문하여 edit 버튼을 누르고,(Github에 로그인 되어 있어야 합니다.) 변경 사항을 작성한 뒤 "Propose Changes"를 누르면 이 파일에 대한 pull request를 열 수 있습니다.

레포지토리를 추가하려면 다음과 같은 섹션을 채워야합니다:
```
repositories:
  ...
  foo:
    tags:
      release: release/groovy/{package}/{version}
    url: https://github.com/ros-gdp/foo-release.git
    version: 0.1.0-0
  ...
```

release 태그에 올바른 ROS distro를 기입하세요 (위 경우엔 groovy)

배포 레포지토리를 기입할 때에는 소스 레포지토리의 url이 아닌, **https://** url을 입력해야 합니다. 또한, 패키지 버전과 배포판 숫자를 -으로 구분한 전체 버전을 입력해야 합니다. 배포판 숫자는 매번 같은 버전의 패키지를 배포할 때마다 증가하며, 배포 레포지토리에 패치를 추가하거나 배포 설정이 변경되었을 때에도 증가합니다. 또한 패키지를 끼워넣을 때 알파벳 순서에 맞게 입력해야 합니다.

> Note: 레포지토리가 여러 패키지를 포함하고 있다면, 모든 패키지 이름이 distro 파일에 명시되어야 합니다:

```
repositories:
  ...
  foo:
    packages:
      foo_msgs:
      foo_server:
      foo_utils:
    tags:
      release: release/groovy/{package}/{version}
    url: https://github.com/ros-gbp/foo-release.git
    version: 0.1.0-0
  ...
```

> release 태그에 정확한 ROS distro 이름을 입력하는 것도 잊지 마세요!

> Note: 패키지 리스트의 각 항목은 콜론(:)으로 끝나야합니다. 패키지가 레포지토리 root에 없다면, 패키지 경로를 콜론 이후에 표시할 수 있습니다. 예시:
```
    packages:
      foo_msgs: util/foo_msgs
      foo_server: tool/foo_server
```

# 5. 다음 단계
pull request가 받아들여지면, ROS 개발자 중 한 명이 당신의 request를 merge할 것입니다. (보통 매우 빠르게 반영됩니다.) 24-48시간 후, 패키지는 build farm에 의해 빌드되고 building repository로 배포됩니다. 빌드된 패키지는 [shadow-fixed]()와 공개 레포지토리에 주기적으로 동기화되며, 패키지가 공개 ROS debian 레포지토리(i.e. apt-get)에 공개되기까지 한 달 정도 걸릴 수 있습니다. 다음 업데이트(동기화)가 언제인지 확인하려면 [ROS discussion forums](https://discourse.ros.org/)를 확인하세요.