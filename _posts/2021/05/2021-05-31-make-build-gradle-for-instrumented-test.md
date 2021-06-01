---
title: "Make build.gradle for Android Instrumented Test"
date: 2021-05-31
categories: [Android]
tags: [Android, gradle]
comments: true
---

Android Instrumented 테스트를 위한 build.gradle 만들기  

작업 중인 프로젝트의 Moderator가 아니고 한 모듈만 개발중인데, Android Instrumented test setup이 필요한 상태에서 메인 gradle을 건드릴 수 없는 상태.  
그래서 우리가 작업중인 라이브러리에만 build.gradle을 추가하기로 했다. 

### Unit test VS Instrumented test  
**Unit Test(단위테스트)**는 로컬 머신에서만 동작하고 JVM에서 수행된다.  
주로 Android Framework에 종속되지 않거나, Mock object에 dependency가 없는 테스트를 수행한다.  
**Instrumented test**는 JUnit package라는 점은 동일하지만 test가 실행되기 전에 인스턴스화가 되어야하고 테스트를 하려면 실제 단말(혹은 에뮬레이터)가 필요하다. 
테스트 또한 단말(혹은 에뮬레이터)에서 수행딘다. 
실제로 안드로이드 컴포넌트에 접근하고(버튼같은 리소스 등), application lifecycle과 동일한 lifecycle을 가진다.  
그래서 Android framework에 의존적이며 Mock 또한 프레임워크에 종속적이다.


### 여러 System application에서 사용하는 static library에 Test 추가하기
일반적인 경우에는 신경쓰지 않아도 되겠지만, 내가 일부를 담당하고 있는 프로젝트는 안드로이드 프레임워크에 아주 아주 의존적이다.  
다른 모듈의 build.gradle을 수정할 수 없으며 우리 팀 담당인 library의 build.gradle도 수정할 수 없었다.😕  
왜냐면 이거를 고치면 다른데서 바로 뭐 안된다고 연락오기 때문에^^~!


먼저 build.gradle을 경로에 만들어주고 
1. testInstrumentationRunner를 defaultConfig에 추가
2. sourceSets에 androidTest를 추가.
여기서 androidTest에는 AndroidManifest와 src 디렉토리만 따로 지정해주었다.
```
apply plugin: 'com.android.application'
...

defaultConfig {
        minSdkVersion 29
        targetSdkVersion 30
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            res.srcDirs = ['../res']
        }
        androidTest {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
        }
    }
```
dependencies에 androidTestImplementation 구문들을 추가했다.  
[StackOverflow](https://stackoverflow.com/questions/52076775/android-difference-between-testimplementation-and-androidtestimplementation-in-b) 에 잘 정리된게 있길래 가져왔다.  

> implementation: The dependency is available in all source sets, including the test source sets.  
> testImplementation: The dependency is only available in the test source set.  
> androidTestImplementation: The dependency is only available in the androidTest source set.  
>  
>**Android source sets are**  
>main: Contains your app code. This code is shared amongst all different versions of the app you can build (known as build variants)  
>androidTest: Contains tests known as instrumented tests.  
>test: Contains tests known as local tests.  

여기서 문제가 있었는데, Test 하려는 모듈이 라이프사이클을 가지고 있는 "library"이고, androidx 라이브러리를 사용해야했다.  
이렇게 aar을 명시적으로 implementation 할 경우,,,
```
dependencies {
    compileOnly project(':external')

    implementation fileTree(include: ['*.jar'], dir: rootDir.path + '/some_library')
    implementation fileTree(include: ['*.aar'], dir: rootDir.path + '/some_library')
```
라이브러리에서 aar을 참조할 수 없다고 에러가 발생했다. 
```
Execution failed for task ':<Testmodule>:bundleDebugAar'.
> Direct local .aar file dependencies are not supported when building an AAR. The resulting AAR would be broken because the classes and Android resources from any local .aar file dependencies would not be packaged in the resulting AAR.  
```
그래서 그냥 아래와 같이 나의 모듈을 전체를 androidTestImplementation로 추가해줬더니 에러는 발생하지 않았다. 
```
    androidTestImplementation project(path: ':<Static library module name>')
    androidTestImplementation 'junit:junit:4.12'
```

<br>
#### Library에서 Resource 참조하기  

빌드도 잘 돌고 TC도 잘 돌길래 TC 수정을 시작했다. 
그런데 자꾸 NPE로 TC가 fail이 났다. StringJoiner의 delimiter가 Null이라고 한다.
  
Test 코드에서 when으로 String 처리를 잘 해주었고
```java
when(mMockContext.getString(R.string.sid)).thenReturn("/");
```
main 코드에서도 특별한건 없었다.
```java
StringJoiner sj = new StringJoiner(mContext.getResources().getString(R.string.sid));
```
둘 다 R 경로 확인시에 com.android.module.test public final class com.android.module.test.R로 test의 R을 동일하게 참조하고 있는데도 자꾸 NPE가 발생했다.
여기에 거의 하루 넘게 매달렸는데ㅠㅠㅠ 이유는 이거때문이었다.
```
apply plugin: 'com.android.application'
```
이걸 
```
apply plugin: 'com.android.library'
```
로 바꾸니 해결 완.  
너무 기뻐서 걍 퇴근해버렸기 때문에 어디서 이 해결책을 찾았는지는 기억이 안난다ㅠㅠ  
이렇게 셋업이 완료됐고 이제 TC만 짜면 됨👀  




<br>
<br>



오류 지적은 언제나 환영입니다😘  
