---
title: 자바 네트워크 소녀 Netty
date: 2015-10-29 00:00:00
categories:
- Book
- Review
tags:
- Book
- Review
---

### 자바 네트워크 소녀 Netty
- 한빛미디어
- 정경석 지음
- 책소개 Link : <http://www.hanbit.co.kr/book/look.html?isbn=978-89-6848-224-3>

 ![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.02.jpg?raw=true)  

#### 첫 인상

처음 책을 받고 깜짝 놀랐습니다.  
내가 분명 주문한 것은 IT 기술서적인데...  
미소녀의 대형 브로마이드가 배송되었으며, 덤으로 책이 한권 같이 왔더군요.  

![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.01.jpg?raw=true)  

두께가 많이 두껍지 않아서 부담없이 들고 다니기 좋아 보입니다. 하지만 표지 그림때문에 약간 들고다니기는 망설여 지더군요.  
종이재질은 `한빛미디어`다 보니 당연히 최상이구요.  
책안의 그림, 소스코드, 스크린샷 모두 눈에 쏙쏙 들어오게 되어 있었습니다.  

- 그림  
  ![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.03.jpg?raw=true)  

- 스크린샷  
  ![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.04.jpg?raw=true)  

- Source Code  
  ![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.05.jpg?raw=true)  

#### 전체적인 서평

수많은 개발자들이 사용하는 언어,`Java`. `Netty`는 범용 자바 네트워크 프레임 워크입니다. `Facebook`에서 출시 된 것을 보고 어떤 내용일까 궁금했는데, `한빛리더스 2기` 활동 중인데 이번 달의 목록에 이 책이 있었습니다. 그것도 선착순 5명만 가능하다고 해서 얼릉 신청했습니다.  
```
Network Server 개발을 하기 위해서는 Socket을 이용한 Chatting Server만 만들어보면 됩니다.
Web Server 개발을 하기 위해서는 게시판만 한번 만들어보면 됩니다.
```
와 같은 말이 있습니다. 마침 Chatting Server 쪽에 대해서 괌심이 있었습니다. Netty의 내부를 이해하기 어려운 점이 있긴 하지만 처음에 추천의 말에서 `네티를 처음 접하신 분은 4-5장을 꼭 정독하라`는 글귀를 읽고 우선 4-5장부터 차례대로 보기 시작했습니다.  

이 책을 읽어야 하는 독자들은 학생들 보다는 Java 현업에서 네트워크 통신 개발이 경험이 있는 프로그래머나 네트워크에 대한 기반과 Java 기본 개념에서 중급으로 넘어갈 수 있는 학생들이라면 추천합니다. 무작정 Java 초급이었다가 Chatting Server를 만들려고 이 책을 접했더라면 엄청 험난한 길에 접어들 것이라 예상됩니다. 기본적인 Linux 사용법과 전반적인 Server에 대한 지식, Java 개발 경험이 있는 개발자들에게 도움이 된다고 할 수 있겠습니다. 저는 Windows 환경의 개발자이고 Java도 공부를 한적은 있지만, Project 경험은 없어서 보는데 애를 좀 먹었습니다. 예전에 TCP를 이용한 Chatting Server / Client는 교육과정을 통해서 직접 만들어 본적은 있었습니다.  

책을 전체적으로 볼 때 각 단원마다 `마치며` 라는 부분이 너무 좋았습니다. 이해 되지 않았던 단어들 및 그리고 현재에 이용해야 하는 개발자들에게 참고 사항같이 독자들의 배려로 보이는 것으로 집필자의 작성 글들이 섬세하다는 것을 많이 느꼈습니다.  

####1장 네티 맛보기  

Network 관련된 기초 설명이 별도로 없기 때문에 Java 네트워크 프로그램에 대한 기본 지식이 있는 상태에서 접근해야 책을 보는데 도움이 될것입니다. 바로 다운받고 설치하는 과정으로 들어가고 특히 환경 설정에서 그림을 보여 주며 설명 해주는 부분은 기존 책들과 비슷합니다. 다만 책에서는 Windows 7을 기준으로 설명하고 있어서, Windows 8.1이나 Windows 10을 사용하는 독자들은 Putty 설치방법 등에 대해서 따로 찾아봐야 합니다. 각각의 필요한 Server에 대해 이해 할 수 있도록 우선 코드를 작성하면서 개념에 대해 설명해주는 것도 좋았습니다.  
Source Code에 번호를 붙여 별도로 설명을 해줌으로써 Code를 작성하고 참고할 개발자들에게는 주석을 달아 놓도록 간단하게 설명해주기때문에 그 부분도 좋았습니다.

