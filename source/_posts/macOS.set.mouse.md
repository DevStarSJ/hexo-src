---
title: Using the gesture function with a common mouse in macOS
date: 2016-05-07 00:00:00
categories:
- Tips
- macOS
tags:
- macOS
---

## macOS(OSX)에서 일반 마우스로 매직마우스의 제스쳐(gesture) 기능 사용하기

맥북프로나 맥북에어의 경우 마우스없이 매직패드만으로도 사용이 편리하다는 분들이 많긴하지만,
그래도 마우스를 사용하는게 익숙하신 분들은 마우스 사용을 선호합니다.

이 경우 매직패드에서 제스쳐로 제공하는 기능이 편리하기 때문에 해당 기능을 사용하기 위해서 매직마우스를 구입하는 경우가 많습니다.
하지만 매직마우스 가격이 좀 후덜덜하죠. (99,000원)
오픈마켓에서 조금 저렴하게 구입하더라도 8만원이 넘습니다.

### 키보드 단축키 확인 및 재할당

사실 매직마우스에서 주로 사용하는 제스쳐 중 `Mission Control`, `왼쪽 Space`, `오른쪽 Space` 이 3가지만 사용하더라도 작업이 편리해 집니다.

기본적으로 이 기능을 굳이 매직마우스를 사용하지 않고 `키보드 단축키`를 사용하는 것도 가능합니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Apple/image/macOS.set.mouse.01.png?raw=true">

`^`는 `control`키를 뜻합니다.
`control`키와 방향키 4개를 이용해서 해당 기능들이 기본으로 할당되어 있습니다.

### Mission Control 기능을 마우스에 할당

이 기능들을 마우스 버튼에 할당하려면 `시스템 환경설정` 에서 `Mission Control`에서 가능합니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Apple/image/macOS.set.mouse.02.png?raw=true">

아래쪽에보면 키보드로는 기본 단축키들이 할당되어 있습니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Apple/image/macOS.set.mouse.03.png?raw=true">

그 오른쪽을 클릭하면 마우스버튼 또는 마우스 버튼 + 특수키(`control` , `alt`, `command`, `shift`) 조합을 이용해서도 가능합니다.


여기에서 마우스버튼을 할당하려고 눌러보면 마우스 버튼이 32번까지 할당이 가능합니다.
하지만, 일반 마우스의 경우 마우스 버튼을 *macOS* 에서 인식하기 위해서는 전용 드라이버를 설치하거나, 별도의 프로그램을 사용해야 합니다.
그리고 결정적으로 여기에서는 **Space** 간 이용에 대해서는 할당이 안됩니다.

### BetterTouchTool

예전에는 무료 프로그램도 많았지만 요즘 대부분 유료화 되었으며 그래도 그 중 가장 기능이 많고 사용하기 편한 것으로는 `BetterTouchTool`가 있습니다.
<https://www.boastr.net/downloads>에서 45일 *Trial* 로 사용이 가능하며 4,500원에서 50,000원 정도의 가격으로 구매가 가능합니다.
제작사는 7,000원 정도의 가격으로 구매해주기를 추천한다고 되어 있습니다.
가격별로 기능적 차이는 없습니다.
그냥 *donation* 적인 측면이라 생각하시면 될 것입니다.

`BetterTouchTool` 의 원래 목적은 *macOS* 및 개별 Application 별로 키보드, 매직마우스, 일반마우스 의 특정 버튼 및 제스쳐에 기능들을 할당하는 것이므로 매직마우스 사용자라도 구매하시면 편리하게 사용이 가능합니다.

일반마우스 사용자는 이 툴을 이용해서 기본적으로 *macOS* 에서 할당되지 않은 버튼이나 버튼 + 특수키(`control` , `alt`, `command`, `shift`)의 조합으로 `Mission Control`, `왼쪽 Space`, `오른쪽 Space` 등의 기능을 할당 할 수 있습니다.

### 제조사 제공 마우스 드라이버

필자는 **Logitech** 사의 `MX Anywhere 2` 마우스를 사용중인데, 제조사에서 *macOS* 용 드라이버를 따로 제공해 줍니다.
이렇게 별도의 드라이버를 제공하는 마우스의 경우에는 거기서 설정이 가능한 경우가 많습니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Apple/image/macOS.set.mouse.04.png?raw=true">

`MX Anywhere 2`는 가운데 버튼에 `Mission Control`, `왼쪽 Space`, `오른쪽 Space` 이 기본적으로 할당되어 있습니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Apple/image/macOS.set.mouse.05.png?raw=true">

- `Mission Control` : 가운데 버튼 클릭 또는 누른체 위로 이동
- `왼쪽 Space` : 가운데 버튼 누른체 좌로 이동
- `오른쪽 Space` : 가운데 버튼 누른체 우로 이동
- `응용 프로그램 윈도우` : 가운데 버튼 누른체 아래로 이동

`Space 이동`의 경우는 비교적 자주 사용하는 기능이므로 좀 더 빨리 사용하기 위해 마우스 휠을 좌, 우로 클릭하는 것이 더 편리할 것으로 판단되어 그렇게 할당을 하였습니다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Apple/image/macOS.set.mouse.06.png?raw=true">

### 마치며...

해당 제품이 아닌 다른 마우스를 구입할 경우 제조사에서 드라이버를 제공해주지 않는다 하더라도 `BetterTouchTool` 툴을 이용해서 할당이 가능합니다.
단, 마우스의 별도 버튼이 있거나, 휠을 좌,우 클릭으로 누르는게 가능한 마우스를 구매하셔야 좀 더 편리하게 사용이 가능하리라 생각됩니다.

조금 저렴한 버튼이 많은 마우스(휠 좌,우 클릭 지원)와 `BetterTouchTool` 구매를 하더라도 매직마우스 보다는 확실히 더 저렴하니깐요.

매직마우스 + `BetterTouchTool` 의 조합의 경우는 더 많은 작업이 가능하겠지만요.
세손가락 클릭, 네손가락 클릭 등 매직마우스 상의 제스쳐를 이용해서 벼나별 기능의 할당이 가능합니다.
