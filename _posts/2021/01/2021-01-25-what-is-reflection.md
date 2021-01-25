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

이전 포스팅에서 테스트했었던 Refelction의 예시다.
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