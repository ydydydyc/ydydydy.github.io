---
title: "Android Context란 무엇인가"
date: 2021-01-16
categories: [Android framework]
tags: [Android, Android framework, Context]
comments: true
---

Android application을 개발하면서 Context를 모를수는 없다.  
하지만 누군가가 나한테 Context가 뭐야? 라고 물어보면 아 그거 있잖아...그거.. 정도로밖에 설명이 안된다.  

작년 Android 11 포팅 작업을 하면서 GUI 이슈가 하나 있었는데 정말 이유를 알 수 없었다.  
난 분명히 API 가이드에 맞춰 사용했고(진짜 하다 못해 함수 라인 수 까지 맞춤) API 제작자도 도무지 원인을 알 수 없다면서 시간을 좀 달라고 했던 일이 있었다.  
두 명이서 거의 일주일을 헤맸고 결론은 **내가 Context를 잘못 사용하고 있었던 것**.  
그런데 그 때도 사실 잘 이해가 가지 않았어서 이 참에 Context에 대한 정리를 다시 할 필요가 있겠다 싶었다.


## Context란
***
(대다수 개발자들이 그렇겠지만) 구글은 절대로 이름을 대충 짓지 않는다.  
>context  
미국[│kɑːntekst]  
영국식[│kɒntekst]  
명사  
1 (어떤 일의) 맥락, 전후 사정  
2 (글의) 맥락, 문맥

Context는 맥락이다.

개발자 사이트를 가보자.
>Interface to global information about an application environment. This is an abstract class whose implementation is provided by the Android system. It allows access to application-specific resources and classes, as well as up-calls for application-level operations such as launching activities, broadcasting and receiving intents, etc.  

~~발번역~~  
애플리케이션 환경에 대한 글로벌 정보(요것이 맥락인가)를 위한 인터페이스. 
***안드로이드 시스템에서 구현***을 제공하는 추상 클래스이며 애플리케이션의 특정 리소스와 클래스에 접근가능하도록 해주며,  
액티비티 런칭, 브로드캐스팅, 인텐트 수신 등의 애플리케이션 레벨에 대한 호출도 가능하다.

[Stack Overflow](https://stackoverflow.com/questions/3572463/what-is-context-on-android) 에 괜찮은 비유가 있어서 발췌해봤다.
>It's like access to android activity to the app's resource.  
It's similar to when you visit a hotel, you want breakfast, lunch & dinner in the suitable timings, right?  
There are many other things you like during the time of stay. How do you get these things?  
You ask the room-service person to bring these things for you.  
Here the room-service person is the context considering you are the single activity and the hotel to be your app, finally the breakfast, lunch & dinner has to be the resources.  

어플리케이션 - 호텔  
룸 서비스 담당자 - Context  
리소스 - 룸 서비스  
로 비유 할 수 있다는 것이다.   
사실 정의는 간단하다. 애플리케이션의 현재 상태를 나타내며 리소스, 데이터베이스 등에 액세스 하는 곳에 사용할 수 있는 추상 클래스.  

**진짜 문제는 내가 Context를 가져올 때 정확히 어떻게 가져와야하는가?** 가 아닐까.  
getContext(), getApplicationContext(), getBaseContext(), 가끔은 그냥 getActivity(), this로 사용하기까지.  

<br>


## Application Context
***
### getApplicationContext()
>Return the context of the single, global Application object of the current process.  
This generally should only be used if you need a Context whose lifecycle is separate from the current context, that is tied to the lifetime of the process rather than the current component.  

Singleton이고 Activity를 통해 접근할 수 있다.  
이렇게 가져온 context는 애플리케이션 Lifecycle과 연결되어 있고 애플리케이션이 살아있는 동안은 항상 동일한 context이다.  
lifecycle이 현재 context와 분리된 context를 필요로 할 경우 또는 Activity의 범위를 벗어나는 영역에 context를 전달해야할 때 사용된다.  
 
<br>

## Activity Context
***
Activity Context는 말 그대로 Activity에서 사용할 수 있고 Activity의 lifecycle과 연관되어있다.  
MyApplication에 MyActivity1, MyActiviy2가 있다고 가정하면  
1. MyApplication은 Application context를 가지고 있다.
2. MyAcitivity1는 Application context, MyActivity1 context 둘 다 가지고있다. 
3. MyAcitivity2는 Application context, MyActivity2 context 둘 다 가지고있다.
<br>

### View.getConext()
ContentProvider(이것도 추후에 포스팅해야지) 가 실행중인 Context를 가져오며, **onCreate()가 호출 된 후에만 사용할 수 있다.**  
Activity가 Destroy되면 소멸되며 생성자에서는 null을 리턴한다.

### ContextWrapper.getBaseContext()
ContextWrapper(요것도 추후에...)의 메소드. 자신의 context가 아닌 다른 Context에 접근하려고 할 때 사용한다. 

**[Quiz1]**  
MyApplication에 **singleton**인 A, B 클래스가 있다.  
이 때 A, B 클래스에는 어떤 context를 전달해야할까요  
답 : `Application context`  
만약 MyActivity1 같은 activity context를 전달하게되면 MyActivity1가 destroy 되었을 때도 A, B가 계속 참조를 하게 되기 때문에 Memory leak을 발생시킨다.  
<br>

Activity에서는 항상 Activity context를 사용한다.  
UI component, 토스트, 다이얼로그 등등 UI 작업에는 항상 Activity Context를 사용해야한다.
**Application context로 GUI 작업을 하게 될 경우 의도한대로 동작하지 않을 가능성이 높다.**  
-> 서두에 언급했던 context 문제가 이것이었다. getApplicationContext()는 사실 우리 앱에서 거의 안쓰는데.. 개발할 때 뭐에 홀렸었는지 context를 application context로 받아오고 있었다.  
Activity context로 변경했더니 정말 깔끔하게 문제가 해결됐었다.

Android developer 사이트에서도 몇가지 예시를 들어놨다.
>Consider for example how this interacts with registerReceiver(android.content.BroadcastReceiver, android.content.IntentFilter):  
<br>
If used from an Activity context, the receiver is being registered within that activity. This means that you are expected to unregister before the activity is done being destroyed; in fact if you do not do so, the framework will clean up your leaked registration as it removes the activity and log an error. Thus, if you use the Activity context to register a receiver that is static (global to the process, not associated with an Activity instance) then that registration will be removed on you at whatever point the activity you used is destroyed.  
<br>
If used from the Context returned here, the receiver is being registered with the global state associated with your application. Thus it will never be unregistered for you. This is necessary if the receiver is associated with static data, not a particular component. However using the ApplicationContext elsewhere can easily lead to serious leaks if you forget to unregister, unbind, etc.  

1. registerReceiver를 activity context로 등록했기 때문에 activity가 destroy 되기 전에 unregi를 해야한다.
2. Application Context를 사용할 경우 unregi 등을 제대로 하지 않을 경우 Memory leak이 발생하게 된다.


<br>
<br>
완벽히 이해했냐고 물어보면 아니요지만 적어도 이제 잘못 쓰는 경우는 없을 것 같다.
<br>
<br>

오류 지적은 항상 환영합니다.

참조  
<https://blog.mindorks.com/understanding-context-in-android-application-330913e32514>  
<https://developer.android.com/>  
<https://stackoverflow.com/questions/3572463/what-is-context-on-android>  
