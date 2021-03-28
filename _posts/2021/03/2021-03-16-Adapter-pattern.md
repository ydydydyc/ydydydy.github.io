---
title: "Adapter Pattern"
date: 2021-03-28
categories: [Design-pattern]
tags: [Java, Design pattern, Adpater]
comments: true
---

**Java 언어로 배우는 디자인 패턴 입문을 공부하며 정리한 내용입니다.**
<br>

굉장히 오랜만에 올리는 포스팅  
공부 좀 소홀하게 한 것도 있고^_ㅠ 앞 내용들이 좀 당연한 내용들이라 포스팅 하기에 적합하지 않았다.  
여튼 디자인 패턴의 첫 포스팅은 **Adpater pattern**  

> 이미 제공되어 있는 것을 그대로 사용할 수 없을 때, 필요한 형태로 교환하고 사용하는 것.  
> '이미 제공되어 있는 것'과 '필요한 것' 사이의 **'차이'** 를 없애주는 디자인 패턴

Adapter patter은 wrapper라고 불리기도 한다.  
가장 적합한 예가 '전기'이다. 
직류 12볼트로 작동하는 노트북을 교류 100볼트의 AC 전원에 연결한다고 가정 할 때, 우리는 AC 어댑터라는 장치를 사용한다.  
AC 어댑터는 가정용 전원으로 제공되고 있는 교류 100볼트를 지금 필요한 직류 12볼트로 교환해 준다.  
**제공되고 있는 것과 필요한 것 사이를 연결**해주는 것이 어댑터의 역할이다. 
<br>
<br>

### Class에 의한 Adpater 패턴(상속을 사용한 패턴)  
![Adpater](https://user-images.githubusercontent.com/77476913/112746178-45bb0080-8fe8-11eb-86fc-9ad2e79e947c.PNG)  
제공되어있는 Adpatee class를 상속하여 Apdater 역할을 하는 Adpater class가 있다.  
그리고 Target interface(필요한 것)를 구현한다.  
**Main에서는 Adapter class를 사용해 Adapter의 인스턴스를 Target 인터페이스의 변수로 대입한다.** 
Adaptee의 코드는 Main 안에서 확인할 수 없으며 Adapter가 어떻게 동작하고 있는지는 알수 없고 알 필요도 없다.  
Main 클래스를 전혀 변형하지 않고 Adapter 클래스의 구현을 바꿀 수 있다.  
<br>
<br>

### Instance에 의한 Adpater 패턴(위임을 사용한 패턴)
Java에서 말하는 위임은 어떤 메소드의 실제 처리를 다른 인스턴스의 메소드에 맡기는 것을 뜻한다.  
위의 패턴과 달리지는 것은 아래와 같다.
1. Target은 인터페이스가 아니라 클래스가 된다.  
2. Adapter는 Adaptee 필드에서 Adaptee 클래스의 인스턴스를 가진다. (Adapter의 생성자에서)
3. 이전 예에서는 자신의 상위 클래스에서 상속한 메소드를 호출하지만 이 경우에는 필드를 경유해서 호출한다.(위임)  

![Adapater2](https://user-images.githubusercontent.com/77476913/112746636-0a6e0100-8feb-11eb-98d5-fc2ed36755fb.PNG)  
<br>
<br>

이미 만들어진 클래스를 새로운 인터페이스(API)에 맞게 개조시킬 때는 Adpater 패턴을 사용해야한다.  
새로운 인터페이스에 맞게 개조시킬 때 기존 클래스의 소스를 바꿔서 수정하려고 하나 이는 검증된 TC의 수정이 필요하게 된다.  
이 때 Adpater만 수정하면 TC 수정 없이 버전 업이 가능하다.  
<br>
하지만 Adpatee와 Target의 역할이 지나치게 동떨어진 경우에는 Adpater 패턴을 사용할 수 없다.  
마치 블루투스 샤워기처럼...

