---
title: 안드로이드 다이얼러 만들기
date: 2018-11-11 10:48:02 +0900
category: [개발]
tags: [android]
---

안드로이드 6.0(마시멜로우) 부터 "기본 전화 앱"을 교체할 수 있는 기능을 제공합니다. 새로운 전화앱을 구현하기 위해서는 [android.telecom](https://developer.android.com/reference/android/telecom/package-summary){:target="_blank"} 패키지의 [TelecomManager](https://developer.android.com/reference/android/telecom/TelecomManager.html){:target="_blank"}, [InCallService](https://developer.android.com/reference/android/telecom/InCallService.html){:target="_blank"}, [Call](https://developer.android.com/reference/android/telecom/Call.html){:target="_blank"} 등의 클래스를 재구현하여야 합니다.

> 이 글에서 설명한 예제코드의 풀소스는 [Github](https://github.com/sogeuni/android-dialer-example)에 등록하여 놓았으니 참고하세요.

<!--more-->

## 기본 전화앱으로 설정하기

### 매니페스트 퍼미션 선언

전화를 받기 위해서는 별다른 퍼미션을 정의하지 않아도 됩니다. 하지만 전화걸기 기능을 위해서 `MANAGE_OWN_CALLS` 퍼미션 혹은 `CALL_PHONE` 퍼미션을 선언하여야 합니다. `MANAGE_OWN_CALLS` 퍼미션의 경우 자체 `ConnectionService`를 구현하는 경우에 선언하여야 하며, `CALL_PHONE` 퍼미션의 경우 Dialer로 인터페이스 없이 Call을 수행할 수 있는 권한입니다. 그 외. 필요에 따라 `READ_CALL_LOG`, `READ_PHONE_STATE` 등의 퍼미션을 추가할 수 있습니다.

```xml
<manifest ... >
  <uses-permission android:name="android.permission.MANAGE_OWN_CALLS"/>
  <!-- or -->
  <uses-permission android:name="android.permission.CALL_PHONE"/>
  ...
</manifest>
```

`MANAGE_OWN_CALLS` 퍼미션을 사용하는 경우 위에서 말한 대로 `ConnectionService`를 구현해 주어야 합니다. 이부분은 [안드로이드 가이드](https://developer.android.com/guide/topics/connectivity/telecom/selfManaged#connectionImplementation){:target="_blank"}에 잘 설명되어 있는데, 제가 테스트 할 때는 잘 동작하지 않아서 저는 `CALL_PHONE` 퍼미션을 선언하여 테스트 했습니다.

안드로이드 6.0 부터 [런타임 퍼미션](https://developer.android.com/training/permissions/requesting?hl=ko){:target="_blank"}이 적용되어 매니페스트에 퍼미션을 정의하는 것만으로는 api 사용시에 Exception이 발생합니다. 따라서 위의 퍼미션 선언 외에 코드에서도 퍼미션을 체크하여 권한이 없는 경우 시스템에 권한 요청하는 작업이 추가로 필요합니다.

### 인텐트 필터 추가

다음과 같이 Activity에 `DIAL` 액션을 추가해 줍니다. `DIAL` 액션은 data에 지정된 번호로 전화를 거는 액션입니다.

```xml
<activity android:name=".MainActivity">
  ...
  <intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <action android:name="android.intent.action.DIAL"/>

    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>

    <data android:scheme="tel"/>
  </intent-filter>
  <intent-filter>
    <action android:name="android.intent.action.DIAL"/>

    <category android:name="android.intent.category.DEFAULT"/>
  </intent-filter>
</activity>
```

위와 같이 두 개의 인텐트 필터를 추가하여야 합니다. 둘 중 하나라도 추가하지 않으면 기본 전화앱으로 설정할 수 없게 됩니다.

### 기본 전화앱 교체를 사용자에게 물어보기

설정에서 수동으로 기본 전화앱을 변경할 수도 있지만, 런타임 퍼미션처럼 전화앱을 실행할 때마다 기본 전화앱을 교체할 것인지 물어볼 수도 있습니다. 그러기 위해서는 MainActivity에서 다음과 같은 동작이 필요합니다.

* [`TelecomManager.getDefaultDialerPackage`](https://developer.android.com/reference/android/telecom/TelecomManager#getDefaultDialerPackage()){:target="_blank"} 메소드를 이용하여 현재 앱이 기본 전화앱인지 확인합니다.
* 현재앱이 기본 전화앱이 아닌 경우 `TelecomManager.ACTION_CHANGE_DEFAULT_DIALER` 액션과 현재앱의 패키지명을 data로 저장한 `Intent`를 만들어 `Activity`를 시작합니다.
* 실행한 `Activity`의 결과를 기다렸다가 Toast로 출력합니다.

예제 코드는 다음과 같습니다.

```java
public class MainActivity extends AppCompatActivity {

  private static final int REQUEST_CODE_SET_DEFAULT_DIALER = 100;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
  }

  @Override
  protected void onStart() {
    super.onStart();
    checkDefaultDialer();
  }

  @Override
  protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_CODE_SET_DEFAULT_DIALER) {
      checkSetDefaultDialerResult(resultCode);
    }
  }

  @Override
  public void onRequestPermissionsResult(int requestCode,
                                         String permissions[], int[] grantResults) {

    if (REQUEST_CODE_GRANT_PERMISSIONS == requestCode) {
      int check = 0;

      for (Integer result : grantResults) {
        check += result;
      }

      if (check < 0) {
        Toast.makeText(this, "need more permissions", Toast.LENGTH_LONG).show();
        finish();
      }
    }
  }

  /**
    * 기본 Dialer를 체크하고 변경
    */
  private void checkDefaultDialer() {
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
      return;
    }

    TelecomManager telecomManager = (TelecomManager) getSystemService(Context.TELECOM_SERVICE);
    boolean isAlreadyDefaultDialer;

    try {
      isAlreadyDefaultDialer = telecomManager.getDefaultDialerPackage().equals(getPackageName());
    } catch(NullPointerException e) {
      isAlreadyDefaultDialer = false;
    }

    if (isAlreadyDefaultDialer) {
      return;
    }

    Intent intent = new Intent(TelecomManager.ACTION_CHANGE_DEFAULT_DIALER);
    intent.putExtra(TelecomManager.EXTRA_CHANGE_DEFAULT_DIALER_PACKAGE_NAME, getPackageName());
    startActivityForResult(intent, REQUEST_CODE_SET_DEFAULT_DIALER);
  }

  private void checkSetDefaultDialerResult(int resultCode) {
    String message;

    switch(resultCode) {
      case RESULT_OK:
        message = "기본 전화 앱으로 설정하였습니다.";
        break;
      case RESULT_CANCELED:
        message = "기본 전화 앱으로 설정하지 않았습니다.";
        break;
      default:
        message = "Unexpected result code " + resultCode;
    }

    Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
  }
}
```

> 참고로 `TelecomManager` 관련 대부분의 메소드는 안드로이드 6.0 이상에서만 사용할 수 있으니, SDK 버전을 확인하는 것을 잊지마세요.

## `InCallService` 콜백

### 서비스 등록

OS로 부터 Call 상태를 받기 위해서는 [InCallService](https://developer.android.com/reference/android/telecom/InCallService){:target="_blank"}를 구현해야 합니다.

서비스 구현을 위해 먼저 매니페스트에 다음과 같이 서비스를 등록합니다.

```xml
<service
  android:name=".CallService"
  android:permission="android.permission.BIND_INCALL_SERVICE">
  <meta-data
    android:name="android.telecom.IN_CALL_SERVICE_UI"
    android:value="true"/>

  <intent-filter>
    <action android:name="android.telecom.InCallService"/>
  </intent-filter>
</service>
```

`InCallService`는 추상 클래스이므로 필요한 메소드들만 추가 구현하면 됩니다.

인텐트 필터와 `InCallService`가 매니페스트에 정상적으로 등록되었다면 안드로이드 설정의 기본 전화앱 선택창에서 이 앱을 볼 수 있게 됩니다.

### 서비스 구현

이제 이 앱은 시스템에서 전화앱으로 인식하게 되었으며, 기본 전화앱으로도 사용할 수 있게 되었습니다. 하지만, 전화를 걸거나 받기 위해서는 위에서 manifest에 정의했던 `InCallService` 콜백에서 필요한 메소드를 구현하여야 합니다.

```java
public class CallService extends InCallService {

  @Override
  public void onCallAdded(Call call) {
    super.onCallAdded(call);
    // 현재 Call 객체에 state 변경상태를 받을 콜백을 등록합니다.
    call.registerCallback(callCallback);
    // 이곳에서 CallActivity를 호출합니다.
    startActivity(new Intent(this, CallActivity.class));
  }

  @Override
  public void onCallRemoved(Call call) {
    super.onCallRemoved(call);
    // Call 객체의 콜백을 해제합니다.
    call.unregisterCallback(callCallback);
  }

  private Call.Callback callCallback = new Call.Callback() {

    @Override
    public void onStateChanged(Call call, int state) {
      // 변경된 Call 상태를 CallActivity로 전송합니다.
      CallManager.get().updateCall(call);
    }
  };
}
```

실제로 `InCallService`의 메소드는 위의 것 말고도 훨씬 더 많습니다만, 이 예제에서는 Call이 생성되고 종료되는 시점과 Call 상태가 변경되는 시점을 구현했습니다. Call이 생성되면 `CallActivity`를 호출하고 `Call`객체의 상태 변경을 받을 콜백을 등록합니다. Call 상태가 변경될 때마다 `onStateChanged` 콜백이 불릴 것이며, 현재 Call 상태를 바탕으로 UI를 업데이트 합니다.

## `CallManager` 클래스

예제코드에서는 어플리케이션에서 전화 기능을 UI에서 쉽게 접근할 수 있도록 `CallManager`클래스를 만들었습니다. `CallManager`는 incoming/outgoing call을 컨트롤할 수 있는 method와 현재 call 상태를 UI로 전달해주는 StateListener를 가지고 있습니다. 각 method와 interface는 `TelecomManager`와 `InCallService`를 래핑합니다. 또한 `CallManager`의 접근을 용이하게 하도록 싱글톤으로 구현하였습니다.

```java
public class CallManager {

    public interface StateListener {
        void onCallStateChanged(UiCall call);
    }

    private static CallManager sInstance = null;

    private TelecomManager mTelecomManager;
    private Call mCurrentCall = null;
    private StateListener mStateListener = null;

    public static CallManager init(Context applicationContext) {
      if (sInstance == null) {
        sInstance = new CallManager(applicationContext);
      } else {
        throw new IllegalStateException("CallManager has been initialized.");
      }
      return sInstance;
    }

    public static CallManager get() {
      if (sInstance == null) {
        throw new IllegalStateException("Call CallManager.init(Context) before calling this function.");
      }
      return sInstance;
    }

    private CallManager(Context context) {
      mTelecomManager = (TelecomManager) context.getSystemService(Context.TELECOM_SERVICE);
    }

    public void registerListener(StateListener listener) {
      mStateListener = listener;
    }

    public void unregisterListener() {
      mStateListener = null;
    }
    ...
}
```

### 전화 상태 업데이트

Incoming/outgoing call 시도시 위에서 구현한 `CallService`를 통해 현재의 전화상태를 알 수 있는 `Call` 객체가 전달됩니다. 하지만, `Call` 객체는 전화에 대한 모든 정보를 담고 있기 때문에 UI에서 사용하기 적절하도록 데이터를 변환해 줍니다.

```java
// CallManager
public void updateCall(Call call) {
  mCurrentCall = call;

  if (mStateListener != null && mCurrentCall != null) {
    mStateListener.onCallStateChanged(UiCall.convert(mCurrentCall));
  }
}
```

`CallManager`의 `updateCall()` 함수는 `Call`객체를 `UiCall`이라는 간소화된 정보로 변환하여 등록된 `StateListener` 콜백으로 전달합니다. `InCallService`의 `onCallAdded()`, `onCallRemoved()` 그리고 `Call.Callback`의 `onStateChanged()` 콜백에서 위의 함수를 호출하면 UI에 전화상태를 전달할 수 있습니다.

## Call 화면

이제 전화상태를 표시하며, 상태에 따라 버튼이 show/hide 되는 Call 화면을 구현합니다. Call 화면은 일반적인 Android Activity로 구현하며, 현재의 전화상태를 얻기 위해 `StateListener`를 구현합니다. 그 외 구현에 따라 전화를 받고 끊기 위한 버튼이나, 전화번호, 통화시간들을 표현할 수 있는 다양한 View 및 비즈니스 로직을 가지게 됩니다.

```java
public class CallActivity extends Activity
  implements View.OnClickListener, CallManager.StateListener Handler.Callback {

  private static final long PERIOD_MILLIS = 1000L;
  private static final int MSG_UPDATE_ELAPSEDTIME = 100;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_call);

    // Activity 생성시 현재 Call 정보로 한번 업데이트
    updateView(CallManager.get().getUiCall());
  }

  @Override
  protected void onResume() {
    super.onResume();
    CallManager.get().registerListener(this);
  }

  @Override
  protected void onPause() {
    super.onPause();
    CallManager.get().unregisterListener();
  }

  @Override
  public void onCallStateChanged(UiCall uiCall) {
    updateView(uiCall);
  }

  /**
    * 현재 전화 상태에 따라 view 모양 변경
    *
    * @param uiCall
    */
  private void updateView(UiCall uiCall) {
    // UiCall 데이터를 바탕으로
    // View의 활성/비활성/Text 등을 재구성
  }

  // Duration을 그리기 위한 Timer
  private void startTimer() {
    stopTimer();

    mElapsedTime = 0L;
    mTimer = new Timer();
    mTimer.schedule(new TimerTask() {
      @Override
      public void run() {
        mElapsedTime++;
        // 쓰레드 내에서 UI 업데이트가 불가하므로 handler를 이용하여 업데이트
        mHandler.sendEmptyMessage(MSG_UPDATE_ELAPSEDTIME);
      }
    }, 0, PERIOD_MILLIS);
  }

  private void stopTimer() {
    if (mTimer != null) {
      mTimer.cancel();
      mTimer = null;
    }
  }

  private String toDurationString(long time) {
    return String.format(Locale.US, "%02d:%02d:%02d",
                        time / 3600, (time % 3600) / 60, (time % 60));
  }

  @Override
  public boolean handleMessage(Message msg) {
    switch (msg.what) {
      case MSG_UPDATE_ELAPSEDTIME:
        mTextDuration.setText(toDurationString(mElapsedTime));
        break;
    }
    return true;
  }
}
```

## 마치며

이상으로 간단한 안드로이드 다이얼러를 구현해 보았습니다. 실제 전화앱으로 사용하려면 contact등과 연동도 해야하고 그 외 추가적인 많은 기능들을 더 구현해야 하지만 일단 전화를 걸고 받는 기본 기능은 완료되었습니다.

이 글과 예제코드는 [Android default dialer replacement](https://medium.com/makingtuenti/android-default-dialer-replacement-part-i-722d05ca50fe){:target="_blank"} 글과 예제코드를 참고하여 만들었습니다. 원 글의 코드가 kotlin으로 구현되어 있어서 java로 컨버팅해보았는데, 작업을 하면서 느낀 점은 동일 기능을 구현하기 위해 kotlin 코드가 더 간단하고 직관적이었다는 점이었습니다. 빨리 kotlin을 배워야 할 것 같네요. :scream: