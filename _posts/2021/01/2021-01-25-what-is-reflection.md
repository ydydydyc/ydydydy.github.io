---
title: "Java Reflection의 이해"
date: 2021-01-25
categories: [Java]
tags: [Java, Reflection]
comments: true
---

이전 포스팅에서 장황하게 @hide, @SystemApi의 설명을 한 이유는 이 Reflection을 위해서였다.  
우리 팀은 종종 Reflection을 통해서 API를 호출하는데, 사실 자주 쓰면서도 정확하게 이해하고 쓰지는 않았던 것 같다.  

Reflection은 자바 런타임에 class name을 가져오고, object를 만들 수 있는 방법이다.  

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
컴파일된 .class를 ClassLoader를 통해 JVM에 로드하고 Method area에 저장하며, 여기에 접근할 수 있도록 java.lang.reflect를 사용한다.  
이 부분의 자세한 설명은 [여기](https://www.holaxprogramming.com/2013/07/16/java-jvm-runtime-data-area/) 참조  

[Oracle Java docs](https://docs.oracle.com/javase/tutorial/reflect/index.html) 의 튜토리얼에서는 Reflection을 아래와 같이 설명한다.  
> Reflection is commonly used by programs which require the ability to examine or modify the runtime behavior of applications running in the Java virtual machine. This is a relatively advanced feature and should be used only by developers who have a strong grasp of the fundamentals of the language. With that caveat in mind, reflection is a powerful technique and can enable applications to perform operations which would otherwise be impossible.

Java virtual machine에서 수행되는 애플리케이션의 **런타임** 동작을 검사하거나 수정하는 기능이고 고급 기능에 속한다.  
실제로 Refelction은 굉장히 느리기 때문에, 너무 남용할 경우 시스템 성능의 저하를 발생시킨다.  

### getMethods, getMethod
클래스가 가지고 있는 public 메소드 각각에 해당하는 Method 객체의 배열을 반환
```java
Method[] methods = wifiManager.getClass().getMethods();
for(Method method : methods) {
   Log.d(TAG, method.getName());
}
```
이렇게 호출했을 때  
```
 01-27 18:24:21.242 D 10290 11791 11791 RandomMacTest: addNetwork
 01-27 18:24:21.243 D 10290 11791 11791 RandomMacTest: addNetworkSuggestions
 01-27 18:24:21.243 D 10290 11791 11791 RandomMacTest: addOnWifiUsabilityStatsListener
 01-27 18:24:21.243 D 10290 11791 11791 RandomMacTest: addOrUpdatePasspointConfiguration
 01-27 18:24:21.243 D 10290 11791 11791 RandomMacTest: addSuggestionConnectionStatusListener
 01-27 18:24:21.243 D 10290 11791 11791 RandomMacTest: allowAutojoin
 01-27 18:24:21.243 D 10290 11791 11791 RandomMacTest: allowAutojoinGlobal
 01-27 18:24:21.243 D 10290 11791 11791 RandomMacTest: allowAutojoinPasspoint
 ...
```
주루룩 나온다.  
첫 예시처럼 어떤 특정한 메소드를 호출하고 싶다면 예제처럼 직접 호출하면 되고, 만약 파라미터가 필요하다면
```java
        WifiManager wifiManager = (WifiManager) this.getSystemService(Context.WIFI_SERVICE);
        WifiConfiguration config = new WifiConfiguration();
        try {
            Method method = wifiManager.getClass().getMethod("updateNetwork", new Class[]{WifiConfiguration.class});
            method.invoke(wifiManager, new Object[]{config});
            Log.d(TAG, "Call updateNetwork");
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
```
메소드 객체와, 인자로 넘겨줄 배열을 만들면 된다. 만약에 이 함수의 파라미터가 하나 더 있다면 new Class 배열에 추가로 넣어주면 된다.  
물론 이 예시는 함수 콜은 되지만 정상적으로 동작하지는 않는다. WifiConfiguration이 null일 경우 종료하도록 되어있다.

### getFields, getField
메소드와 똑같다.
```java
WifiManager wifiManager = (WifiManager) this.getSystemService(Context.WIFI_SERVICE);
Field[] fields = wifiManager.getClass().getFields();
for(Field field : fields) {
   Log.d(TAG, field.getName());
}
```
해당 클래스에 있는 인자들의 배열을 리턴한다.  
```
 01-27 18:56:45.618 D 10290 31387 31387 RandomMacTest: ACTION_LINK_CONFIGURATION_CHANGED
 01-27 18:56:45.619 D 10290 31387 31387 RandomMacTest: ACTION_NETWORK_SETTINGS_RESET
 01-27 18:56:45.619 D 10290 31387 31387 RandomMacTest: ACTION_PASSPOINT_LAUNCH_OSU_VIEW
 01-27 18:56:45.619 D 10290 31387 31387 RandomMacTest: ACTION_PICK_WIFI_NETWORK
 01-27 18:56:45.619 D 10290 31387 31387 RandomMacTest: ACTION_REQUEST_DISABLE
 01-27 18:56:45.619 D 10290 31387 31387 RandomMacTest: ACTION_REQUEST_ENABLE
 01-27 18:56:45.619 D 10290 31387 31387 RandomMacTest: ACTION_REQUEST_SCAN_ALWAYS_AVAILABLE
 01-27 18:56:45.619 D 10290 31387 31387 RandomMacTest: ACTION_WIFI_NETWORK_SUGGESTION_POST_CONNECTION
 01-27 18:56:45.619 D 10290 31387 31387 RandomMacTest: ACTION_WIFI_SCAN_AVAILABILITY_CHANGED
 ...
```
특정 필드를 가져오거나, 변경하고싶다면 get, set을 하면 된다.  
Wi-Fi network의 auto reconnect 여부를 결정하는 변수 allowAutojoin(default true다.)를 가져오고, 변경해 보았다.  
```java
WifiConfiguration config = new WifiConfiguration();
config.networkId = 100;
config.SSID = "Test";

try {
   Field field = config.getClass().getField("allowAutojoin");
   Log.d(TAG, "Before : " + config.SSID + " " + field.get(config));
   field.set(config, false);
   Log.d(TAG, "After : " + config.SSID + " " + field.get(config));
} catch (NoSuchFieldException | IllegalAccessException e) {
    e.printStackTrace();
}
```
임의의 WifiConfiguration을 만들고 대충 network ID랑 SSID를 넣어준 뒤  
메소드와 동일하게 호출하였다.  
```
 01-27 19:12:45.254 D 10290 2121 2121 RandomMacTest: Before : Test true
 01-27 19:12:45.254 D 10290 2121 2121 RandomMacTest: After : Test false
```
디폴트값으로 set 되었다가 변경 후 false로 잘 적용되었다.  
앞으로 이 네트워크는 사용자가 수동으로 연결하기 전까지는 자동으로 연결되지 않을 것이다.  
        

### getDeclaredMethod, getDeclaredField
위 예제들은 모두 public 멤버들이었다.  
만약 호출하려는 메소드나 필드가 private이라면 어떻게 해야할까?
```java
WifiManager wifiManager = (WifiManager) this.getSystemService(Context.WIFI_SERVICE);

try {
   Field field = wifiManager.getClass().getDeclaredField("mVerboseLoggingEnabled");
   field.setAccessible(true);
   Log.d(TAG, "Verbose logging : " + field.get(wifiManager));
} catch (NoSuchFieldException | IllegalAccessException e) {
    e.printStackTrace();
}
```
getDeclaredField로 호출해주고, setAccessible를 true로 set하면 된다.  
하지만 이 call은 불가능하다. 왜냐면 mVerboseLoggingEnabled는 hidden이기 때문에...  
WifiManger와 WifiConfiguration에서 prviate이며, SystemApi인 멤버를 찾지 못했다.  
하지만 private에 직접 접근하려는 사람이 있을까...?  
     


### 단점
위에도 언급했듯이, Reflection은 강력한 기능이지만, 남발해서는 안된다.  
1. Performance overhead  
Reflection은 동적이다. 따라서 JVM optimization을 사용할 수 없다. 결과적으로 direct call보다 느리고, 빠른 성능이 요구되는 애플리케이션이라면 Refelction의 사용을 피해야한다.  
Direct call / Binder call / Reflection call로 동일한 메소드를 호출했을 때,  
속도는 Direct >>>>>> Reflection call > Binder call 순으로 빨랐다. Binder call보다는 일반적으로 빠른 듯 하다.  
여튼, Direct call과는 비교할 수 없을 정도로 느리다.  

2. Security Restrictions  
런타임 퍼미션이 필요한 경우가 있다. 
   
3. Exposure of Internals  
@hide, @SystemApi가 한 경우이다. 비공개 메소드나 필드에 접근하는 방식이기 때문에 결론적으로는 illegal한 방법이고, 이는 예기치 못한 부작용을 낳을 수 있다. 그리고 객체지향의 철학을 깨트리는 방법이기 때문에 업그레이드 등의 상황에서 정상동작을 보장할 수 없으며 아주 신중하게 사용해야한다.
   
이론은 이러하지만,, 개발을 하다보면 어쩔 수 없이 reflection을 최후의 수단으로 사용하게 되는 경우가 있다. 
쓰되 알고 쓰자!



참조 : https://www.hanbit.co.kr/network/category/category_view.html?cms_code=CMS8811889310&cate_cd=

오류 지적은 언제나 환영입니다.


