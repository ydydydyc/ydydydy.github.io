---
title: "Factory Method Pattern"
date: 2021-04-01
categories: [Design-pattern]
tags: [Java, Design pattern, Factory method pattern]
comments: true
---

**Java 언어로 배우는 디자인 패턴 입문을 공부하며 정리한 내용입니다.**
<br>

인스턴스를 생성하는 공장을 Template method pattern으로 구성한 것이 **Factory method pattern**이다.  
Factory method patter은 인스턴스를 만드는 방법을 상위 클래스에서 결정하지만 구체적은 내용은 모두 하위 클래스에서 수행한다. 

![factory](https://user-images.githubusercontent.com/77476913/113308707-66b18780-9341-11eb-9c01-4e07537f97d7.PNG)

Creator의 구체적인 내용은 하위클래스인 ConcreteCreator가 결정한다.  
new를 이용하여 실제의 인스턴스를 생성하는 대신에 인스턴스 생성을 위한 메소드를 호출해서 상위 클래스를 자유롭게 만든다.
이 때 Creator가 Factory가 된다.

### 인스턴스의 생성
예제에서 factoryMethod는 추상메소드이며 하위클래스에서 구현하게 된다.  
이런 factoryMethod의 기술 방법은 아래 세가지가 있다.

#### 추상메소드
예제와 같은 방법이다.
```java
public abstract class AccessPointFactory {
    public abstract AccessPoint createAccessPoint(String ssid);
}
```
하위 클래스는 반드시 이 메소드를 구현해야한다.

#### 디폴트 구현을 준비해 둔다.
디폴트를 준비해두고 하위클래스에서 구현하지 않았을 때 사용하면 된다.
```java
public class AccessPointFactory {
    public AccessPoint createAccessPoint(String ssid) {
        return new AccessPoint(ssid);
    }
}
```

##### 에러를 이용한다.
```java
public class AccessPointFactory {
    public AccessPoint createAccessPoint(String ssid) {
        throw new FactoryMethodRuntimeException();
    }
}
```

<br>

### WifiTracker
지금은 deprecated 되었지만 WifiSettings에서 WifiTracker를 생성할 때 이 Factory method pattern의 디폴트 구현이 적용되어 있다.  
WifiSettings에서 WifiTracker의 생성자를 이용하여 직접 객체를 생성하지 않고 이렇게 WifiTrackerFactory의 factory method를 이용하여 객체를 생성한다.   
[WifiSettings.java](https://android.googlesource.com/platform/packages/apps/Settings/+/refs/heads/android10-release/src/com/android/settings/wifi/WifiSettings.java)
```java
@Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        mWifiTracker = WifiTrackerFactory.create(
                getActivity(), this, getSettingsLifecycle(), true, true);
        mWifiManager = mWifiTracker.getManager();
```
[WifiTrackerFactory.java](https://android.googlesource.com/platform/frameworks/base/+/master/packages/SettingsLib/src/com/android/settingslib/wifi/WifiTrackerFactory.java) 에서 create를 통해 WifiTracker(Product 역할을 하는)를 리턴해준다.  
```java

/**
 * Factory method used to inject WifiTracker instances.
 */
public class WifiTrackerFactory {
    private static WifiTracker sTestingWifiTracker;
    @Keep // Keep proguard from stripping this method since it is only used in tests
    public static void setTestingWifiTracker(WifiTracker tracker) {
        sTestingWifiTracker = tracker;
    }
    public static WifiTracker create(
            Context context, WifiTracker.WifiListener wifiListener, @NonNull Lifecycle lifecycle,
            boolean includeSaved, boolean includeScans) {
        if(sTestingWifiTracker != null) {
            return sTestingWifiTracker;
        }
        return new WifiTracker(context, wifiListener, lifecycle, includeSaved, includeScans);
    }
}
```
[WifiTracker.java](https://android.googlesource.com/platform/frameworks/base/+/master/packages/SettingsLib/src/com/android/settingslib/wifi/WifiTracker.java) 구현
```java
// TODO(sghuman): Clean up includeSaved and includeScans from all constructors and linked
    // calling apps once IC window is complete
    public WifiTracker(Context context, WifiListener wifiListener,
            @NonNull Lifecycle lifecycle, boolean includeSaved, boolean includeScans) {
        this(context, wifiListener,
                context.getSystemService(WifiManager.class),
                context.getSystemService(ConnectivityManager.class),
                context.getSystemService(NetworkScoreManager.class),
                newIntentFilter());
        lifecycle.addObserver(this);
    }
```
물론 더 WifiTracker 자체가 deprecate 되어 더 이상 볼 일은 없지만 Template method pattern과 관련된 패턴이 많은 만큼 익혀두도록~!

<br>
<br>



오류 지적은 언제나 환영입니다.  