---
title: ASP.NET MVC 04. Controller and Action
date: 2016-05-12 03:00:00
categories:
- C#
- ASP.NET
tags:
- C#
- ASP.NET
---

# Controller and Action

## 1. Controlleer 와 Action이란 ?

### 1.1 Controller

MVC Framework에서 가장 핵심이되는 역할을 수행하는 Component입니다.  

- Model을 조작
- User의 요청을 처리
- UI에 출력할 View를 결정

MVC Framework에서 사용자, View, Model은 서로 아무런 연결고리가 없이 이루어집니다.
Controller가 그 중심에서 처리를 수행합니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/ASP.NET/MVC/image/Router-MVC-DB_svg.png?raw=true">
<https://ko.wikipedia.org/wiki/모델-뷰-컨트롤러>

### 1.2 Action

한마디로 설명드리자면 Controller가 수행하는 각각의 method들을 Action이라고 부릅니다.
하나의 Controller는 여러가지 일을 수행 할 수 있습니다.
Action에서는 사용자가 요청하는 작업을 정의합니다.
기본적으로 Action의 이름과 그 결과를 출력하는 View의 이름은 같습니다.

```CSharp
public class HomeController : Controller
{
    public ActionResult Index()
    {
        return View();
    }

    public ActionResult Hello()
    {
        return View();
    }
}
```

위와 같이 정의된 Home Controller가 있는 경우,

- `http://mySite.com/Home/Index` 라는 요청을 하면 `Index()`라는 Action method가 실행되어 `/Views/Home/Index.cshtml`을 출력하며,
- `http://mySite.com/Home/Hello` 라는 요청을 하면 `Hello()`라는 Action method가 실행되어 `/Views/Home/Hello.cshtml`을 출력합니다.

## 2. Controller 구현 방법

### 2.1 IController interface로 구현

지금은 잘 사용하지 않는 방법입니다.

```CSharp
//System.Web.Mvc.IController Interface
public interface IController
{
    void Execute(RequestContext requestContext);
}
```

`Execute()`라는 하나의 구현을 제공해주기 때문에 해당 method 안에서 `RequestContext`를 분석하여 실행합니다.

```CSharp
public BasicController : IController
{
    public void Execute (RequestContext requestContext)
    {
        string controller = (string)requestContext.RouteData.Values["controller"];
        string action = (string)requestContext.RouteData.Values["action"];

        switch (action)
        {
            case "Index":
                ...
                break;
            case "Hello":
                ...
                break;
        }
    }
}
```

### 2.2 Controller를 상속받아 구현

많이 사용하는 방법입니다.
Controller는 3가지 핵심 기능을 제공합니다.

- Action Method : 작업들을 여러 method로 나눌 수 있으며, 서로 다른 URL로 노출됩니다.
- Action Result : Action method의 결과를 사용자에게 return 합니다.
  - Rendering한 View
  - JSON, XML 등의 document
  - 다른 URL로 Indirect
- Filter : Reusable한 기능들을 Filter로 Capsulation할 수 있습니다.

예제 Code는 `1.2 Action`의 예제를 참고하시면 됩니다.

## 3. 요청 데이터 받기

Action method에서 요청 데이터를 가져오는 방법들에 대해서 알아보도록 하겠습니다.

### 3.1 Context를 통해서 데이터 가져오기

ASP.NET Platform에서 제공해주는 Context 개체들이 많이 있습니다.
그 중 자주 사용되는 값들에 대해서 몇가지 소개 드리겠습니다.

- `Request.`
  - `QueryString`(NameValueCollection) : Request와 함께 전송된 GET 변수들
  - `Form` (NameValueCollection) : Request와 함께 전송되어 오는 POST 변수들
     - e.g. `Request.Form["newName"]` (string)
  - `Cookies` (HttpCookieCollection) : Browser가 Request와 함께 전달한 Cookie
  - `HttpMethod` (string) : Request에 사용된 HTTP method (GET, POST, DELETE ...)
  - `Headers` (NameValueCollection) : Request와 함께 전송된 HTTP Header
  - `Url` (Uri) : Request URL
  - `UserHostAddress` (string) : Request한 User의 IP Address
