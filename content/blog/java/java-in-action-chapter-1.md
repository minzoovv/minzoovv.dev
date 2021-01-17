---
title: Modern Java In Action - (1) 자바 8,9,10 무슨 일이 일어나고 있나?
date: 2020-07-27 22:01:48
category: java
draft: false
---

## 자바 8의 새로운 기술

- 스트림 API  
  질의 언어에서 표현식을 처리하는 것처럼 병렬연산 지원  
  `synchronized` 키워드를 사용하지 않아도 된다.

### Synchronized

자바 동기화 블록으로, 같은 객체에 대한 모든 동기화 블록은 한 시점에 한 쓰레드만이 블록 안으로 접근하도록 (실행하도록) 한다.

예를 들어서 한 클래스 안에 synchronized 키워드를 가진 메서드가 여러개 있다면, 한 쓰레드는 한 시점에 두 동기화 메서드 중 하나만 실행할 수 있다고 한다.

4가지 유형의 블록 존재.

1. 인스턴스 메소드 : 인스턴스 당 한 쓰레드
2. 스태틱 메소드 : 클래스 당 한 쓰레드 (JVM안에서 클래스 객체는 클래스 당 하나만 존재할 수 있음, 같은 클래스에 대해서는 오직 한 쓰레드만 동기화된 스태틱 메소드 실행 가능)

아래 두개는 메서드의 일부분만 동기화 처리, 이게 효율적일 수 있음

3. 인스턴스 메소드 코드블록

```java
 public void add(int value){
    synchronized(this){
       this.count += value;
    }
  }
```

이는 인스턴스 메서드 선언이랑 동일하다. Sync 부분이 메서드의 전문이므로.

4. 스태틱 메소드 코드블록
   동일하게 작용하지만, 클래스 객체를 동기화 기준으로 잡는다.

```java
  public class MyClass {

    public static synchronized void log1(String msg1, String msg2){
       log.writeln(msg1);
       log.writeln(msg2);
    }

    public static void log2(String msg1, String msg2){
       synchronized(MyClass.class){
          log.writeln(msg1);
          log.writeln(msg2);
       }
    }
```

이 경우에는 MyClass.class가 아닌 다른 객체를 전달한다면 쓰레드가 동기에 각 메서드를 실행시킬 수 있다.

Synchronized는 그 함수가 포함하는 this(객체에) 락을 건다.
인자 값이 락을 걸 대상이 된다.

아래와 같이 하면 다른 object를 기반으로 락을 걸 수 있다.

```java
public class SyncBlock2 {
	private HashMap<String, String> mMap1 = new HashMap<>();
	private HashMap<String, String> mMap2 = new HashMap<>();

	private final Object object1 = new Object();
	private final Object object2 = new Object();

	public static void main(String[] agrs) {
		SyncBlock2 syncblock2 = new SyncBlock2();
		System.out.println("Test start!");
		new Thread(() -> {
			for (int i = 0; i<10000; i++) {
				syncblock2.put1("A","B");
				syncblock2.get2("C");
				}
		}).start();

		new Thread(() -> {
			for (int i = 0; i<10000; i++) {
				syncblock2.put2("C","D");
				syncblock2.get1("A");
				}
		}).start();
		System.out.println("Test end!");

	}

	public void put1(String key, String value) {
		synchronized(object1) {
			mMap1.put(key, value);
		}
	}

	public String get1(String key) {
		synchronized(object1) {
			return mMap1.get(key);
		}
	}

	public void put2(String key, String value) {
		synchronized(object2) {
			mMap2.put(key, value);
		}
	}

	public String get2(String key) {
		synchronized(object2) {
			return mMap2.get(key);
		}
	}
}

```

