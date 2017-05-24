---
title: ASP.NET MVC 03. Advanced Routing
date: 2016-05-08 02:00:00
categories:
- C#
- ASP.NET
tags:
- C#
- ASP.NET
---

# 고급 Routing

## 1. View에서 outgoing URL 생성 (using routing system)

View에서 왜 굳이 routing system을 사용해서 outgoing URL을 생성해야 할까요 ?
그냥 static link를 사용해서 쉽게 생성이 가능한데 말이죠.

```HTML
<a href="/Home/Index">Home</a>
```

Static link (정적 링크)의 문제점은 URL Schema가 바뀔 경우 hard-coding된 URL들을 모두 찾아서 바꿔줘야 합니다.
그러므로 Route 시스템을 이용하여 outgoing URL을 동적으로 생성하는게 더 바람직합니다.

### 1.1 동일 Controller에서의 다른 Action Method 호출

```CSharp
@Html.ActionLink("This is a outgoing URL", "ActionMethod")
```

`Html.ActionLink` helper method를 이용하여 rendering된 page의 html을 보면 아래와 같이 생성이 됩니다

```HTML
<a href="/Home/ActionMethod">This is a outgoing URL</a>
```

`RouteConfig.cs`의 `RouteConfig`에 설정된 `.MapRoute()`중 matching되는 패턴을 찾아서 해당 URL로 rendering됩니다.  

### 1.2 다른 Controller 호출

다른 Controller로 가려면 `Html.ActionLink`의 overloading된 다른 함수를 사용하면 됩니다.

```CSharp
@Html.ActionLink("This is a outgoing URL", "ActionMethod", "ControllerName")
```

### 1.3 추가값 전달

Controller, Action Method 이외에 추가값을 전달할 경우 익명타입을 이용하여 전달합니다.

```CSharp
@Html.ActionLink("This is a outgoing URL", "ActionMethod", "ControllerName", new { id = "Luna" )
```

이 경우 다음과 같은 Link로 rendering 됩니다.

```HTML
<a href="/ControllerName/ActionMethod?id=Luna">This is a outgoing URL</a>
```

물론 `routes.MapRoute(null, "{controller}/{action}/{id}", ...);`로 URL pattern이 설정되어 있을 경우에는 `"/ControllerName/ActionMethod/Luna"` 로 생성합니다.  

### 1.4 Html Attribute가 적용된 앵커(a) 생성

익명타입의 추가값 다음 인자로 익명타입으로 전달하면 됩니다. 전달할 추가값이 없는 경우에는 추가값에 대한 익명타입란에 `null`을 넣어주면 됩니다.

```CSharp
@Html.ActionLink("Home", "Index", "Home", null, 
    new { id = "anchorID", @class = "cssStyleClass" } )
```

다음과 같이 rendering 됩니다.

```HTML
<a href="/" id = "anchorID", class = "cssStyleClass">Home</a>
```

`@class`를 좀 주의해야하는데, 아마도 C# keyword의 `class`와 중복이 되어서가 아닐까라 추측됩니다.

### 1.5 정규화된 URL 생성

```CSharp
@Html.ActionLink("Home", "Index", "Home",
    "https", "mySite.com", "fragment",    
    new { id = "Luna" },
    new { id = "anchorID", @class = "cssStyleClass" } )
```

`Controller` 다음에 새로 추가된 3개의 매개변수는 각각 Protocol, Server의 URL, FragmentName을 나타냅니다.
그래서 다음과 같이 rendering 됩니다.

```HTML
<a href="https://mySite.com?id=Luna#fragment" id = "anchorID", class = "cssStyleClass">Home</a>
```

### 1.6 링크(a)없이 URL만 생성하기

`Html.ActionLink()` 대신 `Html.Action()`를 사용하면 `<a></a>`없는 URL만 Text로 생성합니다.
말 그대로 `Link`를 빼면 Link없이 URL만 생성합니다.
사용법은 `Html.ActionLink()`와 같습다만 처음의 text 인자와 html attribute 인자가 없습니다.

```CSharp
@Html.Action("ActionMethod", "ControllerName", new { id = "Luna" )
```

의 경우 `/ControllerName/ActionMethod?id=Luna`라는 문자열을 생성합니다.
`routes.MapRoute(null, "{controller}/{action}/{id}", ...);`로 URL pattern이 설정되어 있을 경우에는 `/ControllerName/ActionMethod/Luna` 로 생성합니다.  

