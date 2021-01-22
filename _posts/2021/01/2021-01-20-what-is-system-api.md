---
title: "@Systemapi, @hide 의 이해"
date: 2021-01-23
categories: [Android framework]
tags: [Android, Android framework, systemapi]
comments: true
---

**작성중**

안드로이드의 버전이 높아질수록 Google에 OEM이나 3rd party app에 대해 제한하는 것들의 범위가 점점 넓어지고 있다.  
preload app인데도 불구하고 framework의 api를 사용하지 못하는 것 뿐만 아니라, 나아가서는 framework를 updatable하게 만들기 위해 mainline을 적용하며 OEM에서 아예 소스코드를 수정하지 못하게 만드는 등 제한의 폭은 다양해지고 있다.

내가 담당하는 모듈이 이번 Android 11부터 mainlne의 후보가 되어 이번 포팅을 진행하며 쓰지 못했던 api가 종종 있었다.  
([Mainline과 apex](https://android-developers.googleblog.com/2019/05/fresher-os-with-projects-treble-and-mainline.html) 에 대한 설명은 링크를 참조)
<br>
<br>

그러면 왜 안드로이드에 인스톨 되는 apk중에 가장 막강한 권한을 가진 앱인 Settings도 쓰지못하는 api가 있었는가?  
<br>


### 비SKD 인터페이스 제한사항
Android 9부터 적용된 사항으로 앱에서 사용할 수 있는 비SDK 인터페이스가 제한되었다.
[Developer site](https://developer.android.com/distribute/best-practices/develop/restrictions-non-sdk-interfaces?hl=ko) 참고  
간단히 말하면 [여기](https://developer.android.com/reference/packages?hl=ko) 에 공식적으로 공개되지 않은 SDK는 쓰지 못한다는 것이다. 따라서 reflection의 사용도 금지된다.



[WifiManager.java](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/android11-release/wifi/java/android/net/wifi/WifiManager.java) 의 소스코드를 일부 가져와봤다.  
isConnectedMacRandomizationSupported()이라는, Connected 상태에서의 Random MAC을 지원하는지 여부를 확인하는 API이다.
```java
/**
 * @return true if this device supports connected MAC randomization.
 * @hide
 */
@SystemApi
public boolean isConnectedMacRandomizationSupported() {
  return isFeatureSupported(WIFI_FEATURE_CONNECTED_RAND_MAC);
  }
 ```
메소드 위에는 annotaion들이 붙어있다.  
그리고 실제로 호출해보았다.
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        var wifiManger : WifiManager = getSystemService(Context.WIFI_SERVICE) as WifiManager
        wifiManger.isConnectedMacRandomizationSupported()
    }
```

```kotlin
Unresolved reference: isConnectedMacRandomizationSupported
```
isConnectedMacRandomizationSupported 없는 함수라고 표시된다.
있는데?  
그럼 @hide, @SystemApi, @RequiresPermission 
각각은 무엇을 의미하는걸까?


### @hide
>When applied to a package, class, method or field, @hide removes that node and all of its children from the documentation.

@hide는 javadoc의 일부이며 pulbic API가 아니기 때문에 사용하면 안됨을 의미한다.
@hide를 사용하면 SDK 빌드에 포함되지 않기 때문에 document에는 아예 없는 API가 되는 것이다.  
하지만 javadoc의 일부이기 때문에 Platform 내에서 사용 시에는 문제가 없으나 이 역시 Mainline의 영역에 있는 API는 접근 불가능하다.  

 
### @SystemApi
실제 존재하는 API임은 맞지만, 공개되지는 않은 API이다.
SDK 빌드 시 포함되지만 사용할 수 없고, 사용하고자 한다면 Reflection을 이용해야한다.  
빌드 시 framework와 함께 빌드되는 앱(예를 들면 셋팅)은 빌드 타임에 해당 api를 참조하기 때문에 빌드도 되고, 사용도 가능하다.  
따라서 @hide API를 쓰고싶으면 @SystemApi를 추가로 붙여주면 사용은 가능하지만 이 역시 Mainline 영역일 경우 빌드 자체가 되지 않고, 이 부분은 OEM만 가능한 부분이다.  
실제로 isConnectedMacRandomizationSupported를 reflection으로 호출해보았다. (편의상 여기는 그냥 자바로..)  

```java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "RandomMacTest";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

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
    }
}
```
```
 01-22 17:17:55.171 D 10305 19954 19954 RandomMacTest: Is supporting random MAC? false
```

실제로 함수가 정상적으로 invoke 되었고 해당 값도 잘 리턴 되었다.  
isConnectedMacRandomizationSupported의 경우 요구되는 permission이 "android.permission.ACCESS_WIFI_STATE" 뿐이기 때문에 3rd party app에서도 우회해서 사용할 수 있지만, 실제로 대부분의 함수들을 사용하기는 힘들다.  
예를 들어 저장된 모든 Wi-Fi configuration을 삭제하는 factoryRest()을 호출해본다고 가정했을 때,  

```
 01-22 17:25:15.928 (+0.001s) W 10305 20682 20682 System.err: java.lang.reflect.InvocationTargetException
 01-22 17:25:15.928 W 10305 20682 20682 System.err: at java.lang.reflect.Method.invoke(Native Method)
 01-22 17:25:15.928 W 10305 20682 20682 System.err: at com.example.myapplication.MainActivity.onCreate(MainActivity.java:34)
...
 01-22 17:25:15.931 W 10305 20682 20682 System.err: Caused by: java.lang.SecurityException: WifiService: Neither user 10305 nor current process has android.permission.NETWORK_SETTINGS.
...
 01-22 17:25:15.932 W 10305 20682 20682 System.err: at android.net.wifi.IWifiManager$Stub$Proxy.factoryReset(IWifiManager.java:3651)
 01-22 17:25:15.932 W 10305 20682 20682 System.err: at android.net.wifi.WifiManager.factoryReset(WifiManager.java:5525)
 01-22 17:25:15.932 W 10305 20682 20682 System.err: ... 17 more
```

"android.permission.NETWORK_SETTINGS"이 필요하다는 메시지와 함께 InvocationTargetException으로 정상적인 호출이 되지 않는다.  

```java
    /**
     * Removes all saved Wi-Fi networks, Passpoint configurations, ephemeral networks, Network
     * Requests, and Network Suggestions.
     *
     * @hide
     */
    @SystemApi
    @RequiresPermission(android.Manifest.permission.NETWORK_SETTINGS)
    public void factoryReset() {
        try {
            mService.factoryReset(mContext.getOpPackageName());
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }
```

@RequiresPermission으로 해당 퍼미션이 필요함을 명시하고 있다.  
그러면 저 퍼미션을 추가하면 될까?  
답은 안된다.

```
Permission is only granted to system apps 
```
system app만 이 퍼미션을 획득할 수 있다.  
위에 언급했던 AOSP 소스를 확인해보면 대부분의 API에 RequiresPermission annotation이 달려있고, 대부분의 퍼미션들은 system apps로 제한되어있다.  
참고로 system app에서 factoryReset을 reflection으로 호출해보면 아주 잘 동작한다.  

이 정도면 난 이해 된 것 같다.  

