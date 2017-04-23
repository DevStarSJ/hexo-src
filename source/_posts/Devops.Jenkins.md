---
title: Install Jenkins
date: 2015-11-10 00:00:00
categories:
- DevOps
tags:
- DevOps
- CI/CD
---

# Install Jenkins

### 1. Jenkins 란 ?

Jenkins Server는 Open Source CI (Continuous Integration) 을 자동화해주는 Tool입니다. 대부분 다음과 같은 방식으로 Jenkins를 이용합니다.

1. Repository (SVN, Git)에서 source code를 가져옵니다.
2. 정해진 절차에 따라 build 및 unit test, integration test를 수행합니다. 수행 결과 이상이 있을 때 e-mail을 통해서 reporting해주는 역할까지 수행해 줍니다.
3. 정상적으로 build 되었으며, 모든 test 결과에 이상이 없는 경우 배포 파일(setup, update)을 생성합니다.

Jenkins 자체는 사용자가 입력한 command line 명령어들을 특정 조건에 맞게 또는 특정 시간이나 일정한 간격으로 수행해주는 역할을 해줍니다.
build 및 test, deploy하는 역할 자체는 command line에서 수행가능한 형태로 사용자가 직접 입력해 놓으면 그 명령어를 수행해주는 역할을 할 뿐이지, Jenkins 자체에 build tool이 있다던지 그러진 않습니다.

### 2. 이번 Posting에서 다룰 내용

1. Jenkins download 및 설치 (Windows 기준)
2. MSBuild plugin 설치
3. Jenkins item 설정
  1. SVN Repository에서 source code 내려받기
  2. VisualStudio Solution (.sln)을 command line에서 실행시키기 (MSBuild 이용)
  3. Jenkins item에서 build 후 다른 item 실행

### 3. Jenkins download 및 설치

- Link : <http://jenkins-ci.org>

<img src="https://github.com/DevStarSJ/Study/blob/master/Blog/CI/image/jenkins.01.png?raw=true">

위 Link에서 우측에 있는 `Windows`를 눌러서 다운로드 받으면 됩니다.
zip 파일이 다운로드 받아지는데 압축을 풀어서 셋업을 실행하면 설치가 끝납니다.
바로 Jenkins 창이 실행됩니다. 기본적으로는 <http://localhost:8080> 로 접속이 가능합니다. (다른 PC에서 접속시에는 PC ip나 hostname 에 8080 포트로 접속을 하면 됩니다.)

<img src="https://github.com/DevStarSJ/Study/blob/master/Blog/CI/image/jenkins.02.png?raw=true">

###3. MSBuild plugin 설치

Visual Studio에서 작업한 solution 파일 (.sln)을 command line에서 build 하기 위해서는 MSBuild.exe를 실행해서 build 해야 합니다. MSBuild.exe는 Visual Studio 설치시 같이 설치가 됩니다.
Jenkins에서 MSBuild를 사용하기 위해서는 별도의 plugin이 필요합니다.

먼저 Jenkins 화면에서 `Jenkins 관리`-> `플러그인 관리` 로 들어갑니다.

<img src="https://github.com/DevStarSJ/Study/blob/master/Blog/CI/image/jenkins.03.png?raw=true">  

internet에 연결된 환경이라면 `설치가능` 탭에서 바로 MSBuild를 선택해주시면 됩니다.

<img src="https://github.com/DevStarSJ/Study/blob/master/Blog/CI/image/jenkins.04.png?raw=true">  

offline 상태의 pc라면 <https://updates.jenkins-ci.org/download/plugins>로 들어가서 원하는 버전의 MSBuild plugin을 다운로드 받으시면 됩니다.

<img src="https://github.com/DevStarSJ/Study/blob/master/Blog/CI/image/jenkins.05.png?raw=true">  

다운로드 받은 뒤 `고급` 탭을 눌러서 `플러그인 올리기` 에서 다운로드 받은 파일을 선택하면 됩니다.

<img src="https://github.com/DevStarSJ/Study/blob/master/Blog/CI/image/jenkins.06.png?raw=true">  

그런 뒤 Jenkins를 재가동 하면 됩니다.

### 4. Jenkins 환경설정