## 2. Action Method에서 outgoing URL 생성하기

### 2.1 View에서 사용한 helper method 그대로 사용하여 URL생성

```CSharp
public ViewResult MyActionMethod()
{
    string url = Url.Action("Index", new { id = "Luna"}); // /Home/Index?id=Luna or /Home/Index/Luna 
    string url2 = Url.RouteUrl(new { controller = "Home", action = "Index"}); // /Home/Index or /
    ...
    return View();
}
```

### 2.2 다른 URL로 재전송

#### 2.2.1 다른 Action Method 호출

```CSharp
public RedirectToRouteResult MyActionMethod()
{
    return RedirectToAction("Index");
}
```

다른 Controller 호출 및 segement 전달 가능한 overload된 함수들이 존재합니다.

#### 2.2.2 URL을 생성하여 재전송

```CSharp
public RedirectToRouteResult MyActionMethod()
{
    return RedirectToRoute(new { controller = "Home", action = "Index", id = "Luna"});
}
```

## 3. 특정 Route를 선택적으로 사용하기

이제껏 Routing system이 URL, Link를 Rule에 따라서 생성하였습니다.
우리가 직접 Route를 선택하는 방법도 있습니다. 
(`routes.MapRoute()`의 첫번째 인자로 명칭을 입력했는데 그것을 명시적으로 사용하면 됩니다.)

```CSharp
routes.MapRoute("Route1", "{controller}/{action}");
routes.MapRoute("Route2", "App/{action}", new { controller = "Home" });
```

위와 같이 설정된 경우

```CSharp
@Html.ActionLink("Customer Infomation", "Index", "Customer")
```

는 항상 다음의 Link로 생성됩니다.

```HTML
<a href="/Customer/Index">Customer Information</a>
```

### 3.1 View에서 특정 Route 사용하기

만약 `Route2`를 이용한 Link의 생성을 원할 경우 다음과 같이 명시적으로 Route Name을 선택 할 수 있습니다.

```CSharp
@Html.ActionLink("Customer Infomation", "Route2", "Index", "Customer")
```

```HTML
<a href="/Home/Index?Length=5" Length="8">Customer Information</a>
```
사용자가 `Customer`라는 Controller를 사용하라고 했지만, 해당 Route Pattern에 의해서 `HomeController`로 연결이 됩니다.

### 3.2 Route Attribute에서 특정 Route를 지정

```CSharp
[Route("Add/{user}/{id:int}", Name="AddRoute")]
public string Add(string user, int id)
{
    return $"Add User - Name : {user} , ID : {id}";
}
```

## 4. Area

Project에서 Area를 추가 할 수 있습니다.
각 영역(Area)별로 다른 Controller, Model, View 및 다른 Route 규칙 등을 가집니다.
Area의 특징은 다음과 같습니다.

- Area별로 다른 Folder안에 MVC Project의 구조들을 그대로 가지고 있습니다.
- Area내의 `(Area명칭)AreaRegistration.cs` 내에 Route 규칙을 정의합니다.
- Project의 `Global.asax.cs`내에 `AreaRegistration.RegisterAllAreas();`를 호출하는 부분이 추가됩니다.
- Attribute로 Routing하는 경우 `[RouteArea("Area명칭")]`으로 적용이 가능합니다.
- `Html.ActionLink()`에서 Area를 명시할 경우에는 추가정보를 적는 무명 타입란에 `area`를 지정하면 됩니다.

```CSharp
@Html.ActionLink("Go to Another Area", "Index", new { area = "Admin" })
```

## 5. Disk file에 대한 요청

정적 파일 (Image, html, JavaScript 등...)에 대한 접근은 그냥 해당 주소를 그대로 적어주면 접근이 됩니다.
Routing system은 전달받은 URL이 disk상의 file과 매칭될 경우 Routing pattern과는 상관없이 해당 file을 전달해 줍니다.
단, `RouteConfig.cs`에서 `routes.RouteExistingFiles = true;`로 설정할 경우 file을 찾기전에 Routing pattern을 먼저 적용합니다.
이렇게 설정한 경우 특정 패턴에 대해서는 file로 접근을 먼저 시키고자 할때는 `routes.IgnoreRoute("Content/{filename}.html")`과 같은 형식으로 적용이 가능합니다.

