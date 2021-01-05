---
title: "Android 터치 이벤트 전달 방식"
date: 2018-11-06 11:30:29 +0900
category: [개발]
tags: [android, event-handling]
---

과제 진행 중에 터치에 따라 움직이는 floating view 내에 클릭 가능한 view가 있는 UI를 구현하게 되었는데, 클릭이벤트 때문에 터치이벤트가 동작하지 않는 문제가 있어 분석하다가 안드로이드 터치 이벤트 전달방식에 대해 알아 보게 되었습니다.

<!--more-->

![Android Touch Event 1](/assets/img/android-touch-event-1.png)

위의 그림과 같은 안드로이드 layout을 가정하도록 하겠습니다. 이 때 View에서 터치 이벤트()가 발생하면, Activity 를 시작으로 최상단 View 까지 터치가 발생된 좌표에 존재하는 모든 view에 이벤트가 통보(`dispatchTouchEvent()`)됩니다. 그런 다음 최상단 View에서 부터 Activity까지 모든 View에서 이벤트를 처리할 기회(`onTouchEvent()`)가 주어집니다. 따라서, 다음 그림과 같이 Activity가 이벤트를 최초로 수신하며 마지막으로 이벤트를 처리할 기회를 얻게 됩니다.

![Android Touch Event 2](/assets/img/android-touch-event-2.png)

만약 특정 `ViewGroup`에서 바로 터치 이벤트를 처리하고 상위 View로 터치 이벤트를 전달하지 않으려 한다면, 해당 `ViewGroup`의 `onInterceptTouchEvent()`에서 `true`를 리턴하면 됩니다.

`View`(또는 `ViewGroup`)이 `OnTouchListener`를 가지고 있다면, 터치 이벤트는 해당 리스너의 `OnTouchListener.onTouch()`에 의해 처리됩니다. 리스너가 없을 때는 해당 View의 `onTouchEvent()`에서 이벤트가 처리됩니다. 터치 이벤트 처리 후 `true`를 리턴할 경우 이벤트는 더 이상 하위 View로 전달되지 않습니다.

## 더 자세한 설명

위의 다이어그램은 실제 동작보다는 조금 단순화되어 있습니다. 예를 들어, `Activity`와 `ViewGroup` A 사이에는 `Window`와 `DecorView`가 존재합니다. 일반적으로 개발시에는 이러한 객체와는 직접 상호작용할 필요가 없기 때문에 위의 다이어그램에서는 생략되어 있습니다. 하지만, 아래 설명에서는 코드레벨의 분석이기 때문에 이러한 객체들을 포함해서 설명합니다. 아래 코드들은 AOSP 소스에서 필요한 부분만 발췌한 내용입니다.

1. 터치 이벤트가 발생하면 [Activity](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/Activity.java){:target="_blank"}의 `dispatchTouchEvent()`로 전달됩니다. 터치 이벤트는 x, y 좌표, 시간, 이벤트 타입 등의 정보가 포함된 `MotionEvent`의 형태로 전달됩니다.
2. `MotionEvent`는 Activity 내의 `Window` 객체의 `superDispatchTouchEvent()`로 전달됩니다. `Window`는 추상 클래스이며 실제 구현은 `PhoneWindow`입니다.

    ```java
    // Activity.java
    final void attach(...) {
      mWindow = new PhoneWindow(this, window, activityConfigCallback);
    }

    public Window getWindow() {
      return mWindow;
    }

    public boolean dispatchTouchEvent(MotionEvent ev) {
      ...
      if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
      }
      return onTouchEvent(ev);
    }
    ```

