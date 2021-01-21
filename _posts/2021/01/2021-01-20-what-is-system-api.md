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
isConnectedMacRandomizationSupported()이라는, Random MAC을 지원하는지 여부를 확인하는 API이다.
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
factoryReset은 없는 함수라고 표시된다.
있는데?  
그럼 @hide, @SystemApi, @RequiresPermission 
각각은 무엇을 의미하는걸까?


### @hide
>When applied to a package, class, method or field, @hide removes that node and all of its children from the documentation.

@hide는 javadoc의 일부이며 pulbic API가 아니기 때문에 사용하면 안됨을 의미한다.
@hide를 사용하면 SDK 빌드에 포함되지 않기 때문에 document에는 아예 없는 API가 되는 것이다.
 
### @SystemApi
실제 존재하는 API임은 맞지만, 공개되지는 않은 API이다.
SDK 빌드 시 포함되지만 사용할 수 없고, 사용하고자 한다면 Reflection을 이용해야한다.  
빌드 시 framework와 함께 빌드되는 앱(예를 들면 셋팅)은 빌드 타임에 해당 api를 참조하기 때문에 빌드도 되고, 사용도 가능하다.  

```kotlin
class MainActivity : AppCompatActivity() {

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    val wifiManger : WifiManager = getSystemService(WIFI_SERVICE) as WifiManager

    //wifiManger.isConnectedMacRandomizationSupported();

    val method = wifiManger::class.java.getDeclaredMethod("isConnectedMacRandomizationSupported")
    print(method.invoke(wifiManger))
  }
}
```
