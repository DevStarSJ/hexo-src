---
title: 사용자 정의 MFC Control의 Message 처리
date: 2016-02-16 15:28:00
categories:
- CPP
- MFC
tags:
- CPP
- MFC
---

# 사용자 정의 MFC Control의 Message 처리

CWnd를 상속받아서 사용자가 작성한 MFC Control의 메세지 처리하는 방법을 소개해드리겠습니다.  

통상적으로 Dialog, View, FormView (앞으로 편의상 Dialog라 칭함)에 Control을 올려서 사용하는데, 
그 구현자체를 Control안에서 하는 경우도 있지만,  
해당 Control을 사용하고 있는 Dialog에서 구현을 해야하는 경우도 있습니다.  
2개 이상의 Control을 같이 활용하려면 그렇게 해야 하죠.

```
Textbox에 숫자를 적어두고 Button을 눌렀을 경우 해당 숫자를 화면에 AfxMessageBox로 출력하는 경우  
해당 처리는 Button에서 하는게 아니라 Textbox와 Button을 가지고 있는 Dialog에서 하는게 편합니다.
```

3가지 방법이 있습니다.

1. Message로 처리
2. Command로 처리
3. Notify로 처리

각각에 대해서 소개해 드리겠습니다.

## 준비사항

먼저 간단하게 Dialog 기반으로 MFC Application Project를 생성해주세요.

그런 다음 아래의 2개의 File을 추가합니다.

##### UserWnd.h
```C++
#pragma once
#include "afxwin.h"

#define WM_USER_WND		       WM_USER + 10001          // User-defined Message (#1)

class CUserWnd : public CWnd
{
public:
	HWND m_hwndDlg = nullptr;                               // HWND of Parent Dialog (#2)

protected:
	DECLARE_MESSAGE_MAP()                                   // Message Map Macro (#3)
	afx_msg void OnLButtonDown(UINT nFlags, CPoint point);  // Mouse left button click event (#4)
};
```

- #1 : 사용자 정의 메세지 입니다.
- #2 : 메세지를 보낼려면 해당 메세지를 받을 Dialog의 pointer나 HWND값이 필요합니다.
  - HWND를 알고 있는 경우 : `::SendMessage(m_hwndDlg, 메세지, WPARAM, LPARAM);`
  - Dialog의 pointer를 알고 있는 경우 : `pointer->SendMessage(메세지, WPARAM, LPARAM);`
- #3 : Message Map을 사용할려면 헤더파일에 선언해줘야할 매크로 입니다.
- #4 : 예제로 마우스 왼쪽 버튼을 눌렀을 경우 Dialog로 Message를 전달하기 위하여 해당 메세지를 사용했습니다.

##### UserWnd.cpp
```C++
#include "stdafx.h"
#include "UserWnd.h"

BEGIN_MESSAGE_MAP(CUserWnd, CWnd)                        // Message Map (#1)
	ON_WM_LBUTTONDOWN()                                  // Mouse left button click event (#2)
END_MESSAGE_MAP()

void CUserWnd::OnLButtonDown(UINT nFlags, CPoint point)  // Mouse left button click event (#3)
{
	HWND hWnd = GetSafeHwnd();

	if (hWnd == NULL) return;
	if (!::IsWindow(hWnd)) return;

	int nID = GetDlgCtrlID();

	if (m_hwndDlg != nullptr)
	{

	}
	
	CWnd::OnLButtonDown(nFlags, point);                  // Call Parent Function (#4)
}
```

- #1 : Message Map 정의 부분입니다.
  - 첫번째 인자 : 해당 class를 적어줍니다.
  - 두번째 인자 : 부모 class를 적어줍니다. 해당 class 내에 적어주지 않은 메세지에 대해서는 부모 class에서 처리하게 됩니다.
    - 만약 CXTPChartControl을 상속받아서 만든 사용자 정의 Control일 경우 두번째 인자에 CXTPChartControl을 적어줘야 합니다.