SVN , MSBuild 설정에 대한 부분을 설명드리겠습니다.
다른 Tool 들에 대해서도 어렵지 않게 스스로 설정이 가능하거나, 검색을 하시면 자료가 많이 나올 것입니다.  

먼저 Jenkins 화면에서 `Jenkins 관리`-> `시스템 설정` 으로 들어갑니다.

아래로 쭉 내리시면 `Subversion`이란 부분이 있습니다.
현재 PC에 설치된 SVN과 같은 버전으로 설정하면 끝입니다.

<img src="https://github.com/DevStarSJ/Study/blob/master/Blog/CI/image/jenkins.08.png?raw=true">  

거기서 조금 위로 올리시면 `MSBuild` 부분이 있습니다. 거기 버튼을 누른 뒤 PC상의 `MSBuild.exe`파일의 위치를 입력해줍니다.

<img src="https://github.com/DevStarSJ/Study/blob/master/Blog/CI/image/jenkins.07.png?raw=true">  

단순히 여기에 위치만 입력해 줘도 되지만, 개인적으로는 해당 위치를 시스템환경변수에 PATH로 잡아두시는 것을 추천합니다.
그러면 cmd 창에서 직접 실행도 가능합니다.

<img src="https://github.com/DevStarSJ/Study/blob/master/Blog/CI/image/jenkins.09.png?raw=true">  

`MSBuild`의 자세한 사용법에 대해서는 여기서 다루지 않겠습니다.  

MSDN에 자세한 설명이 있으니 참조하시면 됩니다.
> <https://msdn.microsoft.com/ko-kr/library/ms164311.aspx>

그 중 Jenkins에서 자주 사용하는 예시 한가지만 든다면...

```
/p:Configuration="Release" /t:Clean,Build
```

Release 모드로 Clean 후 Build를 할 경우 위와 같이 써주시면 됩니다.

###5. Jenkins item 추가

- `새로운 item`을 누르면 추가가 가능합니다.
- 원하시는 이름을 입력한 뒤 `Freestyle project`를 선택합니다.
- 먼저 `고급 프로젝트 옵션`에서 `고급` 버튼을 눌러서 `사용자 빌드 경로 사용`을 체크하셔서 source code를 받을 경로를 입력해 줍니다.  

> ex. `d:\build\svn\test_project`  

- `소스코드관리`에는 `Subversion`을 선택하시고 `Repository URL`을 입력합니다.  

> ex. `svn://192.168.0.1/repo/test_project/trunk`  

- `Add Build Step` 을 눌러서 `MSBuild`를 선택합니다.
- build할 solution이나 project 파일명을 전체경로나 위에 정한 사용자 빌드 경로로 부터의 상태경로로 입력합니다.  

> ex. `./workspace/buildAll.sln`  

- 그리고 command line argument에 옵션을 적어줍니다.

> ex. `/p:Configuration="Release" /t:Clean,Build`  

- 한 item에서 여러가지의 build step을 가질 수 있습니다. 예를 들어서 `Release`로 빌드하고, `Debug`로도 빌드 한다던지, 다른 project나 solution 들을 build 한다든지 등요.
- build step에서 `Execute Windows batch command`를 선택하시면 일단 command line에서 선택하는 명령어를 그대로 입력하셔서 실행이 가능합니다. 관련 내용을 미리 batch file (.bat)나 python으로 생성해 놓고 실행하면 편리합니다.
  - 특정 file들을 다른 folder로 옮겨 놓고 파일이름을 다르게 복사하고 등등의 작업이 가능합니다.
- 현재 item이 끝난 후 다른 item을 자동으로 시작 시킬려면 `빌드 후 조치 추가`에서 `Build other projects`를 눌러서 다른 item명을 입력하시면 선택이 됩니다. 콤마(,)를 입력해서 여러개 project 실행이 가능합니다. Test Project를 실행한다던지, InstallShield를 이용하여 배포 파일을 생성하는 등의 작업을 하면 편리하게 활용이 가능합니다.
- 주기적으로 build를 할려면 `빌드 유발`에 `Build periodically`를 선택하셔서 `Schedule`에 입력하시면 됩니다. 참고로 매일 새벽에 돌리실려면 `@midnight`라고 입력하시면 됩니다. 오른쪽 ?를 누르면 자세한 설명이 나옵니다.
