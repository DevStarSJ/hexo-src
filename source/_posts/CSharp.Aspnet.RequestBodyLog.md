---
title: Get Request Body in Action Method
date: 2017-01-23 00:00:00
categories:
- C#
- ASP.NET
tags:
- C#
- ASP.NET
---

# Get Request Body in Action Method

```C#
string body = "";
Request.InputStream.Seek(0, SeekOrigin.Begin);
using (StreamReader reader = new StreamReader(Request.InputStream))
{
    body = reader.ReadToEnd();
}
```