- #2 : MFC에서 미리 정의해놓은 것으로 마우스 왼쪽 버튼 눌렀을때 OnLButtonDown()을 실행하게 됩니다.
- #3 : 사용자 정의 메세지를 보내기위해 필요한 값들 HWND, ControlID를 가지고 있습니다. if 구문 안에서 각각의 메세지 타입에 따른 구현이 달라집니다.
- #4 : 부모 class에서 해당 메세지에 대한 동작을 계속 하도록 호출해 줍니다. 부모 class의 작업이 필요없다면 이 줄은 삭제하면 됩니다.

##### Resource.h

- UserWnd의 Control ID를 생성해줍니다.

```C++
#define IDC_USER_WND					10001
```

##### Dialog Header file (필자의 경우 UserCtrlMsgDlg.h)

- UserWnd.h를 추가해 주세요.

```C++
#include "UserWnd.h"
```

- CUserWnd 타입의 멤버를 선언해 주세요.

```C++
CUserWnd m_wndUser;
```

##### Dialog cpp file (필자의 경우 UserCtrlMsgDlg.cpp)

- OnInitDialog() 에서 UserWnd를 생성해 줍니다.

```C++
// TODO: Add extra initialization here
m_wndUser.m_hwndDlg = GetSafeHwnd();
m_wndUser.Create(NULL, _T(""), WS_VISIBLE | WS_BORDER, CRect(0, 0, 100, 100), this, IDC_USER_WND);
```

이로서 준비는 끝났습니다.  
위에서 언급한 3가지 방법 (Message, Notify, Command)에 대해서 어떤 방법으로 구현을 하더라도 이 준비과정은 똑같습니다.  

## 1. Message 방식

- Dialog에 해당 User Control을 1개만 사용할 경우 유용합니다.
- Dialog에 Message로 전달하는 방식입니다.
- return값을 받을수 있기 때문에 User Control에서 그 값에 따라 처리가 가능합니다.
- 대신 Message 전달 이후 return값을 받아야 하므로 해당 처리가 끝날때까지 대기하게 됩니다.
- Dialog에 같은 User Control이 여러 개 있을 경우 모두 같은 Message로 전달되므로, Parameter로 Control ID를 보내는 등의 방법을 사용하여 Dialog에서 처리하는 함수 내부에서 구분하여 사용해야 합니다.

##### UserWnd.cpp

OnLButtonDown 함수 내부에 아래와 같이 1줄을 추가

```C++
void CUserWnd::OnLButtonDown(UINT nFlags, CPoint point)
{
	HWND hWnd = GetSafeHwnd();

	if (hWnd == NULL) return;
	if (!::IsWindow(hWnd)) return;

	int nID = GetDlgCtrlID();

	if (m_hwndDlg != nullptr)
	{
		::SendMessage(m_hwndDlg, WM_USER_WND, nID, NULL);
	}

	CWnd::OnLButtonDown(nFlags, point);
}
```

##### Dialog Header file (필자의 경우 UserCtrlMsgDlg.h)

Message를 처리할 함수를 선언해주세요.

```C++
afx_msg LRESULT OnUserWnd(WPARAM wParam, LPARAM lPraram);
```

##### Dialog cpp file (필자의 경우 UserCtrlMsgDlg.cpp)

Message Map에 아래 1줄을 추가해주세요.

```C++
ON_MESSAGE(WM_USER_WND, OnUserWnd)
```

- ON_MESSAGE 경우 Message번호 와 실행할 함수를 인자로 설정해줍니다.

Command를 처리할 함수를 구현합니다.

```C++
LRESULT CUserCtrlMsgDlg::OnUserWnd(WPARAM wParam, LPARAM lPraram)
{
	int nID = (int)wParam; // 해당 ID를 비교해서 Control 구분 가능

	AfxMessageBox(_T("ON MESSAGE"));

	return TRUE;
}
```

