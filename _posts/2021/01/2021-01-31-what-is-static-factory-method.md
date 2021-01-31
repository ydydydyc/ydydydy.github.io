---
title: "정적 팩토리 메소드(Static factory method)와 Builder 패턴의 이해"
date: 2021-01-31
categories: [Java]
tags: [Java, Effective Java, Static factory method, Builder]
comments: true
---

**Effective Java 2/E를 공부하며 정리한 내용입니다.**

<br>

## Static factory method
일반적으로 객체를 생성하는 방법은 생성자(Constructor)를 통해 만드는 것이다.  
하지만 public으로 선언된 **정적 팩토리 메소드**를 추가하는 것도 하나의 방법이고 Effective java에서는 생성자 대신 정적 팩토리 메소드를 사용할 수 있다면 대체하는 것을 제안하고 있다.

우리가 일반적으로 알고있는 Factory 디자인 패턴과는 다른 개념이다. (나도 첨에 요거 말하는 줄 알았음,,, 머쓱)  
정적 팩토리 메소드는 **클래스를 정의할 때 생성자가 아닌 다른 메소드로 객체생성을 하는 방법**을 뜻한다.  
처음에는 이게 무슨 말인지 이해가 안갔는데, 내가 많이 쓰고 있는 방법이었고 그렇게 어렵지 않은 개념이다.

### 장점 1 : 정적 팩토리 메소드는 이름을 가진다.
자주 쓰는 valueOf, getInstance, getType 등의 메소드가 대표적인 정적 팩토리 메소드의 좋은 예이다.
```java
    public static Boolean valueOf(boolean var0) {
        return var0 ? TRUE : FALSE;
    }

    public static Boolean valueOf(String var0) {
        return parseBoolean(var0) ? TRUE : FALSE;
    }
```

valueOf라는 메소드를 통해 Boolean type 객체를 반환함으로서 Boolean 객체를 생성한 것이다.  
이 예제는 굉장히 간단한 예제긴 하지만, 생성자에 파라미터들이 많은 경우 해당 API에 익숙하지 않은 개발자들은 생성자를 call 할 때 많은 실수를 범하게 된다.(나처럼^^)  
사실 대부분의 IDE들이 함수 파라미터를 잘 가이드해주고 있긴 하지만.. 우리팀은 얼마 전까지만 해도 IDE의 사용이 불가능한 모듈을 개발했었고  
자동완성만 되어도 감사한 개발 환경에서 생성자 파라미터의 순서가 바뀌는 일은 매우 빈번했다.

새로운 AccessPoint를 생성하는 프로그램이 있다고 가정할 때
```java
import java.sql.Time;
import java.time.LocalTime;

public class AccessPoint {
    static final String DEFAULT_MAC = "02:00:00:00:00:00";

    String mSsid;
    String mMac;
    int mSecurity;
    int mNetworkId;

    public AccessPoint(String SSID, String mac, int security, int netId) {
        mSsid = SSID;
        mMac = mac;
        security = 0; // OPEN
        netId = -1;
    }

    public static AccessPoint newConfig() {
        Time t = Time.valueOf(LocalTime.now());
        return new AccessPoint("AP"+t.toString(), DEFAULT_MAC, 0, -1);
    }

    public String toString() {
        StringBuilder result = new StringBuilder();
        result.append(mSsid).append(" ").append(mMac);
        return result.toString();
    }
}
```
default AccessPoint를 생성하는 객체를 만든다고 가정했을 때, 그냥 일반적인 생성자만 호출하면 AccessPoint(String, String, int, int)형이기 때문에 혼돈이 오는 사람들이 있을 가능성이 크다(그리고 일단 귀찮음)  
AccessPoint 객체를 리턴하는 정적 팩토리 메소드를 구현하면 API 사용자는 혼돈과 실수 없이 객체를 생성할 수 있다.  
현재 시간을 이용해 만든 SSID와 Default MAC 주소 등을 넣어 newConfig라는 static method를 구현해보았다.  
(의도한건 아닌데 Time을 가져오기 위해 호출한 Time.valueOf도, LocalTime.now도 모두 정적 팩토리 메소드다. 여기저기서 많이 사용되고있다.)  
```java
public class WifiSettings {

    public static void main(String[] args) {
        AccessPoint ap = AccessPoint.newConfig();
        System.out.println(ap.toString());
    }
}
```
new 없이 ap 객체를 만들었고 toString으로 호출 시 잘 호출된 것을 확인 가능하다.
```
AP14:06:26 02:00:00:00:00:00

Process finished with exit code 0
```