- `RouteData.`
  - `Route` (RouteBase) : Request에 대해 선택된 RouteTable.Routes Entry
  - `Values` (RouteValueDictionary) : Route parameters (URL로부터 추출된 값이나 기본값)
- `HttpContext.`
  - `Application` (HttpApplicationStateBase) : Applcation 상태 Repository
  - `Cache` (Cache) : Application Cache Repository
  - `Items` (IDictionary) : 현재 Request에 대한 상태 Repository
  - `Session` (HttpSessionStateBase) : 방문자 Session에 대한 상태 Repository
  - `Timestamp` (DateTime) : Request한 시간
- `User` (IPrincipal) : Login된 사용자의 인증정보
  - `.Identity.Name` (string) : 사용자 명칭
- `TempData` (TempDataDictionary) : 현재 사용자에 대한 임시 데이터 항목들
- `Server.MachineName` (string) : Server 명칭

자세한 정보는 MSDN에서 `System.Web.Mvc.Controller` , `System.web.Mvc.ControllerContext`를 살펴보시면 됩니다.

### 3.2 매개변수 사용하기

위에서 살펴본 것처럼 Context를 통해서 값을 읽어 올 수 있지만, Action Method에서 매개변수를 선언할 경우 해당 매개변수로 값이 전달되어서 편리하게 사용이 가능합니다.
`Request.Form`, `Request.QueryString`, `Request.Files`, `RouteData.Values`에서 매개변수와 타입, 명칭을 비교해서 매칭이 되는 경우 매개변수에 값으로 전달을 해 줍니다.
좀 더 자세히 말씀드리자면 `Value Provider`가 위 Context들에서 값을 가져오면 `Model Binder`가 해당 Value들을 Action Method에서 요구하는 형식으로 제공합니다.

단 Action Method에서는 Reference Parameter(`ref`)는 제공하지 않으며, Value Type (`premitive type`, `struct`)에 default value가 설정되지 않은 경우 해당 값이 오지 않으면 Exception이 발생합니다.
`nullable` (`int?`, `DateTime?` ...)로 선언을 하면 해당 값이 없더라도 `null`로 해당 Action Method가 수행됩니다.
Reference Type (`class`)에 대해서는 default value가 없더라도 값이 없는 경우 `null`로 전달되므로 Exception이 발생하지 않습니다.

## 4. 출력 (`return`)

Action Method의 처리 결과를 사용자에게 전달(`return`)해줘야 합니다.

### 4.1 직접 구현하기

지금은 잘 사용하지 않는 방법이지만 `IController`를 구현했을 경우에는 직접 구현을 해줘야만 했습니다.

```CSharp
public class BaseController : IController
{
    public void Execute(RequestContext requestContext)
    {
        string action = (string)requestContext.RouteData.Value["action"];
        if (action.ToLower() == "redirect")
        {
            requestContext.HttpContext.Response.Redirect("Derived/Index");
        }
        else
        {
            requestContext.HttpContext.Response.Write($"Action : {action}");
        }
    }
}
```

물론 `Controller`를 상속 받았을 경우에도 직접 구현은 가능합니다.

```CSharp
public class HomeController : Controller
{
    public void Index()
    {
        Reponse.Write("Hello from the Index Action Method");
    }
}
```

### 4.2 ActionResult로 `return`

#### 4.2.1 ActionResult 란 ?

`Response` 개체를 직접 다루는게 아니라, MVC Framework에게 우리를 대신해서 만들어내도록 전달하게 해주는 역할을 합니다.
MVC Framework는 전달받은 `ActionResult`에 해당하는 `ExecuteResult()` 를 호출하여 출력을 생성합니다.

#### 4.2.2 ActionResult 타입

- `ViewResult` (`View()`) : View template을 rendering
- `PartialViewResult` (`PartialView()`) : Partial view template을 rendering
- `RedirectToRouteResult` : Route System를 통해 URL을 생성하거나 Action Method나 Route Entry로 `HTTP 301 or 302` redirect를 수행
  - (`RedirectToAction()`, `RedirectToActionPermanent()`, `RedirectToRoute()`, `RedirectToRoutePermanent()`)
