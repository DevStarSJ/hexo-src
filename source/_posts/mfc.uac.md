---
title: MFC UAC 관련 사항 정리
date: 2016-02-01 15:28:00
categories:
- CPP
- MFC
tags:
- CPP
- UAC
- MFC
---

## MFC UAC 관련 사항 정리

### Project의 UAC 설정

Project -> Properties -> Linker -> Manifest File -> `UAC Execution Level`

- asInvoker : 응용 프로그램을 시작한 프로세스와 동일한 권한으로 응용 프로그램이 실행됩니다. 관리자 권한으로 실행을 선택하면 응용 프로그램의 권한 수준을 높일 수 있습니다.
- requireAdministrator: 응용 프로그램이 관리자 권한으로 실행됩니다. 응용 프로그램을 시작하는 사용자는 관리자 그룹의 멤버이어야 합니다. 응용 프로그램을 여는 프로세스가 관리자 권한으로 실행되고 있지 않은 경우 자격 증명을 입력하라는 메시지가 표시됩니다.
- highestAvailable: 최대한 높은 권한 수준으로 응용 프로그램이 실행됩니다. 응용 프로그램을 시작하는 사용자가 관리자 그룹의 멤버이면 이 옵션은 requireAdministrator와 같습니다. 사용 가능한 가장 높은 권한 수준이 응용 프로그램을 여는 프로세스의 수준보다 높으면 자격 증명을 입력하라는 메시지가 표시됩니다.
 
* MFC Manifest의 UAC 정보 : <https://msdn.microsoft.com/library/bb384691(v=vs.110).aspx>

### UAC Elevation (UAC Escalation)

asInvoker 권한의 Application 에서 requireAdministrator를 호출하는 경우 권한 상승을 요구하는 창이 뜹니다.  
해당 창에서 Administrator권한을 가진 User로 인증을 하면 실행이 가능하게 됩니다.

MFC에서 다른 Applicationd을 실행할때 `CreateProcess()`함수를 많이 사용하는데 이 함수로는 UAC 권한상승을 할 수 없습니다.  
`ShellExecute()` 라는 함수를 이용하면 가능합니다.

* ShellExecute : <https://msdn.microsoft.com/ko-kr/library/windows/desktop/bb762153(v=vs.85).aspx>

### UAC 관련 Troubleshooting

#### Project를 asInvoker로 했는데도 계속 방패마크가 남아있으면서 Administrator 권한을 요구하는 경우

##### 1. Project -> Propertise -> Manifest Tool -> Input and Oupput -> `Embed Manifest` 항목을 `Yes`로 설정
  - 이렇게 했을 경우 Compile시 아래와 같은 오류가 발생할 수 있습니다.

>CVTRES : fatal error CVT1100: duplicate resource.  type:MANIFEST, name:1. language:0x0409  
LINK : fatal error LNK1123: COFF로 변환하는 동안 오류가 발생했습니다. 파일이 잘못되었거나 손상되었습니다.  

#####2 . 위 오류가 발생할 경우 해당 Project의 Resource View로 가서 MANIFEST 관련항목 삭제

#### UAC 권한 문제로 Application 간의 Drag&Drop이 안 될 경우

* 해당 Link 참고 : <http://devluna.blogspot.kr/2014/12/mfc-window7-file-drag-drop.html>

