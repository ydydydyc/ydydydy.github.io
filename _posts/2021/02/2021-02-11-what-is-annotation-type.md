---
title: "interface와 @interface의 차이"
date: 2021-02-11
categories: [Java]
tags: [Java]
comments: true
---

AOSP 코드 보다가 

```java
public @interface ConnectedState {}

/** Returns connection state of the network defined by the CONNECTED_STATE constants */
    @ConnectedState
    public int getConnectedState() {
        if (mNetworkInfo == null) {
            return CONNECTED_STATE_DISCONNECTED;
        }

        switch (mNetworkInfo.getDetailedState()) {
            case SCANNING:
            case CONNECTING:
            case AUTHENTICATING:
            case OBTAINING_IPADDR:
            case VERIFYING_POOR_LINK:
            case CAPTIVE_PORTAL_CHECK:
                return CONNECTED_STATE_CONNECTING;
            case CONNECTED:
                return CONNECTED_STATE_CONNECTED;
            default:
                return CONNECTED_STATE_DISCONNECTED;
        }
    }

mConnectedWifiEntry = mStandardWifiEntryCache.values().stream().filter(entry -> {
                final @WifiEntry.ConnectedState int connectedState = entry.getConnectedState();
                return connectedState == CONNECTED_STATE_CONNECTED
                        || connectedState == CONNECTED_STATE_CONNECTING;
            }).findAny().orElse(null /* other */);
```
이렇게 되어있는 코드를 발견했다.  
@interface는 첨보는거라 검색해봄  


일반적으로
```java
public class Generation3List extends Generation2List {

   // Author: John Doe
   // Date: 3/17/2002
   // Current revision: 6
   // Last modified: 4/12/2004
   // By: Jane Doe
   // Reviewers: Alice, Bill, Cindy

   // class code goes here

}
```
이렇게 class를 정의할 때 일반적으로는 클래스를 주석으로 시작한다.  
어떤 역할을 하는 클래스인지, 멤버의 의미는 무엇인지 등등...  

이런 클래스를 annotaion을 사용하면 편하게 만들 수 있다.
이러한 방식을 **Annotaion type**이라고 한다.  
```java
@interface ClassPreamble {
   String author();
   String date();
   int currentRevision() default 1;
   String lastModified() default "N/A";
   String lastModifiedBy() default "N/A";
   // Note use of array
   String[] reviewers();
}
```
Annotation type도 interface의 한 종류이며 바디에는 메소드와 유사한 annotaion type element들이 포함되어 있고 디폴트값도 지정할 수 있다.  
```java
@ClassPreamble (
   author = "John Doe",
   date = "3/17/2002",
   currentRevision = 6,
   lastModified = "4/12/2004",
   lastModifiedBy = "Jane Doe",
   // Note array notation
   reviewers = {"Alice", "Bob", "Cindy"}
)
public class Generation3List extends Generation2List {

// class code goes here

}
```

오류 지적은 언제나 환영입니다.  

참고 : (https://docs.oracle.com/javase/tutorial/java/annotations/declaring.html)
