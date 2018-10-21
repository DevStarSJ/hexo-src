---
title: Jupyter Notebook Matplotlib에서 한글폰트 사용하기 (macOS)
date: 2018-10-13 13:00:00
categories:
- Tips
- JupyterNotebook
tags:
- JupyterNotebook
- Matplotlib
- 한글폰트
---

# Jupyter Notebook Matplotlib에서 한글폰트 사용하기 (macOS)

이 포스트를 보기 전에 먼저 [박조은](https://www.facebook.com/zzonee)님의 [강의 : Matplotlib에서 한글 폰트 사용하기](https://programmers.co.kr/learn/courses/21/lessons/950)를 먼저 보고 따라해보기를 바란다.

그런데, 이 글을 보고 있단것은 아마 위 강의대로 다 따라해보았는데도 제대로 안됬을 경우라 생각된다. 사실 필자도 저 강의대로 했는데 안되어서 여러 가지 시도를 해보고 알아낸 방법이다. 참고로 해당 설명은 **macOS** 기준인데, Linux도 비슷할 것이라 생각이되며, Windows의 경우도 어느 정도 참고는 할 수 있을것 같다.

아래의 과정들 중 위 강의를 진행하면서 확인된 결과 필요없다고 판단되는 경우에는 skip 가능한 단계도 있을 수 있다.

## 1. 관련 폴더 위치 알아내기

먼저 위 강의에 있는 코드대로 실행하면서 관련 폴더의 위치를 알아 놓자.

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Python/JupyterNotebook/image/matplotlib.korean.01.png)

여기서 `캐시 위치`와 `설정 파일 위치`값을 잘 기억해 놓아야 한다.

## 2. 시스템에 저장된 한글 폰트 위치 확인

위 강의 과정에서 설치된 한글 폰트가 없다고 판단되는 경우에는 아래 단계가 필요하다.

**Launchpad** 에서 **서체관리자** 를 찾아서 실행하라.

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Python/JupyterNotebook/image/matplotlib.korean.03.png)

거기서 적용할 한글폰트를 하나 찾는다.

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Python/JupyterNotebook/image/matplotlib.korean.04.png)

필자의 경우 **나눔바른고딕**을 선택했다. 해당 폰트에서 마우스 우클릭하여 **Finder에서 보기**를 누른다.

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Python/JupyterNotebook/image/matplotlib.korean.05.png)

**Finder**에서 파일 위치를 확인해보자. 필자의 경우 `/Library/Fonts`내에 있는것이 확인되었다. 이 경로를 기억해 놓자.

## 3. 한글폰트를 Matplotlib 설정 파일 위치로 복사

앞에서 확인한 `설정 파일 위치` 정보가 필요하다. 그 경로 가장 마지막의 `matplotlibrc`는 파일명이고 그 바로 앞까지가 경로이다. 해당 경로 아래에 있는 `/fonts/ttf/`로 앞에서 찾은 한글 폰트 파일을 복사한다.

```shell
cp /Library/Fonts/Nanum*.ttf /anaconda3/lib/python3.6/site-packages/matplotlib/mpl-data/fonts/ttf/
```

## 4. Matplotlib 캐시 파일 리셋

앞에서 확인한 `캐시 위치`로 이동해서 파일들을 확인해보자. 각자 사용자명에 따라 경로가 다를 것이다.

```shell
cd /Users/sjyun/.matplotlib
ls
```

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Python/JupyterNotebook/image/matplotlib.korean.02.png)

정확하지는 않지만, **fontList.json** 은 Python 2.0용 **fontList-v300.json**은 Python 3.0용으로 추정된다. 캐시 파일을 삭제한 뒤 새로 실행하면 캐시 파일이 새로 생성되는데, 그렇다고 파일 삭제는 위험하니 파일명 뒤에 .backup를 붙여주도록 하자.

```shell
mv fontList.json fontList.json.backup
mv fontList-v300.json fontList-voo.json.backup
```

## 5. Jupyter Notebook 및 Matplotlib 재실행

이제 위 변경사항들을 새로 적용시키기 위해서는 **Jupyter Notebook**과 **Matplotlib**의 재실행이 필요하다. 다시 실행한 다음 코드상에서 `import matplotlib`를 실행하면 캐시 파일이 새로 생성 된다.

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Python/JupyterNotebook/image/matplotlib.korean.06.png)

이제 다시 예제 코드를 돌려보면 한글이 제대로 나오는 것을 확인 할 수 있다.

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Python/JupyterNotebook/image/matplotlib.korean.07.png)

## 6. 정리

한글 폰트 및 각종 필요한 위치 확인 및 예제 실행은 아래 노트북 파일을 참고하면 된다.

<https://github.com/DevStarSJ/Study/blob/master/Blog/Python/JupyterNotebook/matplotlib.korea_font.ipynb>

이 예제코드도 [박조은](https://www.facebook.com/zzonee)님의 [강의 : Matplotlib에서 한글 폰트 사용하기](https://programmers.co.kr/learn/courses/21/lessons/950)를 참고한 것이다.