근데 이게 비용이 비싸다네 … 특히 멀티코어에서는 더!!!  
[투덜이의 리얼 블로그 :: Java의 동기화 Synchronized 개념 정리#1](https://tourspace.tistory.com/54)

<br>

---

이를 개선하게 되는게 Stream, 스트림 때문에 아래 두개의 기능이 생겨났다고 볼수도 있다고 하지만, 각각의 기능도 엄청난 편의성을 제공해준다.

- 메서드에 코드 전달
- 인터페이스의 디폴트 메서드

## Stream API

Stream: 한번에 한개씩 만들어지는 연속적인 데이터 항목들의 모임  
`Stream<T> -> T` 형식으로 구성된 일련의 항목, 그리고 이런 항목을 연속으로 제공하고 이 항목들에 내가 실행하고 싶은 것들을 먹일 수 있다.  
추가로 이 각각의 실행을 병렬로 처리할 수 있다. 자동으로 여러 CPU 코어에 쉽게 할당도 가능하다.  
따라서 쓰레드를 사용하지 않으면서 병렬성도 얻을 수 있다.

## 동작 파라미터화로 메서드 코드에 전달하기

코드의 일부를 API로 전달하는 기능 -> 한 메서드의 인자로 다른 메서드를 넣을 수 있다.  
이러한 개념이 **동작 파라미터화**이다.

## 병렬성과 공유 가변 데이터

공유되는 가변 데이터가 잇으면 병렬성에 많은 오류가 발생  
이런 공유 가변 데이터를 없애는 방향으로 코딩해야하며, 스트림을 사용하면 순수함수만을 넘겨야한다.

## 자바함수

자바에서는 클래스, 메서드는 2급시민이었다.

### 일급객체

- 변수나 데이터에 할당 할수 있는 것
- 객체를 인자로 넘길 수 있는 것
- 리턴 값으로 리턴할 수 있는 것

파이썬이나 자바스크립트는 메서드를 일급객체로 활용할 수 있었다.  
하지만 자바는 못그랬다. 하지만 이제는 바뀌었다!

1. 기존에는 메서드를 넘겨주려면 객체 참조를 생성해서 인스턴스를 만든 후 내부에서 해당 메서드를 실행해줬어야했는데, 이제는 객체의 메서드 자체를 메서드 참조를 이용하여 넘길 수 있다.

   Predicate : 수학에서 인수로 값을 받아 true, false를 반환하는 함수, interface로 정의되어 있음. Java.util.function에 있음.

2. 스트림 메서드가 제공해주는 함수들이 파이썬에서 기본 제공해주는 함수들과 거의 비슷하다. 다른 점은 해당 스트림 객체가 map, reduce, collect와 같은 함수를 가지고 있다는 점. (python은 map(function, list)와 같이 대상 이터레이터를 던져줘야 한다. 그리고 저런 함수가 기본으로 라이브러리에서 제공해주는 함수들이다) 과거 컬렉션 API에서는 이런걸 제공해주지 않았지만, Stream API에서는 편안하게 가능하다. 추가적으로 병렬 코어 사용도 알아서 해주기 때문에 더 이득이다.

3. for문을 외부에서 작성하지 않고, 함수 자체만으로 iteration을 돌리는 것을 내부 반복이라고 한다.

## 디폴트 메서드와 자바모듈

기존에는 인터페이스에 디폴트 메서드가 없었는데, 기본적으로 가지고 있는 메서드를 구현할 수 있도록 하였다.  
**default** keyword를 이용해서 인터페이스 내에 구현해줄 수 있다. 이렇게 되면 이 인터페이스를 사용하는 클래스에서 해당 인터페이스를 구현하지 않아도 된다.

근데 하나의 클래스에서는 여러 인터페이스 구현 가능 -> 다중 디폴트 메서드 존재 -> 다중 상속의 허용?  
엄밀히 말하면 그렇다고 한다. (java에서는 원래 다중상속을 허용하지 않았다)

## Optional과 Pattern matching

Optional<T> Generic을 통해서 NullPointer Exception을 잡아주었다.  
Java 8에는 패턴 매칭 기법도 있지만, 완벽하지 않고 제안만 된 상태이다.
