---
title: Modern Java In Action - (3) 람다 표현식
date: 2021-01-17 23:01:83
category: java
draft: false
---

람다는 어떤 클래스에도 종속되지 않기 때문에 메서드가 아니라 **함수**라고 부른다.

형식은 다음과 같다.

- `() -> { return ... }`
- `a -> { return a; }`
- `a -> "something"`
- `() -> "something"`

**함수형 인터페이스**라는 문맥에서 람다 표현식을 사용할 수 있다.

## 함수형 인터페이스

: 하나의 추상 메서드를 지정하는 인터페이스
Ex) Comparator, Runnable, ActionListener, Callable…  
단 후에 인터페이스에 디폴트 메서드가 있을 수 있는데,  
디폴트 메서드가 있더라도 **추상 메서드가 오직 하나면** 함수형 인터페이스라고 한다.

람다 표현식 = 함수형 인터페이스를 구현한 구현체의 인스턴스

## 함수 디스크립터

함수형 인터페이스의 추상 메서드 시그니처, 이는 곧 람다 표현식의 시그니처
이 람다 표현식의 시그니처를 서술하는 메서드를 **함수 디스크립터** 라고 부른다.

```java
public interface Runnable {
	void run();
}

// 이에 대한 함수 디스크립터는 () -> void
```

_한개의 void 메서드 호출은 중괄호로 감쌀 필요가 없다_

`@FunctionalInterface` : 이 어노테이션을 통해 해당 인터페이스가 함수형 인터페이스임을 가리킬 수 있다. 추상메서드가 한개 이상이면 에러가 발생

## 실행 어라운드 패턴

자원 처리에 사용하는 순환 패턴.  
초기화/ 준비코드 -> 작업 A -> 정리/마무리  
등과 같이 어떤 작업을 위한 setup과 cleanup 작업이 붙는 패턴을 실행 어라운드 패턴이라고 한다. (python의 context manager가 생각나는 패턴)

자바 7에서는 **try-with-resources** 구문을 사용한다.

```java
public String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine();
    }
}
```

람다 표현식을 이용해서 효과적으로 처리해줄 수 있다.

```java

String oneLine = processFile((BufferedReader br) -> br.readLine());

// 이 processFile 함수가 함수형 인터페이스를 인자로 받길 원하도록 구현되어있고, 그 시그니처에 맞춰 람다표현식을 작성해 넣어준다.
```

### 함수형 인터페이스 사용

java.util.function 패키지에 이런 여러가지 함수형 인터페이스를 제공해주고 있다.

유용한 함수형 인터페이스와 시그니처
Predicator.test = (T) -> boolean : 해당 T의 true false 실행
Consumer.accept = (T) -> void : 해당 T를 실행
Function.apply = (T) -> (R) : 해당 T에 대해서 새로운 타입 R을 반환

### 기본형 특화

자바의 제네릭은 참조형밖에 안됨.
따라서 기본형 -> 참조형 (박싱) / 참조형 -> 기본형 (언박싱)를 하는 작업을 제공해줌. 그리고 이게 자동으로 이루어지도록 하는 **autoboxing** 기능을 제공해줌.

-> 근데 이러한 과정이 꽤나 비용이 많이 듬. (기본형을 힙에 할당 …)
따라서 이런 오토박싱을 막기 위해서 기본형 특화의 함수형 인터페이스를 제공해준다.
Ex) IntPredicate, ToIntFunction, IntToDoubleFunction .. 이는 책을 참고하면 된다.

## 형식 검사, 형식 추론, 제약

> 형식 검사

람다는 사용 콘텍스트를 통해서 형식을 추론할 수 있다.
어떤 콘텍스트에서 기대되는 람다 표현식의 형식을 **대상 형식**이라고 한다.
인터페이스의 명세를 보고 해당 대상 형식을 기대하며, 기대하는 대상 형식과 맞다면 올바르게 컴파일을 하게 된다.

따라서 내가 만든 람다 형식이 해당 함수형 인터페이스의 형식과 맞다면, 어떤 함수형 인터페이스의 객체로서 동작할 수 있다.

```java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

### 다이아몬드 연산자

Java 7에 나오는 문법으로, 제네릭 메서드에서 콘텍스트에 따라서 제네릭 형식을 추론할 수 있다.

`List<String> listOfStrings = new ArrayList<>();`

### 특별한 void 호환 규칙

람다 바디에 일반 표현식이 있으면 void 반환 함수 디스크립터와 호환된다.
즉, vold 시그니처여도 일반 표현식이 있는 문이 eval 될 때 bool을 반환하더라도 호환됨.

- 같은 함수형 디스크립터를 가진 두 함수형 인터페이스를 갖는 메서드를 오버로딩 하는 경우, 어떤 메서드의 시그니처가 사용되어야 하는지를 명시적으로 구분하도록 람다를 `캐스팅` 할 수 있다.

```java
public void execute(Runnable runnable) {
	runnable.run();
}

public void execute(Action<T> action) {
	action.act();
}

@FunctionalInterface
interface Action {
	void act();
}

// 사용시는 이렇게 캐스팅, 맞춰서 해당 메서드를 사용하게 된다.
execute((Action) () -> {});
```

> 형식 추론

동일하게 해당 람다의 시그니처를 추론하게 되면 (Context와 함수형 인터페이스를 통해서), 들어가는 람다에 들어가는 파라미터의 형식에도 접근할 수 있게 된다. 따라서 람다의 파라미터의 형식 또한 생략할 수 있다.

```java
List<Apple> greenApples = filter(inventory, apple -> GREEN.equals(apple.getColor()));

// filter함수가 (List<Apple> inventory, Predicate<Apple>)이므로, 저 Predicate를 통해서 대상 형식을 추론, 파라미터 또한 추론함.
```

> 지역 변수 사용

람다는 외부에서 사용하는 변수를 캡처링할 수 있다. 이를 **람다 캡처링**이라고 한다. (완전 클로저 느낌)

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```

그런데 중요한 점은, 인스턴스 변수와 지역 변수를 자유롭게 캡처링 할 수 있지만 지역변수에 한해서는 이것들이 `final`로 선언되어 있어야 하거나, final로 선언된 변수와 똑같이 사용 되어야 한다는 것이다.

즉, **한번만 할당할 수 있는 지역 변수를 캡쳐할 수 있다.**  
지역 변수는 스택에 저장, 해당 지역 변수를 사용하는 쓰레드가 사라져 변수 할당이 해제되면 접근할 수 없으므로 캡처링을 할 때 지역 변수의 **복사본**을 이용해서 진행한다. 따라서, 값이 달라지면 안되므로 final이어야 한다.

## 메서드 참조

메서드 정의를 재활용 해 람다처럼 전달 가능
`Apple::getWeight` 와 같은 문법 가능.

기존 메서드 구현을 이용하여 람다 표현식을 만들 수 있으며, 메서드명을 명시적으로 참조함으로서 가독성을 높인다.

만드는 방법은

1. 정적 메서드 참조
2. 다양한 형식의 인스턴스 메서드 참조
3. 기존 객체의 인스턴스 메서드 참조
   의 방법이 존재한다.

메서드 참조는 컨텍스트 형식과 일치해야 한다.

```java
List<String> str = Arrays.asList("a","b","A","B");
str.sort((s1, s2) -> s1.compareToIgnoreCase(s2));

str.sort(String::compareToIgonoreCase);
// String::compareToIgnoreCase의 시그니처가 Comparator의 시그니처와 맞아서 가능하다. (T,T) -> int
```