이제 실행 후 왼쪽 상단의 검은색 선 안을 누르면 해당 메세지가 출력되는 것을 확인 할 수 있습니다.

## 2. Notify 방식

- Dialog에 해당 User Control이 여러개 있고, 전달한 메세지 종류가 2가지 이상인 경우 유용합니다.
- Dialog에 Notify로 알려주고 User Control은 계속 남은 처리를 진행합니다.

##### UserWnd.cpp

OnLButtonDown 함수 내부에 아래와 같이 NMHDR 선언과 SendMessage를 추가

```C++
void CUserWnd::OnLButtonDown(UINT nFlags, CPoint point)
{
	HWND hWnd = GetSafeHwnd();

	if (hWnd == NULL) return;
	if (!::IsWindow(hWnd)) return;

	int nID = GetDlgCtrlID();

	if (m_hwndDlg != nullptr)
	{
		NMHDR nmhdr;
		nmhdr.code = WM_USER_WND;
		nmhdr.idFrom = nID;
		nmhdr.hwndFrom = hWnd;
		::SendMessage(m_hwndDlg, WM_NOTIFY, nID, (LPARAM)&nmhdr);
	}

	CWnd::OnLButtonDown(nFlags, point);
}
```

##### Dialog Header file (필자의 경우 UserCtrlMsgDlg.h)

Message를 처리할 함수를 선언해주세요.

```C++
afx_msg void OnNotyfyUserWnd(NMHDR* pNMHDR, LRESULT* pResult);
```

##### Dialog cpp file (필자의 경우 UserCtrlMsgDlg.cpp)

Message Map에 아래 1줄을 추가해주세요.

```C++
ON_NOTIFY(WM_USER_WND, IDC_USER_WND, OnNotyfyUserWnd)
```

- ON_NOTIFY 경우 Message번호, Contril ID와 실행할 함수를 인자로 설정해줍니다.

Command를 처리할 함수를 구현합니다.

```C++
void CUserCtrlMsgDlg::OnNotyfyUserWnd(NMHDR* pNMHDR, LRESULT* pResult)
{
	AfxMessageBox(_T("ON NOTIFY"));
}
```

## 3. Command 방식

- Dialog에 해당 User Control 별로 전달할 메세지가 1가지 밖에 없을 경우 유용합니다.
  - e.g. button의 경우 눌렀을 경우에만 메세지를 전달하면 되고, 나머지 경우는 전달하지 않아도 될 경우 유용합니다.

##### UserWnd.cpp

OnLButtonDown 함수 내부에 아래와 같이 1줄을 추가

```C++
void CUserWnd::OnLButtonDown(UINT nFlags, CPoint point)
{
	HWND hWnd = GetSafeHwnd();

	if (hWnd == NULL) return;
	if (!::IsWindow(hWnd)) return;

	int nID = GetDlgCtrlID();

	if (m_hwndDlg != nullptr)
	{
		::SendMessage(m_hwndDlg, WM_COMMAND, nID, NULL);
	}

	CWnd::OnLButtonDown(nFlags, point);
}
```

##### Dialog Header file (필자의 경우 UserCtrlMsgDlg.h)

Command를 처리할 함수를 선언해주세요.

```C++
afx_msg void OnCommandUserWnd();
```

##### Dialog cpp file (필자의 경우 UserCtrlMsgDlg.cpp)

Message Map에 아래 1줄을 추가해주세요.

```C++
ON_COMMAND(IDC_USER_WND, OnCommandUserWnd)
```

- ON_COMMAND의 경우 Control ID 와 실행할 함수를 인자로 설정해줍니다.

Command를 처리할 함수를 구현합니다.

```C++
void CUserCtrlMsgDlg::OnCommandUserWnd()
{
	AfxMessageBox(_T("ON COMMAND"));
}
```

이제 실행 후 왼쪽 상단의 검은색 선 안을 누르면 해당 메세지가 출력되는 것을 확인 할 수 있습니다.