### 장점 2 : 정적 팩토리 메소드를 쓰면 호출할 때마다 새로운 객체를 생성할 필요가 없다.

객체를 재사용할 수 있다. 
```java
import java.sql.Time;
import java.time.LocalTime;
import java.util.Date;

public class AccessPoint {
    static final String DEFAULT_MAC = "02:00:00:00:00:00";
    static final AccessPoint INVALID_AP = new AccessPoint("", DEFAULT_MAC, -1, -1);

    ...
  
    public static AccessPoint newConfig() {
        Time t = Time.valueOf(LocalTime.now());
        Date today = new Date();
        if (t.before(today)) {
            return INVALID_AP;
        }
        return new AccessPoint("AP"+t.toString(), DEFAULT_MAC, 0, -1);
    }
    ...
}
```
말이 안되는 코드이긴 하지만... 귀찮아ㅋㅎ  
현재 시간보다 이전에 만들어진 AP라면 INVALID 처리하게 만드는 코드이다.
INVALID_AP는 이미 만들어둔 객체이기 때문에 불필요한 객체가 거듭 생성되는 일을 피할 수 있다.

### 장점 3 : 반환값 자료형의 하위 자료형 객체를 반환할 수 있다.
반환되는 객체의 클래스를 유연하게 결정할 수 있다.

```java
    public static AccessPoint newConfig(int securityType) {
        if (securityType == 1) {
            return Wpa.newConfig();
        } else if (securityType == 2) {
            return Passpoint.newConfig();
        }
    }

    public class Passpoint extends AccessPoint
    public class Wpa extends AccessPoint
```
AccessPoint를 상속받는 Passpoint, Wpa 클래스 두개를 만들었고, security type에 따라 각각 맞는 하위 타입을 리턴해준다.  
(귀찮아서 다 안짰는데.. 저건 컴파일 안된다. 왜냐면 return이 마무리지어지지 않아서)  

그리고 public으로 선언되지 않은 클래스의 객체를 반환하는 API를 만들 수 있다.  
이러한 기법은 인터페이스 기반 프레임워크(Interface-based framework) 구현에 적합한데, 이 프레임워크에서 인터페이스는 장작 팩토리 메소드의 반환값 자료형으로 사용된다.  
인터페이스는 정적 메소드를 가질 수 없기 때문에 관습상 객체 생성 불가능 클래스 안에 둔다.
대표적인 예가 Collections 클래스이다. 
```java
public class Collections {
  public static final Set EMPTY_SET = new Collections.EmptySet();
    ...
  private Collections() {
  }
  ...
  public static final <T> Set<T> emptySet() {
    return EMPTY_SET;
  }
  ...
  private static class EmptySet<E> extends AbstractSet<E> implements Serializable {
}
```
EmptySet은 private class이고 emptySet이라는 정적 팩토리 메소드를 통해 접근 가능하다.
실제 Collections에 대부분은 객체 생성 불가능 클래스의 정적 팩토리 메소드를 통해 이용하는데, 반환되는 객체들의 실제 클래스는 public이 아닌 경우가 대부분이다.  
각 클래스별로 모두 public한 클래스들을 만들었다면 API 규모는 더 커졌을 것이고, API 사용자들도 사용하기 꽤나 번거로웠을 것이다.  

메소드의 인자를 통해 어떤 클래스의 객체를 만들지 말지도 동적으로 결정할 수 있기 때문에 유지보수나 성능 향상에도 유리하다.  

