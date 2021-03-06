# Multithread in MFC 
About develop multi-thread MFC application.

>1. [MFC 는 어떻게 동작하는가?](#MFC는-어떻게-동작하는가?)   
>>- [Message Queue](#Message-Queue)     

>2. [Thread도 한번 알아보자.](#MFC는-어떻게-동작하는가?)   
>>- [동기화](#MFC는-어떻게-동작하는가?)   
>>- [병목](#MFC는-어떻게-동작하는가?)

</br>

## MFC는 어떻게 동작하는가?
MFC를 사용해 Multi-thread App을 개발 할 때 중요한것 중 하나는 GUI Control에 대한 입력은 즉시 처리되지 않는다는 것이다.   

</br>

사용자가 마우스를 사용해 Button Control을 클릭한다면 App에서는 세가지의 동작을 수행하게 될 것이다.   
1. 마우스의 현재 위치를 파악한다.   
2. 마우스 왼쪽 버튼클릭 이벤트 처리.   
3. 위 두 과정을 바탕으로 Button Control이 클릭되었는지 판단 할 것이다.</br> 

그리고 여러분은 버튼을 눌렀을 때의 동작을 정의하기위해 `BN_CLICKED`를 어떤 방법으로든 재정의 했을것이다. 
```cpp
BEGIN_MESSAGE_MAP(CMFCApplicationDlg, CDialogEx)
	ON_BN_CLICKED(IDOK, &CMFCApplication1Dlg::OnBnClickedOk)
END_MESSAGE_MAP()
```
코드에는 위와같은 구문이 생겼을 것이다.   
여기서  `ON_BN_CLICKED` 라는 것을 따라가보면   
```cpp
#define ON_BN_CLICKED(id, memberFxn) \
	ON_CONTROL(BN_CLICKED, id, memberFxn)

#define ON_CONTROL(wNotifyCode, id, memberFxn) \
	{ WM_COMMAND, (WORD)wNotifyCode, (WORD)id, (WORD)id, AfxSigCmd_v, \
		(static_cast< AFX_PMSG > (memberFxn)) },
```
이런 매크로의 형태로 정의되어있고 이어서 `ON_CONTROL` 까지 확인 할 수 있다.   
마지막으로 `WM_COMMAND` 는 특정한 값으로 정의되어있는것을 볼 수 있다.    
[MSDN](https://docs.microsoft.com/en-us/windows/win32/menurc/wm-command)에 `WM_COMMAND`를 찾아보면 컨트롤이 부모창에 알림 `메세지`를 보낼 때 전송된다고 한다. 


</br>

## Message Queue
위에서 확인한 것 처럼 MFC에서는 메세지를 통해 동작을 처리 하고 있을을 볼 수 있다.   
버튼클릭이 아니더라도 화면을 그리기위한 WM_PAINT, WM_ERASEBKGND 창 크기가 변경될 때의 WM_SIZE등 수많은 메세지가 있다. 이 메세지들은 어떻게 처리 될까?

</br>

MFC는 Win32에 대한 wrapper 클래스이다. 당신이 Visual Studio에서 MFC 데스크톱 어플리케이션을 선택하여 새로운 프로젝트를 생성 하게되면 내부에는 Win32가 포함되어 있다는 뜻이다. 
```cpp
while( (bRet = GetMessage( &msg, NULL, 0, 0 )) != 0)
{ 
    if (bRet == -1)
    {
        // handle the error and possibly exit
    }
    else
    {
        TranslateMessage(&msg); 
        DispatchMessage(&msg); 
    }
}
```   
그리고 Win32에는 반드시 위와 같은 Message loop가 포함되어있다.   
메세지 루프는 `Message Queue`로 부터 message를 가져와 변환 작업을 수행하고 적절한 window procedure에게 보내주는 동작을 수행한다.


</br>


# Multi-thread  

우리는 위에서 Message Queue와 Message loop를 확인했다.   
다들 알다시피 Queue라는것은 FIFO(First in first out)의 특성을 가지고 있다. 한번에 하나만 처리 할 수 있다.    
여러개의 스레드에서 GUI Control에 접근하게되면 Message Queue에는 처리되지못한 메세지들이 쌓이게 될 것이다.      
쌓이고 쌓이면 어플리케이션은 먹통이 되거나 뻗어버릴 것이다.    
그러면 어떻게 해야 이런 현상을 피할 수 있을까?   

</br>

## 여러개의 스레드에서 GUI에 접근하는 것을 피하자.   
기본적이면서 효과적인 방법이다. 쉽게 접할 수 있는 게임을 예로 들어보자.    
게임에는 FPS라는것이 있다. Frame per second 1초에 화면이 몇번 그려지는지를 나타내는 수치다. 최근에는 60fps를 넘어 144, 200 까지도 지원하는 경우가 있다.   
저사양게임은 그렇다고 치고, 고사양 게임들은 어떻게 가능할까?    
</br>
화면을 그리기위한 스레드(Display Thread)와 화면을 준비하는 스레드(Worker Thread)가 분리되어있기 때문이다.    
Display Thread는 말 그대로 화면을 그리는 단 하나의 동작만 수행한다. 그리고 Worker Thread에서는 표시 할 화면에 대한 여러가지 물리적 연산이나 서버와의 통신, 여러가지 처리 결과를 바탕으로 메모리에 화면을 준비하는 동작까지 말 그대로 화면을 준비한다.    
단 Worker Thread는 개발하기에 따라 여러개의 스레드가 될 수 있고 하나의 스레드가 될 수도 있다.   

</br>


## 동기화
mutex가 어쩌고 저쩌고

</br>

## 병목
락이 걸려서 어쩌고 저쩌고



