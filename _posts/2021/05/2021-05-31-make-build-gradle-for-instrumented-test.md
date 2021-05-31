---
title: "Make build.gradle for Android Instrumented Test(작성중)"
date: 2021-05-31
categories: [Android]
tags: [Android, gradle]
comments: true
---

Android Instrumented 테스트를 위한 build.gradle 만들기  

작업 중인 프로젝트의 Moderator가 아니고 한 모듈만 개발중인데, Android Instrumented test setup이 필요한 상태에서 메인 gradle을 건드릴 수 없는 상태.  
그래서 우리가 작업중인 라이브러리에만 build.gradle을 추가하기로 했다. 

#### Unit test VS Instrumented test  
**Unit Test(단위테스트)**는 로컬 머신에서만 동작하고 JVM에서 수행된다.  
주로 Android Framework에 종속되지 않거나, Mock object에 dependency가 없는 테스트를 수행한다.  
**Instrumented test**는 JUnit package라는 점은 동일하지만 test가 실행되기 전에 인스턴스화가 되어야하고 테스트를 하려면 실제 단말(혹은 에뮬레이터)가 필요하다. 
테스트 또한 단말(혹은 에뮬레이터)에서 수해오딘다. 
실제로 안드로이드 컴포넌트에 접근하고(버튼같은 리소스 등), application lifecycle과 동일한 lifecycle을 가진다.  
그래서 Android framework에 의존적이며 Mock 또한 프레임워크에 종속적이다.


#### 여러 System application에서 사용하는 static library에 Test 추가하기
일반적인 경우에는 신경쓰지 않아도 되겠지만, 내가 일부를 담당하고 있는 프로젝트는 아주 복잡하고 안드로이드 프레임워크에 대.단.히. 의존적이다.  
다른 모듈의 build.gradle을 수정할 수 없으며 우리 팀 담당인 library의 build.gradle도 수정할 수 없었다.😕



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
```
androidTestImplementation project(path: ':<Static library module name>')

    androidTestImplementation 'junit:junit:4.12'
```
```
dependencies {
    compileOnly project(':external')

    implementation fileTree(include: ['*.jar'], dir: rootDir.path + '/some_library')
    implementation fileTree(include: ['*.aar'], dir: rootDir.path + '/some_library')
```

