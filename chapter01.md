## 1.1 역사의 흐름은 무엇인가?

자바 8에서는 병렬 실행을 새롭고 단순한 방식으로 접근할 수 있는 방법을 제공

간결한 코드, 멀티코어 프로세서의 쉬운 활용이라는 두 가지 요구사항을 기반

- 스트림 API : 병렬 연산을 지원하는 API
- 메서드에 코드를 전달하는 기법 : 메서드 참조와 람다
- 인터페이스의 디폴트 메서드

## 1.2 왜 아직도 자바는 변화하는가?

### 스트림 처리

스트림이란 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임

간단히, 조립 라인처럼 어떤 항목을 연속으로 제공하는 어떤 기능 - 파이프라인

### 동작 파라미터화

코드 일부를 API로 전달하는 기능

자바 8 이전의 자바에서는 메서드를 다른 메서드로 전달할 방법이 없었음

메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공 : 동작 파라미터화

### 병렬성과 공유 가변 데이터

다른 코드와 동시에 실행하더라도 안전하게 실행할 수 있는 코드를 만들려면 공유된 가변 데이터에 접근하지 않아야함

- 순수 함수 (pure)
- 부작용 없는 함수 (side-effect-free)
- 상태 없는 함수 (stateless)

synchronized를 이용할 수 있지만, 보통 시스템 성능에 악영향을 미침

## 1.3 자바 함수

프로그램의 핵심은 값을 바꾸는것 : [일급시민](https://ko.wikipedia.org/wiki/일급_객체)

인스턴스화한 결과가 값으로 귀결되는 클래스를 정의할 때 메서드를 아주 유용하게 활용할 수 있지만 메서드와 클래스는 자체 값이 될 수 없음 : 이급시민

**클래스와 같은 이급시민을 일급시민으로 바꾸기 위해**

### 메서드 참조

```java
// 디렉터리에서 모든 숨겨진 파일을 필터링
// 파일이 숨겨져 있는지 여부를 알려주는 메서드 구현
// File -> isHidden 메서드 제공

//기존
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
	public boolean accept(File file) {
		return file.isHidden();
	}
)};

// 각 행이 무슨 일을 하는지 투명하지 않음

// 자바 8
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```

### 람다 : 익명 함수

이용할 수 있는 편리한 클래스나 메서드가 없을 때 새로운 람다 문법을 이용하면 더 간결하게 코드를 구현할 수 있음

```java
(int x) -> x + 1
```

### 코드 넘겨주기

```java
// Apple 클래스와 getColor 메서드가 있고, Apples 리스트를 포함하는 변수 inventory가 있음
// 모든 녹색 사과를 선택해서 리스트를 반환하는 프로그램을 구현
// 특정 항목을 선택해서 반환하는 동작 : 필터

// 자바 8 이전
// 사과를 녹색으로 필터링
public static List<Apple> fileterGreenApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>();

	for (Apple apple : inventory) {
		if (GREEN.equals(apple.getColor()) {
			result.add(apple);
	}
}

// 사과를 무게로 필터링 하고 싶을 때
public static List<Apple> fileterHeavyApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>();

	for (Apple apple: inventory) {
		if (apple.getWeight() > 150) {
			result.add(apple);
	}
}

// 자바 8
public static boolean isGreenApple(Apple apple) {
	return GREEN.equals(apple.getColor());
}

public static boolean isHeavyApple(Apple apple) {
	return apple.getWeight() > 150;
}

public interface Predicate<T>{
	boolean test(T t);
}

static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple: inventory) {
		if (p.test(apple)) {
			result.add(apple);
		}
	}
	return result;
}

// 메서드 호출 시
filterApples(inventory, Apple::isGreenApple);
filterApples(inventory, Apple::isHeavyApple);
```

### 메서드 전달에서 람다로

```java
// 한두 번만 사용할 메서드를 매번 정의하는 것은 귀찮은 일
filterApples(inventory, (Apple a) -> GREEN.equals(a.getColor()));
filterApples(inventory, (Apple a) -> a.getWeight() > 150);

// 같이 구현할 수도 있음
filterApples(inventory, (Apple a) -> GREEN.equals(a.getColor()) || RED.equals(a.getColor()));
```

## 1.4 스트림

```java
// 리스트에서 고가의 트랜잭션만 필터링한 다음에 통화로 결과를 그룹화 → 많은 기본코드 구현 (for ~ if ~ if)
// 통화별로 트랜잭션을 그룹화한 코드
Map<Currency, List<Transaction>> transactionByCurrencies = new HashMap<>();

for(Transaction transaction : transactions){
  Currency currency = transaction.getCurrency();
  List<Transaction> transactionForCurrency = transactionByCurrencies.get(currency);

  if(transactionForCurrency == null){
    transactionForCurrency = new ArrayList<>();
    transactionByCurrencies.put(currency, transactionForCurrency);
  }

  transactionForCurrency.add(transaction);
}

// 중첩된 제어 흐름 문장이 많아 이해하기 어려움
// 스트림 API를 이용하여 해결
Map<Currency, List<Transaction>> transactionByCurrencies = 
	transactions.stream().filter((Transaction t) -> t.getPrice() > 1000).collect(groupingBy(Transaction::getCurrency));
```

이전 자바 버전에서 제공하는 스레드 API로는 멀티스레딩 코드를 구현해서 병렬성을 이용하는 것은 쉽지 않음

자바 8은 스트림 API로 ‘컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드 문제, 멀티코어 활용 어려움’을 모두 해결

- 데이터의 필터링
- 데이터의 추출
- 데이터의 그룹화

```java
// 순차 처리 방식
List<Apple> heavyApples = 
	inventory.stream().filter((Apple a) -> a.getWeight() > 150).collect(toList());

// 병렬 처리 방식
List<Apple> heavyApples = 
	inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150).collect(toList());
```

## 1.5 디폴트 메서드와 자바 모듈

디폴트 메서드를 이용하면 기존의 코드를 건드리지 않고도 원래의 인터페이스 설계를 자유롭게 확장할 수 있음

```jsx
default void sort(Comparator<? super E> c) {
	Collections.sort(this, c);
}
```

## 1.6 함수형 프로그래밍에서 가져온 다른 유용한 아이디어