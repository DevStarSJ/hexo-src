---
title: ASP.NET MVC 05. View
date: 2016-05-24 04:00:00
categories:
- C#
- ASP.NET
tags:
- C#
- ASP.NET
---

# View

## View 란?

MVC Framework에서 사용자에게 결과를 보여주는 역할을 합니다.

<https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller>

ASP.NET에서는 `Razor Engine`을 이용하여 View를 조금 더 편하게 작성 할 수 있습니다. (`.cshtml`)
`Razor Engine`이 무엇인지 간단하게 얘기하자면 View (HTML page) 작성시 `C#` 문법과 `.NET Framework`를 사용할 수 있습니다.
`Layout`, `Partial View`를 이용하여 특정 영역만 따로 rendering 하는 것도 가능하며,
각종 `Helper method`를 제공하여서 반복적인 HTML TAG 작성 작업을 줄여주며, 직접 `Helper method`를 작성하여서도 활용 할 수 있습니다.

View의 기본적인 사용법은 `Controller and Action` 절에서 예제 작성시 간단히 언급했으므로 생략하도록 하겠습니다.

<https://github.com/DevStarSJ/Study/blob/master/Blog/MVC/04.ControllerAndAction.md>

View를 편리하게 사용하기 위한 방법(주로 재활용 방안) 위주로 진행하겠습니다.

## 1. Layout section

Layout (주로 `_Layout.cshtml` 식의 명칭) 내부에는 section을 제공해 줍니다.

- `@RenderBody()` : 해당 Layout을 사용하는 View의 내용이 이 위치에 삽입됩니다.
- `@RenderSection("Name")` : View 에서 `@section Name { ... }` 의 내용이 해당 section에 삽입됩니다.

Layout에서 선언한 `@RenderSection("...")`이 View에서 사용하지 않으면 오류가 발생합니다.
해당 section에 선택적으로 사용을 하려면 (View에서 사용하지 않아도 오류가 발생하지 않게 하려면) `@RenderSection("...", false)`로 선언을 하면 됩니다.

*_Layout.cshtml

```HTML
// _Layout.cshtml

...

<body>
    @RanderSection("Header")

    ...

    @RanderBody()

    ...

    @RanderSection("Footer", required = false)
</body>
```

* Index.cshtml

```HTML

@{
    Layout = "_Layout";
}

@section Header {
<div>This is Header</div>
}

<div>
    RenderBody contains every contents in this document, except @section ...
</div>

```

## 2. Partial View

Razor Tag, HTML Markup 으로 이루어진 동일한 코드를 반복적으로 사용할 경우 유용합니다.

* Partial.cshtml
```HTML
<div>
    This is Partial View.
    @Html.ActionLink("The Link of Index on this Controller", "Index")
</div>
```

* List.cshtml
```HTML
@{
    ViewBag.Title = "List";
    Layout = null;
}

<h3>List</h3>

@Html.Partial("Partial")
```

위의 경우 `List.cshtml`에서 `Partial.cshtml`의 Partial view를 사용하였는데, Partial view 내부를 보면 `ActionLink`를 Controller 이름을 명시하지 않았습니다.
Partial View를 Rednering하는 곳이 List.cshtml 내부이기 때문에 해당 Controller를 기준으로 동작합니다.

Partial View도 ViewModel을 가질 수 있습니다. (Strongly Typed Partial View)

그럴 경우 `@Html.Partial(...)` 의 두번째 인자로 ViewModel을 전달해 줘야 합니다.

* Partial2.cshtml
```HTML
@model IEnumerable<string>
...
```

* List.cshtml
```HTML
...
@Html.Partial("Partial2", new [] { "Luna" , "Star" } )
```

## 3. Child Action

View 에서 호출되는 Action Method를 의미합니다.
`@Html.Action(...)` helper method를 사용하면 모든 Action method를 View에서 호출이 가능하지만,
`[ChileActionOnly]` 이란 attribute를 붙이면 Routing System에서 사용되지 않고 순수히 View에서 호출할 경우에만 동작하는 것을 의미합니다.

* HomeController.cs
```C#
...
[ChildActionOnly]
public ActionResult Time()
{
    return PartialView(DateTime.Now);
}
```

* Time.cshtml
```HTML
@model DateTime
<div>
    @Model.ToShortTimeString()
</div>
```

* List.cshtml
```HTML
...
@Html.Action("Time")
```

Child Action에 매개변수가 필요한 경우에는 무명형식을 이용해서 전달이 가능합니다.
단 파라메터의 이름이 같아야 합니다.

* HomeController.cs
```C#
...
[ChildActionOnly]
public ActionResult Time(DateTime time)
{
    return PartialView(DateTime.Now);
}
```

* List.cshtml
```HTML
...
@Html.Action("Time", new { time = DateTime.Now })
```