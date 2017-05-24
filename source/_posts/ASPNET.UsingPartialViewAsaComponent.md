---
title: Using Partial View as a Component
date: 2017-05-24 15:20:00
categories:
- C#
- ASP.NET
tags:
- C#
---

## Using Partial View as a Component

**Partial View**를 이용해서 View의 특정부분을 분리하여 관리가 가능하다.  
**Partial View**안에 관련 코드들(script, html...) 들을 모아놓고 그 중 특정 부분만 사용하는 것도 가능하다.  
이걸 이용하면 관련된 기능들을 하나의 **Partial View**에 넣어두고 관리하는게 가능해진다.

예를 들어서

- 여러 개의 버튼들을 모아놓은 html
- 해당 버튼을 눌렀을때 동작하는 script
- 버튼을 눌렀을때 pop-up 처럼보이게 visible처리되는 div 들...

위 3개의 코드들을 각각 별게의 위치에서 불러와야 하는 경우에도 1개의 **Parial View** 파일에 넣어둘 수 있는 방법이 있다.

### _ButtonComponents.cshtml
```HTML
@{
	var section = ViewData["section"] as string;
    ...
}

@if (section == "button") {
	@Model.status
	<button id="btn_ok">
        ...
	</button>
	<button id="btn_select">
        ...
	</button>
}

@if (section == "script") {
	<script type="text/javascript">
        ...
	</script>
}

@if (section == "popup") {
	<div id="statusModal" class="modal" style="width:300px; top: 400px;">
		<div class="modal-content">
			<div class="modal-header">
				<h3>선택</h3>
			</div>
            <select id="select" style="display: inline-block; width: 280px;">
                <option value="" selected>선택</option>
                <option value="1">1</option>
                <option value="2">확2</option>
            </select>
			<div class="btn-group align-right">
				<button id="select_ok">확인</button>
				<button id="select_cancel">취소</button>
			</div>
		</div>
	</div>
}
```

위와 같이 선언된 **Partial View** 를 보면 `section`값이 `script` , `button`, `popup`인 3개의 영역으로 이루어져 있다. 이제 사용하는 곳에서 `section`값을 함께 전달하여 원하는 부분의 코드만 불러오면 된다.

```HTML
@model Models.SampleViewModel
@{
    ...
}

@section scripts{
<script type="text/javascript">
    ...
</script>
@Html.Partial("_ButtonComponents", Model, new ViewDataDictionary { { "section", "script" } } )
}

...

<table>
    <tr>
        <td>
            @Html.Partial("_ButtonComponents", Model, new ViewDataDictionary { { "section", "button" } } )
        </td>
    </tr>
</table>

...
@Html.Partial("_ButtonComponents", Model, new ViewDataDictionary { { "section", "popup" } })
```
