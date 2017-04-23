---
title: Deploying .NET Core Web App with Visual Studio and Git
date: 2016-05-11 00:00:00
categories:
- C#
- ASP.NET Core
tags:
- C#
- ASP.NET Core
---

# Deploying .NET Core Web App with Visual Studio and Git

** ASP.NET Core** 개발환경으로는 Windows에서 Visual Studio를 사용하는 것이 가장 편합니다.
이번 포스팅에서는 Windows에서 `ASP.NET Core Web App`을 작성하여 Git을 이용하여 **Ubuntu**에서 실행하는 것을 소개해 드리겠습니다.

## 1. Github에 Repository 생성하기

본인의 GitHub 계정으로 가셔서 아래와 같이 원하는 이름으로 Repository를 생성합니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/ASP.NET/ASP.NET.Core/image/vs.git.01.png?raw=true">

## 2. PC에 GitHub Repository Clone 하기

원하시는 폴더로 가셔서 GitHub Repository에 연결하세요.
(아래에서 github 주소란에는 본인의 Repository 주소로 적어주세요.)

```
mkdir WebApp

cd WebApp

git init

git remote add origin https://github.com/DevStarSJ/WebApp.git

git pull origin master
```

## 3. Visual Studio로 Web App 예제 만들기

`ASP.NET Core Web Application`으로 프로젝트를 생성합니다.
위치를 좀 전에 Git Repository로 지정한 곳으로 해주세요.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/ASP.NET/ASP.NET.Core/image/vs.git.02.png?raw=true">

다음 그림에서는 `Web Application`을 선택한 후에 `OK`를 눌러주세요.

제대로 만들어졌는지, `F5`를 눌러서 확인해봅니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/ASP.NET/ASP.NET.Core/image/vs.git.03.png?raw=true">

## 4. GitHub에 올리기

```
git add --all

git commit -m "Initial Commit"

git push origin master
```

## 5. Ubuntu에서 내려받기

아직 `git`이 설치되어 있지 않다면 아래와 같이 설치를 해주세요.

```
sudo apt-get install git
```

이제 원하는 폴더로 가셔서 GitHub에서 내려받습니다. 위에 Windows에서 한것과 명령어가 같습니다.

```
mkdir WebApp

cd WebApp

git init

git remote add origin https://github.com/DevStarSJ/WebApp.git

git pull origin master
```

이제 WebApp 폴더로 들어가서 `project.json`의 `framework`부분을 아래와 같이 수정합니다.

```
cd WebApp/src/WebApp
```


```JSON
  "frameworks": {
    "netcoreapp1.0": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "type": "platform",
          "version": "1.0.0"
        },
        "Microsoft.AspNetCore.Server.Kestrel": "1.0.0"
      },
      "imports": "dnxcore50"
    }
  },
```

저장한 뒤 이제 실행합니다.

```
dotnet restore

dotnet build

dotnet run
```

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/ASP.NET/ASP.NET.Core/image/vs.git.04.png?raw=true">


