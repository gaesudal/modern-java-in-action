## 3.1 람다란 무엇인가?

**람다 표현식**은 메서드로 전달할 수 있는 익명 함수를 단순화한 것

- 익명 : 보통의 메서드와 달리 이름이 없으므로 익명이라 표현
- 함수 : 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부름. 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함
- 전달 : 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있음
- 간결성 : 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없음

```java
// 기존 코드
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
            return a1.getWeight().compareTo(a2.getWeight());
        }
};

// 람다 이용 코드
Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2,getWeight());
```

## 3.2 어디에, 어떻게 람다를 사용할까?

함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있음

함수형 인터페이스는 정확히 하나의 추상 메서드를 지정하는 인터페이스 EX) Comparator, Runnable, Prediate ..

람다 표현식의 시그니처를 서술하는 메서드를 **함수 디스크럽터**라고 부름

예를 들어 Runnable 인터페이스의 유일한 추상 메서드 run은 인수와 반환값이 없으므로 Runnable 인터페이스는 인수와 반환값이 없는 시그니처로 생각할 수 있음

## 3.3 람다 활용 : 실행 어라운드 패턴

자원 처리에 사용하는 순환 패턴은 자원을 열고, 처리한다음에, 자원을 닫는 순서로 이루어진다. 즉, 실제 자원을 처리하느 코드를 설정과 정리 두 과정이 둘러 싸는 형태를 가짐

```java
public Stirng processFile() thorws IOException {
	try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
		return br.readLine(); // 실제 필요한 작업을 하는 행
	}
}
```

1단계 : 이 코드는 파일에서 한 번에 한 줄만 읽을 수 있음. 한번에 두 줄을 읽거나 사용되는 단어를 반환하려면, 기존의 설정, 정리 과정은 재사용하고  processFile 메서드만 다른 동작을 수행하도록 명령할 수 있다면 좋을 것 → processFile의 동작을 파라미터화

```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

2단계 : 함수형 인터페이스 자리에 람다를 사용할 수 있음. 따라서 BufferedReader → String과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야함

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
	String process(BufferedReader b) thorws IOException;
}

// 정의한 인터페이스를 processFile 메서드의 인수로 전달
public String processFile(BufferedReaderProcessor p) throws IOException {
	....
}
```

3단계 : 람다 표현식으로 함수형 인터페이스의 추상 메서드를 구현을 직접 전달할 수 있으며 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
	try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
		return p.process(br); // BufferReader 객체처리
	}
}
```

4단계 : 람다를 이용하여 다양한 동작을 processFile 메서도르 전달

```java
// 한 행을 처리
String oneLine = processFile((BufferedReader br) -> br.readLine());
// 두 행을 처리
String twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

## 3.4 함수형 인터페이스 사용

- Predicate : Predicate<T> 인터페이스는 제네릭 형식 T의 객체를 인수로 받아 불리언을 반환
- Consumer : Consumer<T> 인터페이스는 제네릭형식 T 객체를 받아서 void를 반환
- Function : Function<T, R> 제네릭 형식 T를 인수로 받아서 제네릭 R 객체를 반환

## 3.5 형식 검사, 형식 추론, 제약

람다가 사용되는 콘텍스트를 이용해서 람다의 형식을 추론할 수 있음

어떤  콘텍스트에서 기대되는 람다 표현식의 형식을 ********대상형식********이라 부름

대상형식이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있음 EX) Callable, PrivilegedAction

## 3.6 메서드 참조

```java
// 기존코드
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
// 메서드 참조
inventory.sort(comparing(Apple::getWeight)); 
```

메서드 참조의 3가지

- 정적 메서드 참조 : Integer::parseInt
- 다양한 형식의 인스턴스 메서드 참조 : String::length
- 기존 객체의 인스턴스 메서드 참조 : Transaction 객체를 할당받은 expensiveTransaction 지역 변수가 있고 Transaction 객체에는 getValue 메서드가 있다면, expensiveTransaction::getValue

## 3.7 람다, 메서드 참조 활용하기

1단계 : 코드전달

```java
public class AppleComparator implements Comparator<Apple> {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
}
inventory.sort(new AppleComparator());
```

2단계 : 익명 클래스 사용

```java
inventory.sort(new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
});
```

3단계 : 람다 표현식 사용

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));

Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());

inventory.sort(comparing(apple -> apple.getWeight()));
```

4단계 : 메서드 참조 사용

```java
inventory.sort(comparing(Apple::getWeight));
```

4단계 : 메서드 참조 사용

```java
inventory.sort(comparing(Apple::getWeight));
```