3. `PhoneWindow`의 `superDispatchTouchEvent()`는 `DecorView`의 `superDispatchTouchEvent()`를 호출합니다. `DecorView`는 상태바, 내비게이션 바, 컨텐츠 영역 등을 관리하는 View이며, 실제로는 `FrameLayout`의 서브클래스이며 `ViewGroup`을 root View로 가집니다.
    ```java
    // PhoneWindow.java
    @Override
    public boolean superDispatchTouchEvent(MotionEvent event) {
      return mDecor.superDispatchTouchEvent(event);
    }
    ```
    `superDispatchTouchEvent()`는 아래 코드에서 보는 바와 같이 `FrameLayout`의 `dispatchTouchEvent()`를 호출하게 되고 Layout은 결국 `ViewGroup`의 서브 클래스이므로 `ViewGroup`의 `dispatchTouchEvent()`가 호출됩니다.

    ```java
    // DecorView.java
    public class DecorView extends FrameLayout implements RootViewSurfaceTaker, WindowCallbacks {
      ...
      ViewGroup mContentRoot;
      ...

      public boolean superDispatchTouchEvent(MotionEvent event) {
        return super.dispatchTouchEvent(event);
      }
      ...
    }
    ```

4. `ViewGroup`의 `dispatchTouchEvent()` 코드는 실제로는 상당히 복잡한대요. 간략히(그리고 저도 분석한데 까지만..) 설명하자면 다음과 같습니다. 먼저 `onInterceptTouchEvent()`의 리턴 값이 `true` 일 경우 상위 View로 터치 이벤트를 전달하지 않고 자신이 바로 이벤트를 처리합니다. 이벤트를 인터셉트하지 않은 경우 `dispatchTransformedTouchEvent()`를 통해 child(있을 경우) View(혹은 ViewGroup, ViewGroup역시 View의 서브클래스)에 이벤트를 전달(`dispatchTouchEvent()`)합니다. 결국 이러한 로직에 의해 `DecorView`의 child View인 **ViewGroup A** 로 터치 이벤트가 전달됩니다.

    ```java
    // ViewGroup.java
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
      ...
      // Check for interception.
      final boolean intercepted;
      if (actionMasked == MotionEvent.ACTION_DOWN
            || mFirstTouchTarget != null) {
        final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
        if (!disallowIntercept) {
          intercepted = onInterceptTouchEvent(ev);
          ev.setAction(action); // restore action in case it was changed
        } else {
          intercepted = false;
        }
      } else {
        // There are no touch targets and this action is not an initial down
        // so this view group continues to intercept touches.
        intercepted = true;
      }

      ...

      if (!canceled && !intercepted) {
        ...
        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
          ...
        }
        ...
      }

      // 위에서 intercept 한경우 cancelChild 가 true가 되어
      // 현재 target이 recycle()되며 다음 타겟으로 변경
      final boolean cancelChild = resetCancelNextUpFlag(target.child) || intercepted;
      if (dispatchTransformedTouchEvent(ev, cancelChild,
        target.child, target.pointerIdBits)) {
        handled = true;
      }
      if (cancelChild) {
        if (predecessor == null) {
          mFirstTouchTarget = next;
        } else {
          predecessor.next = next;
        }
        target.recycle();
        target = next;
        continue;
      }

      return handled;
    }
    ```

5. 4번과 동일하게 이벤트는 **ViewGroup A** 의 child인 **ViewGroup B** 로 전달됩니다. 역시 모든 ViewGroup에서는 `onInterceptTouchEvent()`를 통해 더이상 하위 View로 이벤트가 전달되지 않도록 할 수 있습니다.
6. `ViewGroup`의 `dispatchTransformedTouchEvent()`는 결국 `View`의 `dispatchTouchEvent()`를 호출합니다.

    ```java
    // ViewGroup.java
    private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
        View child, int desiredPointerIdBits) {
      final boolean handled;

      ...
      if (child == null) {
        // ViewGroup 의 super class 는 'View' 입니다.
        handled = super.dispatchTouchEvent(event);
      } else {
        // child 역시 'View'입니다.
        handled = child.dispatchTouchEvent(event);
      }
      ...

      return handled;
    }
    ```

