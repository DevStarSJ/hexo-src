---
title: .NET Core Install for Ubuntu 14.04
date: 2016-09-17 00:00:00
categories:
- C#
- ASP.NET Core
tags:
- C#
- ASP.NET Core
---

# .NET Core Install for Ubuntu 14.04

`.NET Core`를 **Ubuntu**에 설치하는 과정에 대해서 소개해드리겠습니다.

**Ubuntu** 설치는 필자의 경우는 *Microsoft Azure*에 설치하였습니다.
(참고로 **Azure**에 **Ubuntu**설치시 SSH (22)번 빼고는 모두 막혀있습니다. Portal에서 원하시는 포트를 열어야 합니다.)

## 1. **.NET Core** 설치 후 *Hello World* 출력해보기

먼저 `.Net Core`를 컴파일하고 실행할 수 있도록 **SDK**를 설치하겠습니다.

공식문서에 설명이 잘 되어 있습니다. (<https://www.microsoft.com/net/core#ubuntu>)
아래 설명대로 해서 잘 안되면 Link의 공식문서에 바뀐 점이 있는지 보시고 따라해주세요.

아래 명령어를 하나씩 입력해주세요.

```
sudo sh -c 'echo "deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/dotnet/ trusty main" > /etc/apt/sources.list.d/dotnetdev.list'

sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893

sudo apt-get update

sudo apt-get install dotnet-dev-1.0.0-preview2-003121
```

이제 `.Net Core SDK` 설치가 끝났습니다.
제대로 설치가 되었는지 기본 예제를 실행해 보겠습니다.

```
mkdir hwapp

cd hwapp

dotnet new

dotnet restore

dotnet run
```

아래와 같이 출력이 나오면 제대로 설치가 된 것입니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/ASP.NET/ASP.NET.Core/image/install.ubuntu.01.png?raw=true">  

## 2. **ASP.NET Core** 용 실행환경 구성

먼저 **Web App** 생성에 필요한 것들을 설치해야 합니다.

자세한 설명은 다음 Link에 되어 있습니다. (<https://docs.asp.net/en/1.0.0-rc1/getting-started/installing-on-linux.html#installing-on-ubuntu-14-04>)

### 2.1 .NET Version Manager (DNVM) 설치

Linux 상에서 여러 버전의 .NET 실행 환경 (.NET Execution Environment) `DNX`를 관리해주는 도구입니다.

```
sudo apt-get install unzip curl

curl -sSL https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.sh | DNX_BRANCH=dev sh && source ~/.dnx/dnvm/dnvm.sh
```

### 2.2 .NET Execution Environment (DNX) 설치

Linux 상에서 .NET 프로젝트를 빌드하고 실행해주는 도구입니다.

```
sudo apt-get install libunwind8 gettext libssl-dev libcurl4-openssl-dev zlib1g libicu-dev uuid-dev

dnvm upgrade -r coreclr
```

### 2.3 libuv 설치

`libuv`는 멀티플랫폼 비동기 IO 라이브러리 입니다.
`Kestrel`에서 `libuv`를 사용합니다.
`Kestrel`은 ASP.NET Core를 호스팅하기 위한 크로스-플랫폼 HTTP 서버 입니다.

```
sudo apt-get install make automake libtool curl

curl -sSL https://github.com/libuv/libuv/archive/v1.8.0.tar.gz | sudo tar zxfv - -C /usr/local/src

cd /usr/local/src/libuv-1.8.0

sudo sh autogen.sh

sudo ./configure

sudo make

sudo make install

sudo rm -rf /usr/local/src/libuv-1.8.0 && cd ~/

sudo ldconfig
```

## 3. **ASP.NET Core** Web App 생성하기

다음 Link의 공식문서를 보고 작성하였습니다. (<https://docs.asp.net/en/latest/getting-started.html>)

좀 전에 생성한 예제 코드로 이동하겠습니다.

```
cd ~/hwapp
```

`project.json`의 `dependencies`란에 다음과 같이 `Kestrel`를 추가해주세요.

```JSON
{
  "version": "1.0.0-*",
  "buildOptions": {
    "debugType": "portable",
    "emitEntryPoint": true
  },
  "dependencies": {},
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
  }
}
```

패키지를 project에 다운로드 합니다.

```
dotnet restore
```

`Startup.cs`파일을 추가하려 다음의 내용으로 작성합니다.

```C#
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;

namespace aspnetcoreapp
{
    public class Startup
    {
        public void Configure(IApplicationBuilder app)
        {
            app.Run(context =>
            {
                return context.Response.WriteAsync("Hello from ASP.NET Core!");
            });
        }
    }
}
```

`Program.cs`파일을 다음과 같이 수정해주세요.

```C#
using System;
using Microsoft.AspNetCore.Hosting;

namespace aspnetcoreapp
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var host = new WebHostBuilder()
                .UseKestrel()
                .UseStartup<Startup>()
                .Build();

            host.Run();
        }
    }
}
```

그런 다음 실행합니다.

```
dotnet run
```

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/ASP.NET/ASP.NET.Core/image/install.ubuntu.02.png?raw=true">

위 그림과 같은 메세지가 나오면 성공한 것입니다.

웹브라우저로 붙어보면 아래와 같은 그림이 나옵니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/ASP.NET/ASP.NET.Core/image/install.ubuntu.03.png?raw=true">

## 4. 외부에서 접속가능하게 배포하기

다음 Link의 공식문서를 보고 따라했습니다.
(<https://docs.asp.net/en/latest/publishing/linuxproduction.html>)

`Nginx`를 설치합니다.

```
sudo apt-get install nginx
```

이제 우리가 띄운 `Web App`으로 접속하도록 **Proxy**를 설정합니다.

설정파일은 `/etc/nginx/sites-available/default` 입니다.

```
sudo vi /etc/nginx/sites-available/default
```

버전별로 내용이 조금 다룰수 있는데 눈여겨 볼 부분은 다음과 같습니다.

### 4.1 외부에서 접속할 Port 설정

아래 `80`부분을 원하는 Port로 설정하면 됩니다.
```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
```

### 4.2 내부로 연결한 주소 설정

`location`부분을 아래와 같이 설정합니다.

```
location / {
    proxy_pass http://localhost:5000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection keep-alive;
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
}
```

중요한 부분만 그림으로 보면 다음과 같습니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/ASP.NET/ASP.NET.Core/image/install.ubuntu.04.png?raw=true">

수정한 내용에 이상은 없는지 확인한 후 실행합니다.

```
sudo nginx -t

sudo service nginx start
```

만약 수행 중 변경 후 반영하려면 다음과 같이 입력해야 합니다.

```
sudo nginx -s reload
```

이제 외부에서 Web App으로 접근이 가능합니다

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/ASP.NET/ASP.NET.Core/image/install.ubuntu.05.png?raw=true">