### 장점 4 : 형인자 자료형(Paramiterized type) 객체를 만들 때 편리하다.
문맥상 형인자가 명백하더라도 반드시 인자로 형인자를 전달해야한다.  
```java
Map<String, List<String>> m = new HashMap<String, List<String>>();
```
세상 귀찮은 이짓거리를 정적 팩토리 메소드를 이용하면 컴파일러가 형인자를 스스로 알아낼 수 있도록 할 수 있다.
```java
public static <K, V> HashMap<K, V> newInstance() {
  return new HashMap<K, V>();
  }
  
Map<String, List<String>> m = new HashMap.newInstance();
```
요런 식으로 구현했다면 매우 편리했을 것이다. 이런 기법을 자료형 유추라고 한다.  
다행스럽게도 Java 1.7부터는 저런 짓을 안해도 된다.
```java
Map<String, List<String>> m1 = new HashMap<>();
```

<br>
<br>

## Builder Pattern
AlertDialog.Builder를 사용하면서 상당히 익숙해져있는 패턴이다.
일반적인 생성자나, 정적 팩토리 메소드 둘다 같은 문제를 가지고 있는데, 선택적 인자가 많은 상황에서는 여전히 빡시다는 점.  
```java
    public AccessPoint(String SSID, String mac, int security, int netId) {
        mSsid = SSID;
        mMac = mac;
        security = 0; // OPEN
        netId = -1;
    }
```
실제로 모바일환경에서 새로운 AccessPoint를 만들었을 때, netId는 AccessPoint 단에서 부여하는 것이 아니라 WifiFramework 단에서 동적으로 부여한다.  
즉, netId는 선택 가능한 인자라는 것이다.  
하지만 생성자든, 정팩메든 설정할 필요가 없는 필드에도 항상 인자를 전달해야한다. 이 얼마나 쓸데없는 짓인가? 

즉 인자 수가 늘어나면 클리이언트 코드를 작성하기가 어려워지고, 무엇보다 읽기 어려운 코드가 된다.  
이런 문제를 해결하기 위해 Builder 패턴이 등장했는데  
**필요한 객체를 직접 생성하는 대신 클라이언트는 먼저 필수 인자들을 생성자에 전부 전달하여 빌더 객체를 만들고 빌더 객체에 정의된 설정 메소드를 통해 선택적 인자들을 추가해나간다.**  
그리고 마지막으로 아무런 인자 없이 build 메서드를 호출하여 immutable 객체를 만드는 것이다.  
빌더 클래스는 빌더가 만드는 객체 클래스의 정적 멤버 클래스로 정의한다.

Android에서 빈번하게 사용되는 AlertDialog.Builder다.
필수 요구되는 인자는 Context이고 나머지는 모두 optional이다.
```java
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Title")
                .setMessage("Message")
                .setNegativeButton("No", (dialog, which) -> {
                    
                })
                .setPositiveButton("Yes", (dialog, which) -> {
                    
                })
                .setOnDismissListener(dialog -> {
                    
                })
                .create();
```
각 다이얼로그의 성격에 따라 타이틀, 메시지, 버튼의 종류가 모두 필수가 아니기 때문에 optional로 필요한 것만 호출한 것을 볼 수 있다.  
Builder에 정의된 설정 메소드는 모드 빌더 객체의 자신을 반환하기 때문에 설정 메소드를 호출하는 코드는 죽 이어서 쓸 수 있다.  
작성하기도 쉽고, 읽기도 쉽다.
```java
    public static class Builder {
        @UnsupportedAppUsage
        private final AlertController.AlertParams P;
    
    public Builder setTitle(@StringRes int titleId) {
        P.mTitle = P.mContext.getText(titleId);
        return this;
    }
    
    ...
  
    public Builder setPositiveButton(@StringRes int textId, final OnClickListener listener) {
        P.mPositiveButtonText = P.mContext.getText(textId);
        P.mPositiveButtonListener = listener;
        return this;
    }
```
이렇게 모든 메소드들이 this, builder 자신을 리턴하고 있다.  
빌더 패턴은 인자가 많은 생성자나 정적 팩토리가 필요한 클래스를 설계할 때, 대부분의 인자가 선택적 인자인 상황에 유용하다. 
개인적으로도 예전부터 생각했었지만.. 다음 OS에서는 builder 패턴을 적용해보는게 목표이기도 하다.


<br>
<br>
오류 지적은 언제나 환영입니다:)