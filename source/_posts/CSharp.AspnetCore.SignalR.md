---
title: Using SignalR in ASP.NET Core
date: 2016-05-09 00:00:00
categories:
- C#
- ASP.NET Core
tags:
- C#
- ASP.NET Core
---

`SignalR`을 `ASP.NET 5 Template`에서 사용하기 위해서는 `Startup.cs`의 `public void Configure(IApplicationBuilder app, ...)`에서 아래와 같은 구문이 필요합니다.

```C#
public class Startup
{
	public void Configure(IApplicationBuilder app)
	{
		app.UseServices(services =>
		{
			services.AddSignalR();
		});
		app.UseFileServer();
		app.UseSignalR();
	}
}
```

`app.UserServices()`를 사용하기 위해서는 `Microsoft.AspNet.RequestContainer` assembly를 포함시켜야 `IApplicationBuilder`의 extention method인 `UseServices`의 사용이 가능한데, 현재 version에서는 `ASP.NET 4.5.1`을 지원하지 않아서 사용이 불가능 합니다.

<https://github.com/aspnet/Hosting/tree/8f16060f941b71551be09015d76efb86770d84d7/src/Microsoft.AspNet.RequestContainer>

위 Github repository의 `ContainerExtensions.cs`에 구현되어 있습니다.

추후에라도 적용시키기 위해서는 아래 Link를 참고하시기 바랍니다.

<http://dotnetthoughts.net/using-signalr-in-asp-net-5/>