- `RedirectResult` (`Redirect()`) : 특정 URL로 `HTTP 301 or 302` redirect 수행
- `ContentResult` (`RedirectPermanent()`, `Content()`) : Text 타입의 데이터를 Browser로 return. content-text header 설정 (optional)
- `FileResult` (`File()`) : Binary File을 Browser로 직접 전송
- `JsonResult` (`Json()`) : .NET object를 JSON으로 Serialization하여 전송. Web API에서 보편적으로 사용
- `JavaScriptResult` (`JavaScript()`) : JavaScript source를 Browser로 전송
- `HttpUnauthorizedResult` : `HTTP 401` (인증되지 않음)으로 처리
- `HttpNotFoundResult` (`HttpNotFound()`) : `HTTP 404` (찾을 수 없음) 으로 처리
- `HttpStatusCodeResult` : 특정 HTTP code로 처리
- `EmptyResult` : 아무것도 안함

#### 4.2.3 Returning HTML for View Rendering

ActionResult 중 가장 기본인 `ViewResult`에 대한 간단한 사용법 입니다.

```CSharp
public ViewResult Index()
{
    return View("Homepage"); // Route System에게 Homepage라는 segment에 해당하는 View를 rendering하도록 전달

    return View("Index", "_AlternateLayoutPage"); // 특정 Layout을 지정하여 rendering하도록 전달

    return View("~/Views/Home/Index.cshtml"); // Route System을 통하지 않고 직접 특정 View를 지정
}
```

### 4.3 Action Method에서 View로 Data 전달

#### 4.3.1 ViewModel

`ViewModel`(View에서 사용할 Model)을 parameter로 전달하면 됩니다.

```CSharp
public class HomeController : Controller
{
    public ViewResult Index()
    {
        DataTime now = DateTime.Now;
        return View(now);
    }
}
```

`DateTime` 타입의 ViewModel을 전달한 경우 View에서는 2가지 방법으로 전달받은 ViewModel을 사용할 수 있습니다.

##### 4.3.1.1 Weakly Typed View

약한 형식 뷰 (또는 무형식(Untyped) 뷰)라고 불리는 방법으로, ViewModel을 특정 타입으로 지정하지않고 `object` instance로 취급합니다.
그렇기 때문에 사용시 형 변환을 해줘야 합니다.
이 경우에는 intellisense가 제공되지 않을 뿐더러, code가 다소 지저분해 질 수 있습니다.

```HTML
@{
    ViewBag.Title = "Index";
}
<h1>Index</h1>
<div>The day is: @(((DateTime)Model).DayOfWeek)</div>
```

##### 4.3.1.2 Strongly Typed View

강력한 형식 뷰에서는 `@model` 키워드로 ViewModel type을 지정합니다.
그러면 View내에서 `@Model` 키워드로 사용이 가능합니다.
이 경우 intellisense 기능을 사용할 수가 있어서 편리하며, code도 깔끔하게 만들 수 있습니다.

```HTML
@model DateTime
@{
    ViewBag.Title = "Index";
}
<h1>Index</h1>
<div>The day is: @Model.DayOfWeek</div>
```

#### 4.3.2 ViewBag

Action Method에서 `ViewBag`이라는 dynamic object에 임의의 속성을 정의하면 View에서 읽을 수 있습니다.

```CSharp
public class HomeController : Controller
{
    public ViewResult Index()
    {
        ViewBag.Now = DateTime.Now;
        ViewBag.MyName = "Luna";
        return View();
    }
}
```

```HTML
@{
    ViewBag.Title = "Index";
}
<h1>Index</h1>
<div>The day is: @ViewBag.Now.DayOfWeek</div>
<div>My name is @ViewBag.MyName</div>
```

여러 개의 값을 전달할 경우 `ViewModel`보다 편리하게 사용할 수 있습니다.
위 예제와 같이 `DateTime` 과 `string` 2개의 값을 `ViewModel`로 전달하고자 한다면, 따로 정의를 해야합니다.

