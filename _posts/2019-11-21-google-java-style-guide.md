---
title: Google 자바 코딩 스타일 가이드
date: 2019-11-21 00:00:00 +0900
category: [개발]
tags: [java, coding-rule]
---

원글: [https://google.github.io/styleguide/javaguide.html](https://google.github.io/styleguide/javaguide.html){:target="_blank"}

이 문서는 Java<sup>TM</sup> 프로그래밍 언어로 짜여진 소스 코드에 대한 구글 코딩 표준을 **완전히** 정의한다. 여기의 규칙을 준수하는 Java 소스 파일의 경우에만 *구글 스타일*로 표시한다.

다른 프로그래밍 스타일 가이드와 마찬가지로, 이 문서는 포맷의 미적 문제뿐 아니라 다른 유형의 관습(convention)이나 코딩 표준에 대해서도 다루고 있다. 하지만, 이 문서는 기본적으로 우리가 보편적으로 따르는 **엄격하고 빠른 규칙**(hard-and-fast rules)에 중점을 두고 있으며 (사람이던 도구던)명확하게 시행할 수 없는 조언은 피하고 있다.

## 용어 정리

따로 명시하지 않는 한 이 문서에서는 다음과 같이 정의한다.

1. *클래스(class)* 라는 용어는 "일반적인" 클래스 열거형(enum) 클래스, 인터페이스 또는 어노테이션 타입을 의미하기 위해 포괄적으로 사용된다.
2. (클래스의) *멤버(member)* 라는 용어는 중첩(nested) 클래스, 필드, 메소드 또는 생성자(constructor)를 의미하기 위해 포괄적으로 사용된다. 즉, 이니셜라이저와 주석을 제외한 클래스의 모든 최상위 수준의 컨텐츠이다.
3. *주석(comment)* 이라는 용어는 언제나 코드내의 주석을 의미한다. "Javadoc"이라는 일반적인 용어 대신에 "문서 주석(documentation comments)"이라는 용어를 사용하지 않는다.

다른 "용어 정리"들은 문서 전체에 걸쳐 종종 나올 수 있다.

## 가이드 노트

이 문서의 예제 코드는 **비표준(non-normative)**이다. 따라서, 예제는 구글 스타일을 설명하지만 코드를 스타일리쉬하게 짤 수 있는 방법을 설명하지는 않는다. 예제에 작성된 선택적인 포맷은 규칙으로 적용해서는 안된다.

# 소스 파일 기본

## 파일명

소스 파일 이름은 파일 내의 최상위 클래스의 이름과 [동일하며](#하나의-최상위-클래스만-선언) 대소문자를 구분하는 이름과 `.java` 확장자로 구성된다.

## 파일 인코딩: UTF-8

소스 파일은 **UTF-8**로 인코딩한다.

## 특수 문자

### Whitespace 문자

소스 파일 내의 유일한 whitespace 문자는 **ASCII horizontal space character (0x20)** 이다. 이것은 다음을 의미한다.

1. 문자열(string) 및 문자 리터럴(character literals)내의 다른 모든 whitespace 문자는 이스케이프 처리한다.
2. 들여쓰기에 Tab을 **사용하지 않는다**.

### 특수한 이스케이프 시퀀스

[특수한 이스케이프 시퀀스](https://docs.oracle.com/javase/tutorial/java/data/characters.html){:target="_blank"}를 가지는 문자(`\b`, `\t`, `\n`, `\f`, `\r`, `\"`, `\'`, `\\`)의 경우 8진수 이스케이프(e.g. \012)나 유니코드 이스케이프(e.g. \u000a) 대신에 해당 이스케이프를 사용한다.

### 비 ASCII 문자

ASCII 문자가 아닌경우 실제 유니코드 문자(e.g. ∞)나 대응하는 유니코드 이스케이프(e.g. \u221e)를 사용한다. 문자열 리터럴과 주석에서 유니코드를 이스케이프처리하는 것은 권장하지 않지만, 선택은 전적으로 **읽고 이해하기 쉽게** 코드를 작석하는데 달렸다.

> 팁: 유니코드를 이스케이프할 경우나 실제 유니코드 문자를 사용할 때 주석으로 설명하는 것이 유용하다.

Examples:

| Example | Discussion |
|--------------------------
| `String unitAbbrev = "μs";` | 가장 좋음. 주석이 없어도 명확함 |
| `String unitAbbrev = "\u03bcs"; // "μs"` | 허용하지만 이렇게 쓸 이유가 없음 |
| `String unitAbbrev = "\u03bcs"; // Greek letter mu, "s"` | 허용하지만 어색하고 실수하기 쉬움 |
| `String unitAbbrev = "\u03bcs";` | 나쁨.<br>코드를 보는 사람이 무엇인지 알 수 없음 |
| `return '\ufeff' + content; // byte order mark` | 좋음.<br>출력할 수 없는 문자는 이스케이프 처리하고<br>필요하다면 주석 추가 |

> 팁: 특정 프로그램이 유니코드를 제대로 처리할 수 없을거라는 두려움 때문에 코드를 읽기 힘들게 만들진 말자. 설령 그런 일이 일어난다해도 그 프로그램은 **죽을 것**이며 반드시 **수정**해야하니까.

# 소스 파일 구조

소스 파일은 다음 순서로 구성된다.

1. 라이센스 또는 저작권 정보(있다면)
2. Package 구문
3. Import 구문
4. 하나의 최상위 클래스

각 섹션은 **한 줄씩 띄어서** 구분한다.

## 라이센스 또는 저작권 정보(있다면)

파일에 대한 라이센스 또는 저작권 정보가 있다면 이곳에 적는다.

## Package 구문

Package 구문은 **줄바꿈하지 않는다**. [컬럼 제한](#컬럼-제한-100자)이 적용되지 않는다.

## Import 구문

### 와일드카드 import는 사용하지 않음

static을 포함하여 어떤 것이든 **와일드카드 import는 사용하지 않는다**.

### 줄바꿈하지 않음

Import 구문은 **줄바꿈하지 않는다**. [컬럼 제한](#컬럼-제한-100자)이 적용되지 않는다.

### 순서 및 간격

Import는 다음의 순서를 따른다.

1. 모든 static import를 하나의 블록에 적는다.
2. 모든 static이 아닌 import를 하나의 블록에 적는다.

static 블록과 non-static 블록이 함께 있는 경우 두 블록 사이를 한 줄 띄운다. 다른 import 문 사이에는 빈줄은 없어야 한다.

블록 내의 각 import 문은 ASCII 문자순으로 정렬한다.

### 클래스에 대한 static import는 없음

정적 중첩 클래스에 대해 static import를 사용하지 않고 일반적인 import를 사용한다.

## 클래스 선언

### 하나의 최상위 클래스만 선언

하나의 소스파일에는 하나의 최상위 클래스만 선언한다.

### 클래스 컨텐츠 순서

멤버들과 이니셜라이저의 순서는 코드를 이해하는데(learnability) 큰 영향을 줄 순 있지만, 순서에 대해 하나의 명확한 룰은 없다. 각 클래스 마다 각각의 방식으로 순서를 조정할 수 있다.

중요한 것은 각 클래스는 코드 관리자가 설명할 수 있는 **논리적인 순서**를 사용한다는 것이다. 예를 들어, 새로운 메소드를 클래스에 끝에 추가하는 것은 "추가된 날짜순으로" 순서를 산출하는 것이기에 논리적인 순서가 아니다.

#### 오버로딩(Overload)은 분리하지 않음

동일한 이름의 생성자나 메소드를 여러개 가진 클래스의 경우 순차적으로 작성하며 이들 사이에 (private 멤버를 포함한) 다른 코드를 넣지 않는다.

# 형식

**용어 정리:** *블록과 같은 구조(block-like construct)*는 클래스, 메소드 또는 생성자 본문을 나타낸다. [배열 이니셜라이저](#배열-이니셜라이저-블록처럼-사용가능)는 블록과 같은 구조로 간주할 수 있다.

## 괄호(Braces)

### 선택사항일 경우의 괄호 사용법

`if`, `else`, `for`, `do`, `while` 문의 본문이 없거나 한줄일 경우에도 괄호를 사용한다.

### 비어있지 않은 블록: K & R 스타일

*비어있지 않은* 블록과 블록과 같은 구조의 경우 Kernighan and Ritchie style ("[Egyptian brackets](http://www.codinghorror.com/blog/2012/07/new-programming-jargon.html){:target="_blank"}")을 따른다.

* 여는 괄호 전에 줄바꿈하지 않는다.
* 여는 괄호 뒤에 줄바꿈한다.
* 닫는 괄호 전에 줄바꿈한다.
* 메소드, 생성자, 클래스의 닫는 괄호일 경우 뒤에 줄바꿈한다. 예를 들어 닫는 괄호 뒤에 `else`나 콤마가 따라오는 경우 줄바꿈하지 않는다.

예제:
```java
return () -> {
  while (condition()) {
    method();
  }
};

return new MyClass() {
  @Override public void method() {
    if (condition()) {
      try {
        something();
      } catch (ProblemException e) {
        recover();
      }
    } else if (otherCondition()) {
      somethingElse();
    } else {
      lastThing();
    }
  }
};
```

열거형 클래스에 대한 약간의 예외는 [Enum 클래스](#enum-클래스)를 참고한다.

### 빈 블록: 줄일 수 있음

비어있는 블록 및 블록과 같은 구조는 [위에 서술](#비어있지-않은-블록-k--r-스타일)한대로 K & R 스타일을 사용할 수도 있고, *다중 블록 구문(multi-block statement*(`if/else`나 `try/catch/finally`)이 아닌 경우 한 줄에 써도 된다. (`{}`)

예제:
```java
// This is acceptable
void doNothing() {}

// This is equally acceptable
void doNothingElse() {
}

// This is not acceptable: 다중 블록 구문에서는 빈 블록을 줄일 수 없다
  try {
    doSomething();
  } catch (Exception e) {}
```

## 블록 들여쓰기(indentation): 스페이스 2칸

블록이나 블록과 같은 구조가 시작될 때마나, 들여쓰기가 스페이스 2칸 만큼 증가한다. 블록이 끝나면 들여쓰기는 이전 레벨로 돌아간다. 들여쓰기 레벨은 코드와 주석 모두에 적용된다. ([비어있지 않은 블록: K & R 스타일](#비어있지-않은-블록-k--r-스타일)의 예제 참고)

## 한 줄에 하나의 구문

각 구문은 마지막에 줄바꿈한다.

## 컬럼 제한: 100자

자바 코드는 100자로 컬럼을 제한한다. "문자(character)"는 모든 유니코드 코드 포인트를 의미한다. 아래 언급한 예외를 제외하고는 모두 [줄 바꿈(Line-wrapping)](#줄-바꿈)항목에서 설명하는 대로 줄바꿈한다.

> 각 유니코드 코드 포인트는 하면에 크게 보이던 작게 보이던 하나의 문자로 계산한다. 에를 들어, [전각 문자(fullwidth characters)](https://en.wikipedia.org/wiki/Halfwidth_and_fullwidth_forms){:target="_blank"}를 사용할 경우 이 규칙에서 요구하는 것보다 일찍 줄바꿈할 수도 있다.

예외:
1. 컬럼 제한을 준수할 수 없는 경우 (예를 들어 Javadoc 내의 긴 URL이나 긴 JSNI 메소드 레퍼런스)
2. `package` 와 `import` 구문 ([Package 구문](#package-구문) 과 [Import 구문](#import-구문) 참조)
3. shell 에 붙여넣기할 수 있도록 명령어를 주석화 한 경우

## 줄 바꿈

**용어 정리:** 한줄로 작성할 수 있는 코드를 여러 줄로 분할하는 행위를 *줄 바꿈(line-wrapping)*이라 한다.

모든 상황에 맞는 정확하고 포괄적인 규칙은 존재하지 않으며, 몇몇 상황에 적절한 방법이 있을 뿐이다.

> **참고:** 컬럼 제한에 의해 줄 바꿈 하는 것이 일반적이지만, 제한 내에 있더라도 작성자에 판단에 따라 줄 바꿈이 있을 수 있다.

> **팁:** 메소드나 지역 변수를 추출하여 줄바꿈하지 않도록 할 수 있다.

### 언제 바꿀까

줄 바꿈하기 위한 주요 지시어는 다음과 같다. 

1. *비 대입* 연산자에서 컬럼이 넘어서면 심볼 *앞*에서 줄바꿈한다. (구글의 C++, JavaScript 과 같은 코딩 스타일과는 다름)
  * 이 규칙은 다음과 같은 "연산자 형" 심볼에도 적용된다.
    * dot separator (`.`)
    * the two colons of a method reference (`::`)
    * an ampersand in a type bound (`<T extends Foo & Bar>`)
    * a pipe in a catch block (`catch (FooException | BarException e)`).
2. *대입* 연산자에서 컬럼이 넘어서면 일반적으로 심볼 *뒤*에서 줄바꿈하지만, 다른 방식도 허용한다.
  * 이 규칙은 "할당 연산자 형" 심볼인 `for` ("`foreach`")의 콜론에도 적용된다.
3. 메소드나 생성자의 이름 뒤의 열린 괄호(`(`)는 줄바꿈하지 않는다.
4. 콤마(`,`)는 앞의 항목에 붙여쓴다.
5. 람다에 인접한 화살표는 람다의 내용이 괄호없는 한줄로 구성된 경우를 제외하고는 줄바꿈하지 않는다.


```java
MyLambda<String, Long, Object> lambda =
    (String label, Long value, Object obj) -> {
        ...
    };

Predicate<String> predicate = str ->
    longExpressionInvolving(str);
```

> **참고:** 줄 바꿈의 기본 목표는 코드를 명확하게 하는 것이지 라인을 줄이는 것이 아니다.

### 연속된 들여쓰기는 스페이스 4칸 이상

줄 바꿈시 연속된 라인은 원래 줄에서 4칸이상 들여쓰기 한다.

연속된 라인이 여러줄일 경우 4칸 이상 원하는 만큼 들여쓰기한다. 일반적으로 두라인 이상이 문법적으로 수평적인 항목으로 시작한다면 동일한 들여쓰기 레벨을 적용한다.

[수평 정렬](#수평-정렬은-필요하지-않음)에 관한 섹션에서 여러 토큰을 이전행과 정렬하기 위해 스페이스를 다양하게 쓰지 않는 것이 좋음을 보여준다.

## 공백(Whitespace)

### 수직 공백

다음의 경우 항상 한 줄씩 띄운다.

1. 연속된 멤버나 클래스의 이니셜라이저 간: 필드, 생성자, 중첩 클래스, 정적 이니셜라이저 인스턴스 이니셜라이저
  * **예외:** (사이에 아무런 코드도 없는) 두 개의 연속된 필드의 경우 옵션이다. 필드의 *논리적 그룸*을 생성할 필요가 있을 때 한 줄씩 띄워쓴다.
  * **예외:** enum 상수 사이의 빈 줄은 [Enum 클래스](#enum-클래스)에서 다룬다.
2. 이 문서의 다른 항목에서 요구하는 방식에 따라 띄운다.([소스 파일 구조](#소스-파일-구조)나 [Import 구문](#import-구문))

코드의 논리적인 하위 섹션을 구정하기 위해서 라던지 코드의 가독성을 높이기 위해 어디서나 한 줄씩 띄울 수 있다. 클래스의 첫번째 멤버, 이니셜라이저의 앞 혹은 마지막 멤버, 이니셜라이저의 뒤에 한 줄 띄우는 것은 권장하지도 금지하지도 않는다.

연속된 여러개의 빈 줄은 허용하지만 절대 권장하지는 않는다.

### 수평 공백

언어나 다른 스타일 규칙을 넘어 문자열, 주석, Javadoc에서 요구되는 규칙을 제외하고 하나의 ASCII 스페이스는 다음의 경우에만 나타난다.

1. `if`, `for`, `catch` 등의 예약어는 뒤에 따르는 괄호(`(`)와 한 칸 띄운다.
2. `else`, `catch`와 같은 예약어 앞에 닫는 중괄호(`{`)와 한 칸 띄운다.
3. 아래 두 예외를 제외하고 여는 중괄호(`{`) 앞은 한 칸 띄운다.
  * `@SomeAnnotation({a, b})` (스페이스를 사용하지 않음)
  * `String[][] x = {{"foo"}};` (\{\{ 사이에 스페이스를 사용하지 않음. 8번 항목 참고)
4. 2항 연산자, 3항 연산자 양쪽으로 공백 추가. 
  * 이 규칙은 아래의 "연산자 형" 심볼에도 적용된다.
    * the ampersand in a conjunctive type bound: `<T extends Foo & Bar>`
    * the pipe for a catch block that handles multiple exceptions: `catch (FooException | BarException e)`
    * the colon (`:`) in an enhanced `for` ("`foreach`") statement
    * the arrow in a lambda expression: `(String str) -> str.length()`
  * 다음의 경우 공백을 사용하지 않는다.
    * the two colons (`::`) of a method reference, which is written like `Object::toString`
    * the dot separator (`.`), which is written like `object.toString()`
5. `,:;` 의 뒤, 형변환을 위한 괄호(`)`) 의 뒤는 한 칸 띄운다.
6. 코드 끝에 오는 주석의 시작인 더블 슬래시(`//`) 좌우로 한 칸씩 띄운다. 여러칸도 허용하지만 필수는 아님.
7. 변수 선언시 타입과 변수명 사이에 한 칸 띄운다.
8. 배열 이니셜라이저에서 양쪽 괄호 안쪽
  * `new int[] {5, 6}` 과 `new int[] { 5, 6 }` 모두 허용
9. 타입 어노테이션 `[]` 또는 `...` 사이는 한 칸 띄운다.

위의 규칙은 라인의 시작 또는 끝의 추가 공백에는 적용하지 않고 코드 *내부* 공간에만 적용된다.

### 수평 정렬은 필요하지 않음

**용어 정리:** *수평 정렬*이란 특정 토큰을 이전 행의 다른 토큰과 줄맞추기 위해 임의의 추가 스페이스를 입력하는 것을 말함

이것은 허용되나 구글 스타일에서는 **결코 필요하지 않으며** 이미 사용하고 있더라도 유지할 필요가 없다.

```java
private int x; // this is fine
private Color color; // this too

private int   x;      // permitted, but future edits
private Color color;  // may leave it unaligned
```

> **팁:** 정렬이 가독성을 높이긴 하지만 향후 유지보수시 문제가 발생한다. 향후에 한줄만 고치게 된다면 잘 맞춰놓은 포맷은 엉망이 되며 정렬을 맞추기 위해 근처 라인의 연속적인 수정이 발생할 수 있다. 한 줄 변경이 "나비 효과"가 되어 최악의 경우에는 버전 기록 정보가 손상되고 리뷰에 시간이 걸리게 되고 병합 충돌이 발생할 수 도 있다.

## 그룹화를 위한 괄호: 권장됨

그룹화를 위한 괄호는 코드가 잘못 해석될 가능성이 없고 읽는데 무리가 없다면 작성자와 리뷰어 동의하에 생략할 수 있다. 하지만, 모든 독자들이 자바 연산자의 우선순위 테이블을 기억하고 있다고 가정하는 것은 합리적이지 않다.

## 특수 구조

### Enum 클래스

각 enum 상수 뒤의 콤마 뒤의 줄바꿈은 가능하며, 추가적인 빈줄(일반적으로 한 줄)도 허용한다.

```java
private enum Answer {
  YES {
    @Override public String toString() {
      return "yes";
    }
  },

  NO,
  MAYBE
}
```

메소드가 없고 도큐먼트도 없는 enum 클래스는 [배열 이니셜라이저](#배열-이니셜라이저-블록처럼-사용가능) 처럼 작성할 수 있다.

```java
private enum Suit { CLUBS, HEARTS, SPADES, DIAMONDS }
```

enum 클래스도 *클래스* 이므로 다른 형식은 클래스의 형식을 따른다.

### 변수 선언

#### 한번에 하나의 변수를 선언한다

모든 변수(field, local)는 하나씩 선언한다. `int a, b;` 와 같이 사용하지 않는다.

**예외:** `for` 루프의 헤더내에서는 여러 변수 선언을 허가한다.

#### 필요할 때 선언한다

지역 변수는 습관적으로 블록의 시작에 선언하지 않으며, 변수의 스코프를 최소화하기 위해 처음 사용되는 지점에 가깝게 선언한다. 지역 변수는 일반적으로 이니셜라이저를 가지거나 선언하자마자 초기화 한다.

### 배열

#### 배열 이니셜라이저: 블록처럼 사용가능

배열 이니셜라이저는 "블록과 같은 구조"로 작성할 수 있다. 따라서 다음 모든 경우는 허용된다.
```java
new int[] {           new int[] {
  0, 1, 2, 3            0,
}                       1,
                        2,
new int[] {             3,
  0, 1,               }
  2, 3
}                     new int[]
                          {0, 1, 2, 3}
```

#### C 스타일의 배열 선언은 허용하지 않음

대괄호(square bracket)는 변수가 아닌 *타입*의 일부분이다: `String[] args` 이지 `String args[]`는 아니다.

### Switch 구문

**용어 정리:** *switch 블록*의 중괄호는 하나 이상의 *statement group*을 가진다. 각 statement group은 하나 이상의 *switch label*(`case F00:`이나 `default:`)과 그에 따르는 구문을 가진다.

#### 들여쓰기

다른 블록과 같이 switch 블록도 2칸 들여쓰기한다.

switch label 뒤는 link break를 하고 블록시작과 같이 2칸 들여쓰기 한다. 다음 switch label이 오면 블록의 끝과 같이 이전 들여쓰기 레벨로 지정한다.

#### Fall-through: 주석

`break` 없이 다음 switch 블록으로 이동하는 형태로 코드를 작성하는 경우 주석으로 표시한다.(일반적으로 `// fall through`) 마지막 그룹은 이 주석을 달 필요가 없다.

```java
switch (input) {
  case 1:
  case 2:
    prepareOneOrTwo();
    // fall through
  case 3:
    handleOneTwoOrThree();
    break;
  default:
    handleLargeNumber(input);
}
```

`case 1:` 의 경우처럼 구문이 없는 경우에는 주석이 필요없다.

#### `default` case는 존재해야 함

코드가 없더라도 `default` statement group은 존재해야 한다.

**예외:** `enum` 타입의 경우 생략할 수 있다. 모든 명시적인 열거형 값이 포함되지 않은 경우, IDE나 다른 정적 분석 툴에서 경고를 나타낼 수 있다.

### 어노테이션

클래스, 메소드, 생성자에 적용되는 어노테이션은 도큐먼트 블록 바로 다음에 표시하며, 한줄에 하나의 어노테이션을 적는다. 어노테이션은 [줄 바꿈](#줄-바꿈)에 포함되지 않으므로 들여쓰기 하지 않는다.

예제:
```java
@Override
@Nullable
public String getNameIfPresent() { ... }
```

**예외:** 어노테이션이 하나이고 파라미터가 없는 경우에는 다음과 같이 한줄로 쓸 수 있다.

```java
@Override public int hashCode() { ... }
```

필드에 어노테이션을 적용할 때도 도큐먼트 블록 바로 다음에 표시하는데, 이 경우에는 여러 어노테이션(파라미터가 있어도 가능)을 다음과 같이 한줄에 써도 된다.

```java
@Partial @Mock DataLoader loader;
```

파라미터, 지역 변수, 타입에 대한 어노테이션 형식은 특별한 규칙이 없다.

### 주석

이 섹션은 *코드 내의 주석*에 대해 다룬다, Javadoc은 [Javadoc](#javadoc) 항목을 참고한다.

#### 블록 주석 스타일

블록 주석은 주변 코드와 동일한 들여쓰기 레벨을 가진다. `/* ... */` 스타일, `// ...` 스타일 모두 허용하며, 멀티라인을 위한 `/* ... */` 주석의 경우 이후 라인은 `*`로 시작해야 하며 이전행의 `*`과 줄맞춤 한다.

```java
/*
 * This is                     
 * okay.                     
 */

// And so
// is this. 

/* Or you can
 * even do this. */
```

주석은 asterisk(`*`)나 그외 다른 문자로 만든 박스로 에워싸지 않는다.

> **팁:** 멀티라인 주석을 작성할때 자동 코드 포맷팅이 필요하다면 `/* ... */` 스타일을 사용한다. 대부분의 포매터는 `// ...` 스타일의 주석 블록을 자동으로 정렬해주지 않는다.

### 제어자(Modifier)

클래스나 맴버 제어자의 경우 Java 언어 스펙에서 추천하는 순서대로 표기한다.

```java
public protected private abstract default static final transient volatile synchronized native strictfp
```

### 숫자 리터럴

`long` 리터럴은 대문자 `L` 접미어를 사용한다.(숫자 `1`과의 혼동을 피하기 위해서) `3000000000l` 보다 `3000000000L`을 사용한다.

# 명명법

## 모든 식별자에 대한 공통 규칙

식별자는 ASCII 문자와 숫자만 사용하며 다음에 서술할 몇몇 경우에만 추가로 underscore를 사용한다. 따라서 유효한 식별자는 정규 표현식 `\w+`에 매치된다.

구글 스타일에서는 접두어나 접미어는 **사용하지 않는다.** 예를 들어, `name_`, `mName`, `s_name`, `kName` 등은 구글 스타일이 아니다.

## 식별자 타입에 따른 규칙

### 패키지 명

패키지 명은 모두 소문자로 작성하며 underscore를 사용하지 않는다. 예를 들어, `com.example.deepspace`는 가능하지만, `com.example.deepSpace`나 `com.example.deep_space`등은 사용하지 않는다.

### 클래스 명

클래스 명은 [UpperCamelCase](#camel-case-정의)로 작성한다.

클래스 명은 일반적으로 `Character`나 `ImmutableList` 같이 명사나 명사구를 사용한다. 인터페이스 명도 일반적으로 명사나 명사구를 사용하지만(`List`와 같이), 종종 형용사나 형용사구를 사용할 수도 있다(`Readable`과 같이).

어노테이션 타입에 대한 특별한 규칙은 없다.

*테스트* 클래스는 `HashTest`나 `HashIntegrationsTest`처럼 테스트할 클래스의 이름 뒤에 `Test`를 붙인다.

### 메소드 명

메소드 명은 [lowerCamelCase](#camel-case-정의)로 작성한다.

메소드 명의 경우 일반적으로 `sendMessage`나 `stop`과 같이 동사나 동사구를 사용한다.

JUnit 테스트 메소드의 경우 underscore로 이름의 논리적 컴포넌트를 분리하고 각 컴포넌트는 [lowerCamelCase](#camel-case-정의)로 작성한다. 한가지 일반적인 패턴은 `pop_emptyStack`과 같이 *`<methodUnderTest>_<state>`* 형태로 작성하는 것이지만 테스트 메소드를 이름을 정하는 정확한 방법은 없다.

### 상수 명

상수명은 대문자와 단어사이의 underscore를 써서 `CONSTANT_CASE` 형태로 사용한다.

상수는 불변하는 static final 필드이다. primitive, String, immutable 타입, immutable 타입의 immutable 콜렉션을 포함한다. 인스턴스의 상태에 따라 변하면 상수가 아니다. 예를 들면,

```java
// Constants
static final int NUMBER = 5;
static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
static final ImmutableMap<String, Integer> AGES = ImmutableMap.of("Ed", 35, "Ann", 32);
static final Joiner COMMA_JOINER = Joiner.on(','); // because Joiner is immutable
static final SomeMutableType[] EMPTY_ARRAY = {};
enum SomeEnum { ENUM_CONSTANT }

// Not constants
static String nonFinal = "non-final";
final String nonStatic = "non-static";
static final Set<String> mutableCollection = new HashSet<String>();
static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
static final ImmutableMap<String, SomeMutableType> mutableValues =
    ImmutableMap.of("Ed", mutableInstance, "Ann", mutableInstance2);
static final Logger logger = Logger.getLogger(MyClass.getName());
static final String[] nonEmptyArray = {"these", "can", "change"};
```

이름은 일반적으로 명사나 명사구를 사용한다.

### 상수가 아닌 필드 명

(static을 포함한) 필드 명은 [lowerCamelCase](#camel-case-정의)를 사용한다.

일반적으로 `computeValues`나 `index`와 같이 명사나 명사구를 사용한다.

### 파라미터 명

파라미터는 [lowerCamelCase](#camel-case-정의)로 작성한다.

public 메소드에서 한글자 파라미터 명을 사용하는 것은 피한다.

### 지역 변수 명

지역 변수는 [lowerCamelCase](#camel-case-정의)로 작성한다.

final 이고 불변이더라도 지역 변수는 상수처럼 취급하지 않으며, 상수처럼 작성하지 않는다.

### 타입 변수 명

타입 변수는 다음 두가지 스타일 중 하나를 따른다.

* 대문자 한자, 추가적으로 숫자 하나까지 (`E`, `T`, `X`, `T2` 등)
* [클래스에 사용되는 이름 형식](#클래스-명)에 대문자 `T` (예: `RequestT`, `FooBarT`)

## Camel case 정의

"IPv6"나 "iOS"와 같이 특별한 약어를 사용하는 경우에 영어 어구를 camel case로 바꾸기 위해 구글 스타일은 다음과 같은 규칙을 따른다.

1. 아포스트로피를 지우고 plain ASCII로 변환한다. 예를 들어, "Müller's algorithm"은  "Muellers algorithm"으로 변환한다.
2. 결과를 단어 단위로 공백으로 나눈다.
  * *추천:* 단어가 이미 camel-case 형태로 구성되어있다면 그것 역시 구성 단위로 나눈다. (예를 들어, "AdWords"는 "ad words") "iOS"와 같은 단어는 camel case가 아니므로 이 경우 적용하지 않는다.
3. (약어를 포함하여) 모두 소문자로 변환하고 첫 글자만 대문자화 한다.
  * *upper camel case*의 경우 모든 단어에 적용하고,
  * *lower camel case*의 경우 첫단어의 대문자화는 제외한다.
4. 마지막으로 모든 단어를 하나의 식별자로 join 한다.

다음의 예처럼 원래 단어의 대소문자는 거의 무시된다.

| 문장 형식 | 변경 후 | 잘못된 예 |
|-----------|---------|-----------|
| "XML HTTP request" | `XmlHttpRequest` | XMLHTTPRequest |
| "new customer ID" | `newCustomerId` | newCustomerID |
| "inner stopwatch" | `innerStopwatch` | innerStopWatch |
| "supports IPv6 on iOS | `supportsIpv6OnIos` | supportsIPv6OnIOS |
| "YouTube importer" | `YouTubeImporter`<br>`YoutubeImporter`\* | |

\* 허용하지만 권장하지 않음

> **노트:** 일부 영단어의 경우 "nonempty"와 "non-empty" 처럼 모호하게 하이픈으로 구별되는 경우가 있는데, 이경우 `checkNonempty`와 `checkNonEmpty` 모두 허용한다.

# 코딩 규칙

## `@Override`는 항상 사용한다

슈퍼 클래스로 부터 오버라이드된 클래스 메소드, 인터페이스 메소드는 항상 `@Override` 어노테이션을 붙인다.

**예외:** 부모 메소드가 `@Deprecated` 된 경우 `@Override`를 생략할 수 있다.

## Exception catch 를 무시하지 않는다

아래 명시한 예외케이스를 제외하고는 exception catch 구문에서 아무 것도 하지 않으면 않된다. (일반적으로 로그메시지를 출력하던가, 동작불능 상태라면 `AssertionError`를 다시 던진다)

catch 블록에서 아무 것도 하지 않는 것이 정말 정당하다면 이유를 주석으로 표시한다.

```java
try {
  int i = Integer.parseInt(response);
  return handleNumericResponse(i);
} catch (NumberFormatException ok) {
  // it's not numeric; that's fine, just continue
}
return handleTextResponse(response);
```

**예외:** 테스트 코드에서 의도한 exception일 경우 주석없이 파라미터를 `expected`로 사용할 수 있다. 

```java
try {
  emptyStack.pop();
  fail();
} catch (NoSuchElementException expected) {
}
```

## static 멤버의 경우 클래스를 사용한다

static 클래스 멤버를 참조할 때는 클래스 명으로 참조하여야 한다.

```java
Foo aFoo = ...;
Foo.aStaticMethod(); // good
aFoo.aStaticMethod(); // bad
somethingThatYieldsAFoo().aStaticMethod(); // very bad
```

## Finalizer는 사용하지 않는다

`Object.finalize`를 오버라이드하는 케이스는 거의 없다.

> **팁:** 하지마라, 꼭 해야겠다면 [Effective Java 규칙 7](http://books.google.com/books?isbn=8131726592){:target="_blank"} "Avoid Finalizers" 를 읽고 이해한 다음 절대로 하지마라.

# Javadoc

## 형식

### General form

Javadoc의 *일반적인* 형식은 다음과 같다.

```java
/**
 * Multiple lines of Javadoc text are written here,
 * wrapped normally...
 */
public int method(String p1) { ... }
```

또는, 다음과 같이 한줄로 쓸 수도 있다.

```java
/** An especially short bit of Javadoc. */
```

한줄로 적을 때로 전체적인 Javadoc 형식(comment marker 등)을 지켜야 하며, `@return`과 같은 block tag가 없을 경우에만 한줄로 적는다.

### 단락(Paragraphs)

한줄 띄우는 경우, 즉 하나의 asterisk(`*`)만 존재하는 라인은 단락 사이와 block tag 그룹 앞에 사용한다. 첫 단락을 제외한 단락은 첫단어 앞에 `<p>`로 시작한다. `<p>` 다음에 스페이스 없이 바로 내용이 시작된다.

### Block tags

block tag는 `@param`, `@return`, `@throws`, `@deprecated`의 순서로 작성하며 내용이 없을 경우 넣지 않는다. 내용을 한줄에 다 적지 못하는 경우 다음줄에 `@`로 부터 4칸 이상 들여쓰기 한다.

## The summary fragment

Javadoc 블록은 간단한 summary fragment로 시작한다. 이것은 클래스나 메소드를 설명하는 중요한 부분이다.

이 fragment는 명사구나 동사구로 작성하며 완전한 문장 형태가 아니다. `A {@code Foo} is a...`나 `This method returns...`와 같이 시작하지 않고, `Save the record.`와 같이 명령형으로 적지 않는다. 하지만, 완벽한 문장처럼 대문자로 시작하고 문장부호를 표시한다.

> **팁:** 일반적인 실수는 간단한 Javadoc을 `/** @return the customer ID */`과 같은 형태로 적는 것이다. 이것은 `/** Returns the customer ID. */`처럼 고쳐야 한다.

## Javadoc은 어디에 적을까

*최소한* 모든 public 클래스와 모든 public, protected 멤버에 있어야 하며, 몇가지 예외는 아래에 설명한다.

아래 설명할 [필수가 아닌 Javadoc](#필수가-아닌-javadoc) 컨텐츠도 있을 수 있다.

### 예외: self-explanatory methods

`getFoo`와 같이 간단하고 명확한 경우, 실제로 "Returns the foo" 정도밖에 설명할게 없을 경우 생략가능하다.

> **중요:** 예를 들어 `getCanonicalName`과 같은 메소드에서 `/** Returns the canonical name. */ 만으로 설명이 부족한 경우, 읽는 사람이 "canonical name"의 의미를 알지 못하는 경우에 Javadoc을 생략하는 것은 좋지 않다.

### 예외: overrides

슈퍼타입 메소드를 상속받은 메소드의 경우 Javadoc은 옵션이다.

### 필수가 아닌 Javadoc

코드 내 주석이 클래스 멤버의 전체 목적이나 동작방식을 설명하는데 사용된다면 Javadoc 형식으로 작성한다.(`/**`를 사용)

이러한 Javadoc은 필수는 아니므로 위의 규칙을 엄격히 따를 필요는 없다.