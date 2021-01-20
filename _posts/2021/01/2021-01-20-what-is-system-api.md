---
title: "@Systemapi, @hide 의 이해"
date: 2021-01-23
categories: [Android framework]
tags: [Android, Android framework, systemapi]
comments: true
---

**작성중**

안드로이드의 버전이 높아질수록 Google에 OEM이나 3rd party app에 대해 제한하는 것들의 범위가 점점 넓어지고 있다.  
preload app인데도 불구하고 framework의 api를 사용하지 못하는 것 뿐만 아니라, 나아가서는 framework를 updatable하게 만들기 위해 mainline을 적용하며 OEM에서 아예 소스코드를 수정하지 못하게 만드는 것 등등 제한의 폭은 다양해지고 있다.

내가 담당하는 모듈이 이번 Android 11부터 mainlne의 후보가 되어 이번 포팅을 진행하며 쓰지 못했던 api가 종종 있었다.  
([Mainline과 apex](https://android-developers.googleblog.com/2019/05/fresher-os-with-projects-treble-and-mainline.html) 에 대한 설명은 링크를 참조)
<br>
<br>

그러면 왜 나는 안드로이드에 인스톨 되는 apk중에 가장 막강한 권한을 가진 앱인 Settings를 개발하면서도 쓰지못하는 api가 있었는가?  
<br>


### 비SKD 인터페이스 제한사항
Android 9부터 적용된 사항으로 앱에서 사용할 수 있는 비SDK 인터페이스가 제한되었다.
[Developer site](https://developer.android.com/distribute/best-practices/develop/restrictions-non-sdk-interfaces?hl=ko) 참고  
간단히 말하면 [여기](https://developer.android.com/reference/packages?hl=ko) 에 공식적으로 공개되지 않은 SDK는 쓰지 못한다는 것이다. 따라서 reflection의 사용도 금지된다.



[WifiManager.java](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/android11-release/wifi/java/android/net/wifi/WifiManager.java) 의 소스코드를 일부 가져와봤다.  
(내가 Wi-Fi 개발을 담당하고 있어서 WifiManager의 코드를 가져온 것도 있지만, 실제로도 많은 3rd party 개발자들이 Wi-Fi 관련 API를 많이 쓰고 있었고(특히 스캔) Android 10(Q OS)부터 상당히 많은 불편함을 겪은 것으로 알고있다. ~~아님말고~~)
```java
/**
     * Set if scanning is always available.
     *
     * If set to {@code true}, apps can issue {@link #startScan} and fetch scan results
     * even when Wi-Fi is turned off.
     *
     * @param isAvailable true to enable, false to disable.
     * @hide
     * @see #isScanAlwaysAvailable()
     */
    @SystemApi
    @RequiresPermission(android.Manifest.permission.NETWORK_SETTINGS)
    public void setScanAlwaysAvailable(boolean isAvailable) {
        try {
            mService.setScanAlwaysAvailable(isAvailable);
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }
```
setScanAlwaysAvailable() 이라는 메소드가 있는데 그 위에 annotaion이 붙어있다.  
그리고 실제로 호출해보았다.
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        var wifiManger : WifiManager = getSystemService(Context.WIFI_SERVICE) as WifiManager;
        wifiManger.setScanAlwaysAvailable();
    }
```
setScanAlwaysAvailable는 없는 함수라고 표시된다.
있는데?  
그럼 @hide, @SystemApi, @RequiresPermission 
각각은 무엇을 의미하는걸까?


### @hide
>When applied to a package, class, method or field, @hide removes that node and all of its children from the documentation.

@hide는 javadoc의 일부이다.
### @SystemApi

###@RequiresPermission