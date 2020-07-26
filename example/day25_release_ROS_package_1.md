# 서론
애써 만든 `turtle_teleop`이 널리 퍼졌으면 좋겠어서 `index.ros.org`에 등록하고, `apt`를 통해 설치할 수 있게 만들고 싶습니다. 다음 글을 참고하여 절차를 진행하겠습니다.

http://wiki.ros.org/rosdistro/Tutorials/Indexing%20Your%20ROS%20Repository%20for%20Documentation%20Generation

# Documentation Generation을 S 레포지토리 인덱싱하기
자신만의 ROS 패키지 레포지토릴를 만들었다면, 스택과 패키지를 ROS.org에 등록할 수 있습니다.

## 1. 패키지를 indexer에 올리면 좋은 점
![index image](http://wiki.ros.org/rosdistro/Tutorials/Indexing%20Your%20ROS%20Repository%20for%20Documentation%20Generation?action=AttachFile&do=get&target=wiki.ros.org_packageHeader.png)

(이 이미지는 [여기](https://docs.google.com/drawings/d/1aa8tFOi0L-v_dRYQlvrNc7w1SBFeosHQJRzHl3xoZ94/edit?usp=sharing)에서 수정할 수 있습니다.)
- 패키지 위키 페이지는 `package.xml`로부터 자동으로 정보를 가져옵니다. (위 이미지를 참고)
- (매우 좋은) 부수 효과; 가져온 정보는 검색 엔진이 페이지를 수집할 때 도움이 되며, 패키지의 가시성을 높여줍니다.

## 2. 최초로 패키지를 indexer에 추가할 때
### 2.1 패키지를 indexer에 추가하는 과정
- 먼저 GitHub에서 다음 레포지토리를 포크합니다. https://github.com/ros/rosdistro
![rosdistro_fork](http://wiki.ros.org/rosdistro/Tutorials/Indexing%20Your%20ROS%20Repository%20for%20Documentation%20Generation?action=AttachFile&do=get&target=rosdistro_fork.png)
  - GitHub 레포지토리를 포크하는 법에 대해 더 알고 싶다면 다음을 참고: https://help.github.com/articles/fork-a-repo

- 복사한 레포지토리를 수정하기 위해 Checkout 합니다.
  - git clone git@github.com:my_github_name/rosdistro.git
- 복사한 레포지토리에서 업데이트 하려는 배포판의 디렉토리로 이동합니다.
  - cd rosdistro/<DISTRO_NAME>
- 해당 폴더 내부에서 `distribution.yaml` 파일을 수정하고 `doc` 세션과 함께 커스텀 레포지토리 정보를 추가합니다. (레포지토리의 알파벳 순서를 지키면서)
  - 레포지토리 url은 release 레포지토리가 아닌, upstream 레포지토리를 가리켜야 합니다.
  - 경우에 따라서 레포지토리 엔트리에 다른 레포지토리 이름으로 이루어진 의존성 리스트를 추가할 수 있습니다.
- 아래 스크립트를 rosdistro 최상위 폴더에서 실행하여 파일이 rosdistro 포맷 형식을 따르도록 만듭니다. 파일이 변경되므로, 안전을 기한다면 변경사항을 백업해놓으세요.
  - rosdistro_reformat file://"$(pwd)"/index.yaml
- 커밋하기 전에 변경 사항을 확인하세요
  - git diff distribution.yaml
  - git commit -m "Adding my_repo_name to documentation index for distro" distribution.yaml
- 변경 사항을 `git push origin master`를 이용하여 원격 저장소 레포지토리에 푸시하세요.
- GitHub rosdistro 프로젝트에 당신의 rosinstall 파일을 indexer에 추가해달라는 pull request를 남기세요.
![pull request example](http://wiki.ros.org/rosdistro/Tutorials/Indexing%20Your%20ROS%20Repository%20for%20Documentation%20Generation?action=AttachFile&do=get&target=rosdistro_pull_submit.png)
  - pull request에 대한 명확한 예시: [1](https://github.com/ros/rosdistro/pull/13688/files), [2](https://github.com/ros/rosdistro/pull/13665/files)

## 2.2 간단하게 패키지 위키 페이지 생성하기
1. 계정이 없다면 wiki.ros.org에서 계정을 생성하고 로그인하세요.
2. 만들고 싶은 페이지 이름의 url로 접속하세요.
    - e.g. "http://wiki.ros.org/your_package" for "your_package"
3. 페이지에 접속하면 페이지가 존재하지 않는다고 표시될 것입니다. 적절한 템플릿을 선택하여 페이지를 생성하세요.
    - 패키지의 경우, "PackageTemplate"를 선택합니다.
4. 위키는 패키지에 대한 사용법, 튜토리얼, 리뷰를 비롯한 다양한 콘텐츠 문서를 만드는 걸 장려하고 있습니다. [패키지 문서 작성 가이드라인](http://wiki.ros.org/PackageDocumentation)을 참고하세요.

끝입니다! pull request가 받아들여지면, rosinstall 파일에 대한 문서가 생성됩니다.

# 3. upstream으로부터 pull 하기
github.com/ros/rosdistro에서부터 변경사항을 받아오려면, 다른 원격 저장소를 추가할 필요가 있습니다.
```
git remote add upstream https://github.com/ros/rosdistro.git
```
이제 다른 사람들이 만들고 ROS가 깃허브에 커밋한 변경사항을 가져옵니다.
```
git pull upstream
```
깃이 upstream으로부터 정보를 받아오고 자동으로 브랜치를 머지합니다. 이 과정은 로컬 레포와 업스트림의 충돌 사항을 알리지 않습니다. 다음 명령어를 사용할 때 브랜치를 명시해야 합니다.:
```
git pull upstream master
```
다음 명령어와 같습니다.
```
git fetch upstream
git merge upstream/master
```
이 두 절차의 과정은 로컬 레포와 업스트림 간 충돌 사항이 있을 경우 이를 표시합니다.

# indexer에 패키지가 반영되었는지 확인하기
모든 문서 작업을 추가한 메타/패키지/스택은 젠킨스 서버에 생성됩니다: http://build.ros.org/ (반영까지 시간이 걸릴 수 있습니다)

패키지를 찾으려면 그냥 메타/패키지/스택 이름을 검색창에 입력하기만 하면 됩니다. job 페이지에서 상태, 실패 메시지와 전체 콘솔 출력을 확인할 수 있습니다.

# ROS 패키지 레포지토리를 ROS.org에 문서화하기
레포지토리가 인덱싱되었다면, 문서 작업은 재생성되고 실행됩니다.

위에 설명한 것처럼 위키 패이지를 생성하고, 위키에 문서를 추가하기 위해 다음 규칙을 따르세요:
- [위키 스타일 가이드](http://wiki.ros.org/StyleGuide)
- [패키지 문서화 가이드](http://wiki.ros.org/PackageDocumentation)
- [튜토리얼 작성 가이드](http://wiki.ros.org/WritingTutorials)
- [ROS.org 위키 매크로 가이드](http://wiki.ros.org/WikiMacros)

훌륭한 위키 문서화의 예시:
- [tf](http://wiki.ros.org/tf)
- [carrot_planner](http://wiki.ros.org/carrot_planner)
- [actionlib](http://wiki.ros.org/actionlib)
- [ground_station](http://wiki.ros.org/ground_station)
- [velodyne_driver](http://wiki.ros.org/velodyne_driver)

# 패키지 발표하기
`ros.org`에 인덱싱되면, 다른 어떤 글로벌 커뮤니티보다 가장 눈에 잘 띕니다. 누군가 Google이나 [ROS wiki의 패키지 검색](http://www.ros.org/browse/list.php)을 통해 찾을 수 있습니다. 그러나 사람들에게 작업물에 대해 알리는 건 항상 중요합니다. ([발표가 얼마나 도움이 되는지에 대한 논의](https://discourse.ros.org/t/new-software-release-announcements/1203)를 참고하세요)

이를 위해 [discourse.ros.org "\`general\`" 카테고리](http://discourse.ros.org/c/general)에 새로운 포스트를 작성하세요.

이미 릴리즈 된 패키지에 대한 대규모 업데이트가 있다면, 이러한 내용을 발표하는 것 또한 중요합니다.

## 1. 새로운 패키지 발표를 위한 템플릿
패키지를 발표할 때 무엇을 포함해야 할 지 모르겠다면 다음을 참고하세요:
- 패키지 이름
- 패키지에 대한 짧은 설명 (package.xml을 복사해도 됩니다. package.xml에 설명을 추가하는 걸 잊지 마세요!!)
- wiki.ros.org의 패키지 페이지 링크
- 튜토리얼이 있다면 튜토리얼 링크
- 모든 기여자에 대한 헌사

[많은 명확한 예시](https://www.google.com/search?client=ubuntu&channel=fs&q=site:lists.ros.org+(announce+OR+announcement+OR+announcing)&ie=utf-8&oe=utf-8)를 찾아볼 수 있습니다.