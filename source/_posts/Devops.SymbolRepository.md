---
title: Make Symbol Repository for Visual Studio
date: 2016-04-04 00:00:00
categories:
- DevOps
tags:
- DevOps
- CI/CD
---

# Make Symbol Repository for Visual Studio

### 1. Debugging Tools for Windows 설치

<https://msdn.microsoft.com/en-us/library/windows/hardware/ff551063(v=vs.85).aspx>

### 2. Symbol 등록

```
call svnindex.cmd -source="{SolutionFolder}" -symbols="{SolutionFolder}\bin\Unicode Release"
```

```
symstore.exe add /r /f "{SolutionFolder}\bin\Unicode Release\*.*" /s "{SymbolRepository}" /t "{Name}" /compress
```

>e.g.  
call svnindex.cmd -source="d:\svn\trunk" -symbol="d:\svn\trunk\bin\Unicode Release"  
symstore add /r /f "d:\svn\trunk\bin\Unicode Release\*.*" /s "d:\symbol" /t "MyApp" /compress

- 통상적으로 `symstore.exe`는 아래 폴더에 위치합니다.
```
C:\Program Files\Debugging Tools for Windows (x64)
```
- `svnindex.cmd`는 설치된 Debugging Tools for Windows 폴더 아래 `srcsrv` 폴더 내에 있으며 다른 CI 제품들의 cmd 파일들도 존재합니다.

### 3. 저장된 Symbol 확인

- `{SymbolRepository}\000Admin\server.txt`에서 확인
- 각 ID별 Build일시. 이름 (/t 옵션 뒤에 이름) 확인이 가능

### 4. Symbol 삭제

```
symstore del /i ID /s "{SymbolRepository}"
```

>e.g.  
symstore del /i 142 /s "d:\symbol"

### symstore 사용법 (Symbol 삭제 포함)
<https://msdn.microsoft.com/en-us/library/windows/desktop/ms681378(v=vs.85).aspx>
