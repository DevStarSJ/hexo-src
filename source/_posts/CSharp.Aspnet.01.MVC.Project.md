---
title: ASP.NET MVC 01. Project
date: 2016-05-07 00:00:00
categories:
- C#
- ASP.NET
tags:
- C#
- ASP.NET
---

## MVC Project 항목요약

- `/App_Data` : File기반 저장소의 데이터 파일,XML 파일 등을 저장
- `/App_Start` : Route, Filter, Contents 등에 대한 정의 같은 Project에 대한 주요 구성 설정들
  - `/RouteConfig.cs` : Route 정보 
- `/Areas` : 큰 Application을 작은 Area로 나누는 방법
- `/bin` : compile된 어셈블리 및 기타 어셈블리들이 위치, 소스 제어에 포함시켜서는 안됨
- `/Content` : css, Image 등 static contents들
- `/Controllers` : Controller classes
- `/Models` : VewModel, Domain Model 들. 별도 Project에 Domain Model을 정의하는 방법을 더 많이 선호
- `/Scripts` : JavaScript Library
- `/Views` : View, PartialView
  - `/Web.config` : View가 동작하는데 필요한 구성, ISS에 의해 View 파일 자체가 서비스되지 않도록 제한하는데 필요한 구성, View에 import하는 namespace에 대한 정보 등...
  - `/Share` : 특정 Controller에 종속되지 않는 Layout, View
- `/Global.asax` : 전역 ASP.NET Application class.
- `/Web.config` : Application을 위한 구성 파일

## MVC 규약

- Controller
  - `/Controllers/명칭Controller.cs` 란 이름으로 저장
  - HTML helper method에서 `명칭`만 입력하면 `명칭`Controller class를 자동으로 찾아줌

- View
  - `/Views/명칭` 폴더안에 저장
  - 해당 Controller의 Action method 명칭으로 View이름을 정함 : `return View();`
    - ex) ProductController의 Index에 대한 View는 /View/Product/Index.cshtml 이란 이름으로 작성
  - 다른 View 이름을 명시적으로 사용도 가능함 : `return View("View명칭");`
  - Controller 명칭의 폴더에서 View를 먼저 찾은 후 `/Views/Shared` 폴더에서 찾는다.

- Layout
  - `_`(언더바)로 시작
  - 기본적으로 `/Views/_ViewStart.cshtml`을 모든 View에 적용
  - 특정 Layout을 지정하면 _ViewStart.cshtml을 적용하지 않음 : `Layout = "~/Views/Shared/_MyLayout.cshtml";`
  - 아무런 Layout도 참조하지 않으려면 `null`값을 대입 : `Layout = null;`
