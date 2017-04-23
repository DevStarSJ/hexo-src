---
title: ASP.NET MVC 02. URL Routing
date: 2016-05-07 01:00:00
categories:
- C#
- ASP.NET
tags:
- C#
- ASP.NET
---

# ASP.NET MVC URL Routing

## 1. URL Routing이란

ASP.NET MVC로 생성된 Web Application의 경우 아래와 같은 형식으로 접근이 가능합니다.

```
http://mySite.com/Home/Index
```

기본 URL Routing 형식으로 해석을 할 경우
- `mySite.com` : Web Application의 주소
- `Home` : Controller 명칭
- `Index` : Action method 명칭

으로 동작합니다.

## 2. RouteConfig.cs

### 2.1 `Global.asax.cs` 를 보면 아래와 같이 Route를 설정하는 Code가 있습니다.
```C#
RouteConfig.RegisterRoutes(RouteTable.Routes);
```

### 2.2 `\App_Start\RouteConfig.cs` 파일에 `RouteConfig.RegisterRoute`라는 static method가 있습니다.

매개변수로 전달받은 `RouteCollection route`에 `.MapRoute()`로 Routing 규칙들을 추가하는 Code들이 있습니다.

## 3. `.MapRoute()` parameters

### 3.1 name

Mapping할 Routing 명칭입니다. 별 의미가 없으므로 `null`로 입력해도 됩니다.

### 3.2 url

URL 패턴을 지정합니다.  
`Web Application 주소/` 다음부터의 패턴을 지정 할 수 있습니다.  

- Static URL Segment : url안에 일반 문자열로 입력이 가능합니다.
- Dynamic URL Segment : 중괄호`{ }`내에 `변수명`을 입력하면 해당 ActionMethod에서 `RouteData.Vaues["변수명"]`으로 읽을 수 있습니다.  

ex) `url: "Page{page}"`로 설정할 경우 `http://mySite.com/Page3`으로 접근하면 해당 Route로 mapping되며 `{page}`에 3의 값이 전달됩니다.  

아래의 3가지 명칭은 변수명으로 지정이 불가능 합니다.
- `controller` : Controller 이름을 지정합니다.
- `action` : Action Method 이름을 지정합니다.
- `area` : Area를 지정합니다.

만약 해당 `Controller`의 `Action Method`의 `argument`와 같은 변수명으로 Mapping하게되면 `Action Method`의 `argument`로 그 값을 전달합니다.

ex) `url : "{controller}/{action}/{id}`로 설정하였을 경우,  
`http://mySite.com/Product/Index/Luna`란 주소로 접근할 경우
`ProductController`에 `Index(string id)`라는 Action Method가 있으면 `id`라는 argument를 `Luna`로 전달하게 됩니다.

#### 가변길이 parameter

`{controller}/{action}/{*all}` 이라고 정의한 경우 segment가 3개 이상인 경우 모두 all변수에 할당되어서 Matching됩니다.  

### 3.3 defaults

기본값 설정이 가능합니다.

```C#
defaults : new { controller = "Product", action = "Index", id = "Luna" }
```

와 같은 형식으로 지정이 가능합니다.  
해당 URL와 매칭되는 경우 URL 패턴의 변수명과 같은 곳에 아무런 값도 안 넣은 경우 default로 설정된 값이 사용되며,  
URL 패턴에 없는 변수명을 default로 설정한 경우 해당 변수명(또는 controller, action)에 대해서는 default로 설정된 값으로 동작합니다.  

- Optinal로 설정될 경우 `id = UrlParameter.Optional` 해당 segment는 입력되지 않아도 Matching 된 것으로 간주됩니다.

### 3.4 constratins

URL parameter에 제약사항을 설정합니다.  

#### 3.4.1 Regular Expression (정규표현식)을 이용하여 검사

```C#
constraints: new { page = @"\d+" }
```

위의 경우 page의 값은 숫자형식만 가능하게 됩니다.  

#### 3.4.2 여러 개의 값을 지정하여 검사

```C#
constraints: new { page = @"\d+", action="^Index$|^About$"}
```

위의 경우 추가로 action segment가 Index 나 About인 URL만 Matching합니다.

#### 3.4.3 HTTP Method를 사용하여 검사

```C#
constraints: new { page = @"\d+", action="^Index$|^About$",
    httpMethod = new HttpMethodConstarint("GET", "POST")}
```

위의 경우 추가로 GET, POST 요청만 처리하도록 제한합니다.

