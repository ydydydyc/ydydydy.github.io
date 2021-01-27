---
title: "Java Reflection의 이해"
date: 2021-01-25
categories: [Java]
tags: [Java, Refelction]
comments: true
---

***작성중***

이전 포스팅에서 장황하게 @hide, @SystemApi의 설명을 한 이유는 이 Refelction을 위해서였다.  
우리 팀은 종종 Reflection을 통해서 API를 호출하는데, 사실 자주 쓰면서도 정확하게 이해하고 쓰지는 않았던 것 같다.  

이전 포스팅에서 테스트했었던 Reflection의 예시다.
```java
        WifiManager wifiManager = (WifiManager) this.getSystemService(Context.WIFI_SERVICE);

        try {
            Method method = wifiManager.getClass().getMethod("isConnectedMacRandomizationSupported");
            boolean result = (Boolean) method.invoke(wifiManager);
            Log.d(TAG, "Is supporting random MAC? " + result);
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
```
이 예제는 getClass()를 통해 WifiManager의 클래스의 정보를 가져오고, getMethod()를 통해 해당 클래스에 정의되어있는 메소드 중 파라미터로 전달된 이름을 가진 메소드를 반환한다.  
그리고 invoke

Reflection은 자바 런타임에 class name을 가져오고, object를 만들 수 있는 방법이다.

[Oracle Java docs](https://docs.oracle.com/javase/tutorial/reflect/index.html) 의 튜토리얼에서는 Reflection을 아래와 같이 설명한다.  
> Reflection is commonly used by programs which require the ability to examine or modify the runtime behavior of applications running in the Java virtual machine. This is a relatively advanced feature and should be used only by developers who have a strong grasp of the fundamentals of the language. With that caveat in mind, reflection is a powerful technique and can enable applications to perform operations which would otherwise be impossible.

Java virtual machine에서 수행되는 애플리케이션의 **런타임** 동작을 검사하거나 수정하는 기능이고 고급 기능에 속한다.  
실제로 Refelction은 굉장히 느리기 때문에, 너무 남용할 경우 시스템 성능의 저하를 발생시킨다.  




### 단점
위에도 언급했듯이, Reflection은 강력한 기능이지만, 남발해서는 안된다.  
1. Performance overhead  
Reflection은 동적이다. 따라서 JVM optimization을 사용할 수 없다. 결과적으로 direct call보다 느리고, 빠른 성능이 요구되는 애플리케이션이라면 Refelction의 사용을 피해야한다.

2. Security Restrictions  
런타임 퍼미션이 필요한 경우가 있다. 
   
3. Exposure of Internals  
@hide, @SystemApi의 경우이다. 비공개 메소드나 필드에 접근하는 방식이기 때문에 결론적으로는 illegal한 방법이고, 이는 예기치 못한 부작용을 낳을 수 있다. 그리고 객체지향의 철학을 깨트리는 방법이기 때문에 업그레이드 등의 상황에서 정상동작을 보장할 수 없으며 아주 신중하게 사용해야한다.
   
이론은 이러하지만,, 개발을 하다보면 어쩔 수 없이 reflection을 최후의 수단으로 사용하게 되는 경우가 있다. 