![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.06.jpg?raw=true) 

#### 2장 네티 주요 특징

Netty의 주요 특징에서는 동기와 비동기에 대해 자세히 설명 해주지만 프로그램적 보다는 사전에 있는 내용을 인용했기 때문에 독자들의 호불호가 갈릴것 같습니다. 그리고 다른 책과는 달리 한 장 한 장씩 동기와 비동기를 그림으로 보여 주고 각각 설명 해주는 부분이 읽기에 부담 없고 이해하는데도 쉽게 도움이 되어 좋았습니다.  

블로킹과 논블로킹 소켓에 대한 부분은 직접 Code를 입력해 보면서 숫자를 매겨 설명을 별도로 해주는 방식이라 Source Code에 대한  분석도 좋았습니다. 그림을 통해 다양한 종류로 보여 주기 때문에 이해되지 않는 부분에 대해서도 쉽게 이해할 수 있도록 노력한 점도 좋았습니다. 글만 읽게 되는 것과 글과 그림을 동시에 보여 주는 이해도는 엄청나게 차이가 난다는 것을 다시 한번 느꼈습니다.  

![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.07.jpg?raw=true) 

#### 3장 부트스트랩 
부트스트랩은 개발자, 디자이너, 퍼블리셔라면 접해야 되는 부분인데 부트스랩의 기본 정의부터 API, 다양한 이벤트 핸들러 관련 부분과 설정까지 다양하게 정리되어 있고 우선 구조에 대해서도 그림으로 보여줌으로써 한눈에 특징을 볼 수 있습니다. Java 개발자들에게 볼 수 있는 다이어그램 및 패턴을 code로 작성해 보면서 code에 붙여 진 주요한 method의 특징을 기호를 찾아 설명을 읽으면 더 자세하면서도 바로 실무에 적용할 수 있는 code가 있는 점이 너무 좋았습니다.

![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.08.jpg?raw=true) 

#### 4장 채널 파이프라인과 코덱

Event 실행을 할 경우에 실행 과정을 보여 줄 때 기호를 표기 해 준 것이 좋았습니다. 보통 책을 읽을 때 순서를 매기지 않고 설명 식으로 진행 해주는 책이 많은데 이 책은 이해하기 쉬웠습니다. 채널 파이프 라인 예시는 관련 개념을 모르는 사람이 보기에도 적절 했습니다. 전력 공급 과정으로 채널과 채널 파이프 라인의 관계를 표현 해주는 쉽게 이해가 되었고 그림 자체도 도움이 많이 되었습니다. 각각의 구성과 설명을 그림으로 표현하고 번호를 붙여 설명해 주는 것이 특히 채널 생성과 채널 파이프라인의 구성에 대해 1장에서 작성했던 code와 주석과 설명 그리고 그림까지 한꺼번에 번호를 매겨 보여 주는 것은 독자들로 하여금 작성하면서 이해 안되어 있던 부분까지 상세하게 보여 주는 배려가 보여 너무 좋았습니다. 이벤트 핸들러의 각각의 이벤트마다 간단한 설명만 있어서 독자들이 각각 해봐야만 이해할 수 있는 부분은 조금 아쉬웠습니다.  

![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.09.jpg?raw=true) 

#### 5장 이벤트 모델  

 웹 개발자라면 눈 여겨 봐도 좋은 chapter입니다. `Node.js` 와 `ver.x`(Netty)의 차이점을 이용해서 Netty 이벤트 모델에 대해 설명 해주기 때문에 기존에 웹 프로그램 경험이 있는 개발자들이라면 바로 개념을 이해하고 사용할 수 있도록 그림 설명과 처리량에 대해서는 그래프로 작성해 주어 바쁜 상황에 글을 읽지 못하더라도 한눈에 표로 확인할 수 있도록 배려한 것이 좋았습니다. 그리고 code가 예시로 나오기 때문에 직접 독자들이 작성해 보면서 이해할 수 있도록 해주는 것이 다른 기존 책들보다 상세하게 표현되어 있어 이해하는데 훨씬 도움이 되었습니다.
 
