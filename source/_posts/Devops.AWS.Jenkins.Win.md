---
title: ASP.NET Web Project를 EC2 or Elastic BeanStalk에 배포하는 Jenkins 시스템 구축
date: 2017-08-01 17:00:00
categories:
- DevOps
tags:
- DevOps
- CI/CD
- Jenkins
- AWS
---

# ASP.NET Web Project를 EC2 or Elastic BeanStalk에 배포하는 Jenkins 시스템 구축

## 1. EC2 생성
- ec2 생성
  - **Visual Studio**를 설치할려면 최소 2.5GB이상의 메모리 (t2.medium)
  - **MSBuild**만 설치할 것이라면 훨씬 작은 것도 가능. 1G 메모리를 가진 (t2.micro)
  - Security Group 에 8080 포트에 접속 허용 ip 대역 설정

## 2. 필요한 설치 파일들

- Copy files to ec2
  - Jenkins : <https://jenkins.io/download>
  - MSBuild Tools
    - MSBuild Tools 2015 기준 : <https://www.microsoft.com/ko-kr/download/details.aspx?id=48159>
    - AWS CLI : <http://docs.aws.amazon.com/cli/latest/userguide/awscli-install-windows.html#install-msi-on-windows>
    - AWS Tools and SDK : <https://aws.amazon.com/ko/visualstudio>
    - Git : <https://git-scm.com/download/win>
    - Microsoft Web Deploy v3.6 <https://www.microsoft.com/en-us/download/details.aspx?id=43717>
    - 만약 TypeScript를 썼다면 ? <https://www.microsoft.com/en-us/download/confirmation.aspx?id=48593>
- 설치 : 전부다

- Visual Studio를 설치할 경우 Web 설치가 아닌 offline 설치용 ISO를 구해야함

## 3. MSBuild 실행 확인

- MSBuild.exe 가 실행될수 있도록 환경변수에 PATH 등록
![](/images/msbuild.03.png)

- 이제 빌드가 정상적으로 되는지 확인

- 혹시 안되면 Visual Studio 가 설치된 PC의 `MSbuild` 폴더 내부를 전부 다 복사
- 대부분 문제가 되는건 `MSBuild` 자체보다는 `Web Deploy` 설치 및 버전이 문제가 됨

- 참고그림 : Web Deploy 설치전 ec2와 Visual Studio가 설치된 개발PC의 폴더 내용들

![](/images/msbuild.01.png)
![](/images/msbuild.02.png)

## 4. AWS Deploy 확인

이제 빌드가 정상적으로 되는건 확인 단, Deploy가 안됨

`aws configure --profile 프로파일명` 식으로 AWS 프로파일 등록

디플로이까지 성공 확인

## 5. Jenkins 설치 후 item 등록

이하 생략...
