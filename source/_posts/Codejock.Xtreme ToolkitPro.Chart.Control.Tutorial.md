---
title: Codejock Xtreme ToolkitPro Chart Control Tutorial
date: 2016-02-04 15:28:00
categories:
- CPP
- MFC
tags:
- CPP
- MFC
- ToolkitPro
---

### Codejock Xtreme ToolkitPro Chart Control Tutorial

ToolkitPro의 Chart Control 인 CXTPChartContorl의 간단한 사용법을 소개해 드리도록 하겠습니다.  
기본적으로 ToolkitPro를 설치하고 환경설정하는 방법은 생략하겠습니다.  
아래 Link를 참조해 주세요.

먼저 MFC Application Project를 생성하여 Dialog로 생성해 주세요.  

![](/images/XTPChart.01.png)

![](/images/XTPChart.02.png)

물론 ToolkitPro를 사용하려면

`stdafx.h`에 ToolkitPro 헤더파일을 추가해 주세요.
```C++
#include <XTToolkitPro.h>
```

#### 1.Resource File (.rc)에서 Dialog Design에 찾아서 아래와 같이 Control을 타이핑
```
CONTROL         "Chart", IDC_CHARTCONTROL, "XTPChartControl", WS_TABSTOP, 7, 7, 245, 186
```

#### 2. Resource.h에 추가
```C++
#define IDC_CHARTCONTROL				103
```

Resource View에서 마우스로 대충 크기 변환해주세요. 대충 아래와 같은 모양이 됩니다.

![](/images/XTPChart.03.png)

```
IDD_XTPCHARTSAMPLE_DIALOG DIALOGEX 0, 0, 427, 287
STYLE DS_SETFONT | DS_MODALFRAME | DS_FIXEDSYS | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU
EXSTYLE WS_EX_APPWINDOW
CAPTION "XTPChartSample"
FONT 8, "MS Shell Dlg", 0, 0, 0x1
BEGIN
    DEFPUSHBUTTON   "OK",IDOK,312,266,50,14
    PUSHBUTTON      "Cancel",IDCANCEL,370,266,50,14
    CONTROL         "Chart",IDC_CHARTCONTROL,"XTPChartControl",WS_TABSTOP,7,7,413,255
END
```

#### 3. 컨트럴에 마우스 우 클릭 Add Variable 누른 뒤 그림대로 추가해주세요.

![](/images/XTPChart.04.png)

위와 같이 하지 않고 직접 코딩하여도 됩니다.

* XTPChartSampleDlg.h

```C++
public:
	CXTPChartControl m_wndChartControl;
```
* XTPChartSampleDlg.cpp

```C++
void CXTPChartSampleDlg::DoDataExchange(CDataExchange* pDX)
{
	CDialogEx::DoDataExchange(pDX);
	DDX_Control(pDX, IDC_CHARTCONTROL, m_wndChartControl);
}
```

모든 구현은 `void InitChart();`를 추가하여 여기다가 하겠습니다.  


`BOOL CXTPChartSampleDlg::OnInitDialog()`에서 `InitChart();`를 호출하도록 해주세요.

#### 4. 구현

어떤 작업을 하던지 Content 객체를 가져와서 작업을 해야 합니다.

```C++
CXTPChartContent* pContent = m_wndChartControl.GetContent();
```

* Title 설정

```C++
// Title 설정
CXTPChartTitleCollection* pTitles = pContent->GetTitles();
CXTPChartTitle* pTitle = pTitles->Add(new CXTPChartTitle());
pTitle->SetText(_T("My Chart"));
```

* Series에 Point 추가

```C++
// Series, Series Point 추가
CXTPChartSeriesCollection* pCollection = pContent->GetSeries();
CXTPChartSeries* pSeries = pCollection->Add(new CXTPChartSeries());

pSeries->SetStyle(new CXTPChartLineSeriesStyle());
CXTPChartSeriesPointCollection* pPoints = pSeries->GetPoints();

pPoints->Add(new CXTPChartSeriesPoint(0, 3));
pPoints->Add(new CXTPChartSeriesPoint(1, 1));
pPoints->Add(new CXTPChartSeriesPoint(2, 2));
pPoints->Add(new CXTPChartSeriesPoint(3, 0.5));
```

실행시키면 다음과 같은 화면이 나옵니다.

![](/images/XTPChart.05.png)

여기에서 Series를 하나 더 추가해 보겠습니다.