![Netty](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.10.jpg?raw=true) 

####6장 바이트 버퍼  

이 부분은 Java 개발자이면서 전공자면 조금 쉽게 접근할 수 있을듯 합니다. 예전에 `NIO`에 대해서 교육받은 적이 있어서 쉽게 이해할 수 있었습니다. 자료 구조와 관련된 부분이 많고 source code로 설명을 해주기는 하나 이 내용을 이해할 수 있는 것은 학생들 보다는 현업에 있는 개발자들 특히 버퍼를 많이 이용하는 개발자에게도 내용이 조금은 어려울 것 같습니다. 버퍼를 많이 이용한 개발자들에게는 예제를 통해서 Netty와의 차이점을 확연히 알아볼 수 있을 것 같습니다.

![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.11.jpg?raw=true) 

#### 7장 네티와 채널 보안  

7장부터 끝까지는 `3부 Netty응용부분`이라 기존에 Netty를 사용하는 개발자들에게는 필요한 부분과 이용할 때 참고 사항이 많아지는 부분입니다. 특히 Netty는 범용 네트워크 프레임워크기 때문에 보안에 대해 관심이 많고 그 보안을 어떻게 해야 할지 모르는 개발자들에게는 한줄기 빛과 같은 단원일듯 판단됩니다.  

그리고 네트워크 보안에 대해서도 짧게나마 설명해주는 부분이 있기 때문에 개념에 대해 잘 모르고 이용이 어려울 경우에는 책에 있는 자료를 참고하는것도 좋은 방법일것 같습니다. Netty server에 SSL을 이용하는 내용을 추가해 주는데 Windows와 Lunux에 대한 설명이 같이 있고, OpenSSL을 이용하는 방법에 압축이나 명령 같은 부분을 직접 이용한 명령어로 보여줌으로써 쉽게 따라 할 수 있도록 배려해 준 것이 좋았습니다. 그리고 채널 보안 적용하기에 서버 부트스트랩 설정 코드와 번호를 붙여 간단하게 주석을 작성할 수 있도록 해주는 배려도 좋았습니다. 실제 네트워크 데이터를 캡쳐하여 유용하게 사용하는 도구와 방법을 언급해주므로서 필요할 경우 적용 할 수 있는 점이 좋았습니다.

![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.12.jpg?raw=true) 

#### 8장 네티와 서드 파티 연동  

`Spring framework`에 대한 기본 지식이 있어야 `Maven`에 접근이 가능합니다. 즉 Java만 배운 학생에게는 어려운 부분이 많아 보입니다. 하지만 `Eclipse`를 이용할 수 있는 사람이라면 실습은 쉽게 따라 할 수 있도록 잘 설명되어 있습니다. Eclipse에서  market이용하여 Maven 설치와 실습까지 동시에 진행이 되나 Linux에서 실행되는 것이기 때문에 `CentOS`를 잘 아는 학생이나  개발자들에게는 쉽게 이해 할수 있을듯 합니다. 그렇지 않을 경우 Linux와 Spring 기본 개념에 대해 공부를 하고 source code를 봐야지만 잘 알아 볼 수 있을 것입니다. 기본을 잘 알고 Spring까지 애플리케이션 작성 source code까지 있기 때문에 그대로 따라 해보고 실행해 보거나 응용할 수 있도록 해 주었습니다.

![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.13.jpg?raw=true)  

#### 9장 네티로 구현한 API 서버 
마지막은 Netty로 아예 처음부터 설계와 작업을 해서 실습이 되고 API 통합 테스트까지 해주므로서 Netty를 한번 더 과정을 볼 수 있게 해주는 과정이라 좋았습니다.  

![](https://github.com/DevStarSJ/Study/blob/master/Blog/Review/Books/image/small.hanbit.netty.14.jpg?raw=true)