#### 3.4.4 형식,범위 등의 제약조건

```C#
constraints: new { page = @"\d+", action="^Index$|^About$",
    httpMethod = new HttpMethodConstarint("GET", "POST"), 
    id = new RangeRouteConstraint(10, 20) }
```

위의 경우 추가로 id값이 10에서 20 사이에 경우에만 허용합니다.  

```C#
using System.Web.Mvc.Routing.Constarints;
```

위 namespace에 제약조건 class들이 정의되어 있으며 목록은 아래 link에서 확인이 가능합니다.

<https://msdn.microsoft.com/ko-kr/library/system.web.mvc.routing.constraints(v=vs.118).aspx>

### 3.5 namespaces

`string[]` 타입으로 여러개의 namespace 값을 전달 받습니다.  
해당 값을 전달받은 경우 먼저 전달받은 namespace내의 controller부터 찾게 됩니다.  
배열로 전달받은 namespace들은 모두 동일한 우선순위를 가지게 됩니다.  
만약 특정 namespace에 우선순의를 두고자 할 경우 `MapRoute()`를 따로 설정하여야 합니다.  

## 4. Segment Matching 규칙

- 위에 설정한 규칙부터 차례대로 적용합니다.
- 그러므로 동일한 segment를 가지는 Route를 여러개 설정할 경우 좀 더 특수한 경우를 먼저 선언해야 적용됩니다.

ex)

```C#
routes.MapRoute(
    name: null,
    url: "Luna{controller}/{action}"
);

routes.MapRoute(
    name: null,
    url: "{controller}/{action}"
);
```

위와 같이 설정했을 경우에는  
- `http://mySite.com/LunaProduct/Index`로 접근할 경우 `Product` controller의 `Index` Action Method를 실행하게 되며,
- `http://mySite.com/Product/Index`로 접근할 경우에도 `Product` controller의 `Index` Action Method를 실행하게 됩니다.

```C#
routes.MapRoute(
    name: null,
    url: "{controller}/{action}"
);

routes.MapRoute(
    name: null,
    url: "Luna{controller}/{action}"
);

```

위와 같이 설정되었을 경우 segment가 2개인 경우 무조건 위에 것만 처리되며 아래에 설정된 URL패턴으로는 검사하지 않습니다.  
즉, 
- `http://mySite.com/LunaProduct/Index`로 접근할 경우 `LunaProduct` controller의 `Index` Action Method를 실행하게 되며, (없으면 404 오류)  
- `http://mySite.com/Product/Index`로 접근할 경우에도 `Product` controller의 `Index` Action Method를 실행하게 됩니다.

## 5. Attribute Routing

위에 살펴본 것은 Rule-based Routing (규칙기반 라우팅)이었습니다.  
각각의 Action Method에 Attribute를 이용하여 적용하는 방법도 있습니다.  

### 5.1 Attribute Route가 동작하도록 설정

`RouteConfig.cs`의 `RegisterRoutes(RouteCollection routes)`함수 내에  `routes.MapMvcAttributeRoutes();`를 호출하면 Attribute 기반의 Route 기능을 활성화합ㄴ다.

### 5.2  Attribute Route 설정
```C#
public class CustomerController : Controller
{
    [Route("Test")]
    public ActionResult Index()
    {
        ...
    }
}
```

와 같이 경우 `http://mySite.com/Test`로 접근할 경우 `Customer` Controller의 `Index` Action method가 실행됩니다.

```C#
[Route("User/Add/{user}/{id}"]
public string Create(string user, int id) { ... }
```

`user`와 `id`가 인자로 전달됩니다.  
각각의 인자에 제약조건 설정도 가능합니다.
- `{id:int}` : id는 int 값으로전달이 가능합니다.
- `{password:alpha:length(6)}` : password에 alpha와 length 2가지 제약조건을 설정합니다.

### 5.3 Prefix(접두어) 사용

```C#
[RoutePrefix("User")]
public class UserController : Controller
{
    [Route("~/Test")]
    public ActionResult Index() { ... }
    
    [Route("Add/{user}")]
    public ActionResult Create(string user) { ... }
}
```

- `http://mySite.com/User/Add/Luna`로 접근할 경우 `Create` Action method가 실행됩니다.
- `http://mySite.com/Test`로 접근할 경우 `Index`가 실행됩니다. 즉,  `~/`로 설정할 경우 Prefix가 무시됩니다.