```C++
CXTPChartSeries* pS2 = pCollection->Add(new CXTPChartSeries());
pSeries->SetName(_T("Series2"));

pS2->SetStyle(new CXTPChartLineSeriesStyle());
CXTPChartSeriesPointCollection* pPoints = pS2->GetPoints();

pPoints->Add(new CXTPChartSeriesPoint(0, 2));
pPoints->Add(new CXTPChartSeriesPoint(1, 0.5));
pPoints->Add(new CXTPChartSeriesPoint(2, 3));
pPoints->Add(new CXTPChartSeriesPoint(3, 1));
```

![](/images/XTPChart.06.png)

Series의 Label을 설정하려면 다음과 같이 하면 됩니다.  
소숫점 1자리까지 나오게 하는 예제 입니다.

```C++
// SeriesLabel의 설정
for (int i = 0; i < pCollection->GetCount(); i++)
{
	CXTPChartSeriesLabel* pLabel = pCollection->GetAt(i)->GetStyle()->GetLabel();
	pLabel->GetFormat()->SetCategory(xtpChartNumber);
	pLabel->GetFormat()->SetDecimalPlaces(1); // 소숫점 표시
}
```

![](/images/XTPChart.07.png)

Point Label 자체를 안보이게 하려면 다음 문장을 추가하면 됩니다.

```C++
pLabel->SetVisible(FALSE);
```

![](/images/XTPChart.08.png)

* 범주 보이게 하기

```C++
pContent->GetLegend()->SetVisible(TRUE);
```

![](/images/XTPChart.09.png)

* Series의 통계값 계산

아래와 같은 함수들을 지원해줘서 통계값을 쉽게 계산할 수 있습니다. Min, Max등의 다양한 함수가 많습니다.

```C++
double dArithmeticMean = pPoints->GetArithmeticMean(0);
double dVariance = pPoints->GetVariance(0);
double dStd = pPoints->GetStandardDeviation(0);
```

* 축(AXIS 설정)

```C++
// Axis 설정
//CXTPChartDiagram2D* pDiagram = DYNAMIC_DOWNCAST(CXTPChartDiagram2D, pCollection->GetAt(0)->GetDiagram());
CXTPChartDiagram* pDiagram = pCollection->GetAt(0)->GetDiagram();
CXTPChartDiagram2D* pD2D = DYNAMIC_DOWNCAST(CXTPChartDiagram2D, pDiagram);

CXTPChartAxis *pAxisX = pD2D->GetAxisX();

CXTPChartAxisTitle* pTitle = pAxisX->GetTitle();

pTitle->SetText(_T("X-Argument"));
pTitle->SetVisible(TRUE);

CXTPChartAxis *pAxisY = pD2D->GetAxisY();

CXTPChartAxisTitle* pTitle2 = pAxisY->GetTitle();

pTitle2->SetText(_T("Y-Value"));
pTitle2->SetVisible(TRUE);
```

![](/images/XTPChart.10.png)

* Series Marker 안보이게 하기

```C++
for (int i = 0; i < pCollection->GetCount(); i++)
{
	CXTPChartPointSeriesStyle* pStyle = (CXTPChartPointSeriesStyle*)pCollection->GetAt(i)->GetStyle();
	pStyle->GetMarker()->SetVisible(FALSE);
	//pStyle->GetMarker()->SetSize(20); // Maeker Size 조정
	//pStyle->GetMarker()->SetType(xtpChartMarkerCircle); // enum XTPChartMarkerType
}
```

![](/images/XTPChart.11.png)

* 마우스 휠을 이용한 Zoom 허용 및 Scroll 허용

```C++
pD2D->SetAllowZoom(TRUE);	// 마우스 휠을 이용한 Zoom 허용
pD2D->SetAllowScroll(TRUE); // Scroll 허용
```

![](/images/XTPChart.12.png)

* Chart Image 저장

```C++
m_wndChartControl.SaveAsImage(_T("D:\\A.PNG"),CSize(600,400));
```

![](/images/A.PNG)

### 위 설명한 내용의 Full Source