7. `View`의 `dispatchTouchEvent()`에서는 `OnTouchListener`가 존재할 경우 `OnTouchListener.onTouch()`를 호출하며, 리스너가 없다면 View의 `onTouchEvent()`를 호출하게 됩니다.

    ```java
    // View.java
    public boolean dispatchTouchEvent(MotionEvent event) {
      ...

      ListenerInfo li = mListenerInfo;
      if (li != null && li.mOnTouchListener != null
              && (mViewFlags & ENABLED_MASK) == ENABLED
              && li.mOnTouchListener.onTouch(this, event)) {
          result = true;
      }

      if (!result && onTouchEvent(event)) {
          result = true;
      }
      ...

      return result;
    }
    ```

8. 계속해서 재귀적으로 parent 레이어의 `dispatchTouchEvent()`가 호출됩니다. 위에서 언급한대로 `ViewGroup` 역시 `View`의 서브클래스기 때문에 `OnTouchListner.onTouch()` 또는 `onTouchEvent()`가 호출됩니다.
9. 이벤트를 parent View로 계속해서 전달한다면(`onTouchListener.onTouch()` 또는 `onTouchEvent()`의 리턴이 `false`) 마지막으로 `Activity`의 `onTouchEvent()`가 호출됩니다.

## 메서드 고급 활용

레이아웃 내에서 좀 더 복잡한 이벤트를 처리하기 위해서 다음과 같은 메서드를 재구현하여 이벤트를 가로채거나 특정 View에서 처리하도록 할 수 있습니다.

### `Activity.dispatchTouchEvent(MotionEvent)`

위의 설명에서 이벤트의 최초 진입점은 이 메소드입니다. 따라서, 이 메소드를 재구현하여 child View로 이벤트가 전달되기 전에 가로챌 수 있습니다. 반대로 잘못 사용할 경우 모든 터치 이벤트에 영향을 주기 때문에 조심해서 사용해야 합니다.

### `ViewGroup.onInterceptTouchEvent(MotionEvent)`

이 메소드에서 `true`를 리턴하면 해당 이벤트는 더이상 child로 전달되지 않으며, 해당 View에서 마지막으로 처리됩니다. 보통은 child View에서 모든 이벤트를 처리할 경우 parent View에서 이벤트 처리가 되지 않는 경우가 많기 때문에 특정 이벤트를 이 메소드를 통해 인터셉트하여 먼저 처리한 다음 하위 View로 이벤트를 전달하기 위해 사용합니다. 기본 구현은 대부분의 이벤트를 child View로 전달하도록 구현되어 있습니다.

```java
// ViewGroup.java
public boolean onInterceptTouchEvent(MotionEvent ev) {
      if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
            && ev.getAction() == MotionEvent.ACTION_DOWN
            && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
            && isOnScrollbarThumb(ev.getX(), ev.getY())) {
        return true;
      }
      return false;
  }
```

### `ViewParent.requestDisallowInterceptTouchEvent(boolean)`

위와는 반대로 child view에서 parent view에게 이벤트를 가로채지 말도록 요청할 때 이 메소드를 사용합니다. 이 메소드의 인자가 `true`인 경우 `FLAG_DISALLOW_INTERCEPT` 플래그를 셋팅하게 되고 이 플래그는 아래 코드에서 보는 것처럼 `ViewGroup.dispatchTouchEvent()`에서 `onInterceptTouchEvent()`를 타지않게 합니다.

```java
// ViewGroup.java -> dispatchTouchEvent() 메소드
// Check for interception.
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN
        || mFirstTouchTarget != null) {
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {
        intercepted = onInterceptTouchEvent(ev);
        ev.setAction(action); // restore action in case it was changed
    } else {
        intercepted = false;
    }
} else {
    // There are no touch targets and this action is not an initial down
    // so this view group continues to intercept touches.
    intercepted = true;
}
```

## 참고

* [stackoverflow: How are Android touch events delivered?](https://stackoverflow.com/questions/7449799/how-are-android-touch-events-delivered/46862320#46862320)
* [Android developers: 입력 이벤트](https://developer.android.com/guide/topics/ui/ui-events)
* [Android Source](https://android.googlesource.com)