```CSharp
public class IndexViewModel
{
    public DateType Now;
    public string MyName;
}
```

`TempData`와 `SessionData`를 이용한 전달도 가능한데, 이것은 `Redirect` 시에 주로 사용되므로 다음에 그때 가서 설명드리겠습니다.

## 5. Redirect (재전송)

Action Method의 결과를 직접적으로 출력하는게 아니라, 다른 URL로 재전송해야 하는 경우 사용합니다.  

>만약 `HTTP` `POST` 요청에 대해서 처리결과를 바로 출력할 경우, 사용자가 다시 `Browser Refresh`를 할 경우 `POST`요청이 다시 수행되게 됩니다.
그래서 이런 경우에는 `POST`에 관련된 처리를 한 후 그 결과를 보여주는 View를 따로 생성하여 `GET` 요청으로 `URL Redirect`를 수행하는 것이 더 안전합니다.
그럼 사용자가 `Refresh`를 하더라도 `POST` 요청이 다시 수행되지 않습니다.

`Redirect`에는 2가지 방식이 있습니다.

- `HTTP 302` : 임시적인 재전송을 의미합니다. 가장 많이 사용되는 방식이며, Post/Redirect/Get 패턴을 사용할 경우에도 `302`로 전송되어야 합니다.
- `HTTP 301` : 영구적인 재전송을 의미합니다. 이 경우 원본 URL은 더이상 사용되지 않을 것이고, `Redirect`된 새로운 URL이 앞으로 사용됩니다.

### 5.1 Action Method에서 Redirect하는 방법
```CSharp
public RedirectResult Index()
{
    return Redirect("/Example/Index"); // 문자열로 URL을 작성하여 Redirect

    return RedirectPermanent("/Example/Index"); // 영구적인 Redirect (Redirect("...", true);를 사용하는 것도 가능)
}

public RedirectToRouteResult Index()
{
    return RedirectToRoute(new {        // Redirect to Routing System URL
        controller = "Example",
        action = "Index",
        ID = "Luna"
    });

    return RedirectToAction("Index");   // Redirect to Action Method

    return RedirectToAction("Index", "Example");   // other Controller
}
```

### 5.2 Redirect시 Data 전달

#### 5.2.1 TempData

Redirect하기 전에 TempData에 필요한 값들을 저장합니다.

```CSharp
public RedirectToRouteResult Index()
{
    TempData["MyName"] = "Luna";
    TempData["Now"] = DateTime.Now;
    return RedirectToAction("Hello");
}
```

전달받은 TempData는 값을 읽기 전까지는 유지가 되며, 읽으면 삭제됩니다.
값을 읽는 방법은 `TempData["MyName"]`으로 읽을 수 있습니다.
`TempData.Peek(""MyName)`을 이용하면 값을 삭제하지 않고 읽을 수 있습니다.

이 값을 View까지 전달하고자 한다면 TempData에서 읽어서 ViewBag이나 ViewModel에 넣어서 전송을 하면 됩니다.

```CSharp
public ViewResult Hello()
{
    ViewBag.MyName = TempData["MyName"];
    ViewBag.Now = TempData["Now"];
    return View();
}
```

그냥 따로 전달하지 않고, View에서 바로 읽는 것도 가능합니다.

```HTML
<strong>@TempData.TryPeek("MyName") Entered.</strong></p>

The day is : @(((DateTime)TempData["Now"]).DayOfWeek) </p>

My Name is @TempData["MyName"]
```

#### 5.2.2 Session Data

`TempData`와 비슷한 방법으로 사용이 가능하지만, 해당 Session 내에서 계속해서 유지가 된다는 차이점이 있습니다.
일회성으로 사용할 정보는 `TempData`를 활용하는 것이 좋고, Session 내에서 계속해서 필요한 정보는 `Session`을 사용하는게 좋습니다.

## 6. HTTP Code, Error 전송

```CSharp
public HttpStatusCodeResult StatusCode()
{
    return new HttpStatusCodeResult(404, "URL cannot be serviced"); // 직접 생성해서 반환

    return HttpNotFound(); // 404

    return HttpUnauthorizedResult(); // 401
}
```