```C++
void CXTPChartSampleDlg::InitChart()
{
	// Content를 이용해서 Chart의 Title, Series, Legends등의 설정이 가능
	CXTPChartContent* pContent = m_wndChartControl.GetContent();
	if (!pContent) return;

	pContent->GetLegend()->SetVisible(TRUE);

	// Title 설정
	CXTPChartTitleCollection* pTitles = pContent->GetTitles();
	if (pTitles)
	{
		CXTPChartTitle* pTitle = pTitles->Add(new CXTPChartTitle());
		if (pTitle)
		{
			pTitle->SetText(_T("My Chart"));
		}
	}

	// Series, Series Point 추가
	CXTPChartSeriesCollection* pCollection = pContent->GetSeries();
	if (pCollection)
	{
		CXTPChartSeries* pSeries = pCollection->Add(new CXTPChartSeries());
		if (pSeries)
		{
			pSeries->SetName(_T("Series1"));
			pSeries->SetStyle(new CXTPChartLineSeriesStyle());
			CXTPChartSeriesPointCollection* pPoints = pSeries->GetPoints();
			if (pPoints)
			{
				pPoints->Add(new CXTPChartSeriesPoint(0, 3));
				pPoints->Add(new CXTPChartSeriesPoint(1, 1));
				pPoints->Add(new CXTPChartSeriesPoint(2, 2));
				pPoints->Add(new CXTPChartSeriesPoint(3, 0.5));

				double dArithmeticMean = pPoints->GetArithmeticMean(0);
				double dStd = pPoints->GetStandardDeviation(0);

			}
		}

		CXTPChartSeries* pS2 = pCollection->Add(new CXTPChartSeries());
		if (pS2)
		{
			pS2->SetName(_T("Series2"));
			pS2->SetStyle(new CXTPChartLineSeriesStyle());
			CXTPChartSeriesPointCollection* pPoints = pS2->GetPoints();
			if (pPoints)
			{
				pPoints->Add(new CXTPChartSeriesPoint(0, 2));
				pPoints->Add(new CXTPChartSeriesPoint(1, 0.5));
				pPoints->Add(new CXTPChartSeriesPoint(2, 3));
				pPoints->Add(new CXTPChartSeriesPoint(3, 1));
			}
		}

		// SeriesLabel의 설정
		for (int i = 0; i < pCollection->GetCount(); i++)
		{
			CXTPChartSeriesLabel* pLabel = pCollection->GetAt(i)->GetStyle()->GetLabel();
			pLabel->GetFormat()->SetCategory(xtpChartNumber);
			pLabel->GetFormat()->SetDecimalPlaces(1); // 소숫점 표시
			pLabel->SetVisible(FALSE);
		}

		// Axis 설정
		//CXTPChartDiagram2D* pDiagram = DYNAMIC_DOWNCAST(CXTPChartDiagram2D, pCollection->GetAt(0)->GetDiagram());
		CXTPChartDiagram* pDiagram = pCollection->GetAt(0)->GetDiagram();
		CXTPChartDiagram2D* pD2D = DYNAMIC_DOWNCAST(CXTPChartDiagram2D, pDiagram);
		if (pD2D)
		{
			CXTPChartAxis *pAxisX = pD2D->GetAxisX();
			if (pAxisX)
			{
				CXTPChartAxisTitle* pTitle = pAxisX->GetTitle();
				if (pTitle)
				{
					pTitle->SetText(_T("X-Argument"));
					pTitle->SetVisible(TRUE);
				}
			}

			CXTPChartAxis *pAxisY = pD2D->GetAxisY();
			if (pAxisX)
			{
				CXTPChartAxisTitle* pTitle = pAxisY->GetTitle();
				if (pTitle)
				{
					pTitle->SetText(_T("Y-Value"));
					pTitle->SetVisible(TRUE);
				}
			}
		}

		// Marker 안보이게 하기
		for (int i = 0; i < pCollection->GetCount(); i++)
		{
			CXTPChartPointSeriesStyle* pStyle = (CXTPChartPointSeriesStyle*)pCollection->GetAt(i)->GetStyle();
			pStyle->GetMarker()->SetVisible(FALSE);
			//pStyle->GetMarker()->SetSize(20); // Maeker Size 조정
			//pStyle->GetMarker()->SetType(xtpChartMarkerCircle); // enum XTPChartMarkerType
		}

		pD2D->SetAllowZoom(TRUE);	// 마우스 휠을 이용한 Zoom 허용
		pD2D->SetAllowScroll(TRUE); // Scroll 허용

		m_wndChartControl.SaveAsImage(_T("D:\\A.PNG"),CSize(600,400));
	}
}
```

### 예제 소스

<https://github.com/DevStarSJ/Cpp/tree/master/MFC/XTPChartSample>
