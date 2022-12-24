# CHAPTER 5 : 스트림 활용

## 5.1 필터링

filter 메서드는 프레디케이트(boolean을 반환)를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환.

```java
List<Dish> vegetarianMenu = menu.stream().filter(Dish::isVegetarian).collect(toList());
```

스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct 메서드도 지원 (고유 여부는 객체의 hashCode, equals)

## 5.2 스트림 슬라이싱

스티림의 요소를 선택하거나 스킵하는 방법

### TAKEWHILE

filter 연산을 이용하면 전체 스트림을 반복하면서 각 요소에 프레디케이트를 적용

⇒ 따라서 리스트가 이미 정렬되어 있다면, takeWhile을 이용하면 해당 스트림을 슬라이스 할 수 있음

### DROPWHILE

dropWhile은 takeWhile과 정반대의 작업을 수행 ⇒ 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버림

### 요소 건너뛰기

스트림은 처음 n개 요소를 제외한 스트림을 반환하는 skip 메서드를 지원

## 5.3 매핑

스트림은 함수를 인수로 받는 map 메서드를 지원 ⇒ 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑

### flatMap

flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑 ⇒ 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행

## 5.4 검색과 매칭

### 프레디케이트가 적어도 한 요소와 일치하는지 확인

anyMatch 메서드를 이용

```java
if (menu.stream().anyMatch(Dish::isVegetarian)) {
	System.out.println("the menu is (somewhat) vegetarian friendly!!");
}
```

### 프레디케이트가 모든 요소와 일치하는지 검사

allMatch 메서드는 anyMatch와 달리 스트림의 모든 요소가 주어진 프레디케이트와 일치하는지 검사

```java
boolean isHealthy = menu.stream().allMatch(dish -> dish.getCalories() < 1000);
```

### NONEMATCH

noneMatch는 allMatch와 반대 연산을 수행. 즉, noneMatch는 주어진 프레디케이트와 일치하는 요소가 없는지 확인

```java
boolean isHealthy = menu.stream().noneMatch(d -> d.getCalories() >= 1000);
```

anyMatch, allMatch, noneMatch 세 메서드는 스트림 **쇼트서킷** 기법 ⇒ 자바의 &&, || 연산

### 요소검색

findAny 메서드는 현재 스트림의 임의의 요소를 반환 ⇒ 쇼트서킷을 이용해서 결과를 찾는 즉시 실행 종료

```java
Optional<Dish> dish = menu.stream().filter(Dish::isVegetarian).findAny();
```

### Optional

Optional<T> ⇒ 값의 존재나 부재 여부를 표현하는 컨테이너 클래스

findAny는 아무 요소도 반환하지 않을 수 있음. null은 쉽게 에러를 일으킬 수 있으므로 Optional<T>를 만듦

- isPresnt : Optional이 값을 포함하면 참, 아니면 false
- ifPresent (Consumer <T> block) : 값이 있으면 주어진 블록 실행 (void를 반환하는 람다를 전달할 수 없음)
- T get() : 값이 존재하면 값을 반환하고, 값이 없으면 NoSuchElementException을 일으킴
- T orElse(T other) : 값이 있으면 값을 반환. 없으면 기본값들 반환

```java
menu.stream().filter(Dish::isVegetarian).findAny(); // Optinal<Dish> 반환
```

## 5.5 리듀싱

모든 스트림 요소를 처리해서 값으로 도출하는 연산

### 요소의 합

reduce를 이용하면 애플리케이션의 방복된 패턴을 추상화 할 수 있음

```java
int sum = numbers.stream().reduce(0, (a,b) -> a+b);
```

reduce는 두 개의 인수를 가짐

- 초깃값 0
- 두 요소를 조합해서 새로운 값을 만드는 BinaryOperator<T>

### 초깃값 없음

초깃값을 받지 않도록 오버로드된 reduce, 그러나 이 reduce는 Optional 객체를 반환

```java
Optinal<Integer> sum = numbers.stream().reduce((a b) -> (a + b));
```

### 최댓값과 최솟값

reduce 연산은 새로운 값을 이용해서 스트림의 모든 요소를 소비할 때 까지 람다를 반복 수행하면서 최댓값을 생산한다.

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max); // 최댓값
Optional<Integer> min = numbers.stream().reduce(Integer::min); // 최솟값
```

## 5.6 실전 연습

문제들이라서 스킵합니다

## 5.7 숫자형 스트림

숫자 스트림을 효율적으로 처리할 수 있도록 **기본형 특화 스트림**을 제공

### 기본형 특화 스트림

IntStream, DoubleStream, LongStream을 제공

### 숫자 스트림으로 매핑

스트림을 특화 스트림으로 변환할 때는 mapToInt, mapToDouble, mapToLong 세가지 메서드를 가장 많이 사용, map과 정확히 같은 기능을 수행하지만, Stream<T> 대신 특화된 스트림을 반환

```java
int calories = menu.stream().mapToInt(Dish::getCalories).sum();
```

mapToInt는 IntStream을 반환하므로, IntStream 인터페이스에서 제공하는 sum 메서드를 사용할 수 있음

### 객체 스트림으로 복원

boxed 메서드를 이용해서 특화 스트림을 일반 스트림으로 변환할 수 있음

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); // 스트림을 숫자 스트림으로 변환
Stream<Integer> stream = intSteram.boxed(); // 숫자 스트림을 스트림으로 변환
```

### 기본값 : OptionalInt

OptioanalInt를 이용해서 IntStream의 최댓값 요소를 찾을 수 있음

```java
OptioanlInt maxCalories = menu.stream().mapToInt(Dish::getCalories).max();
int max = maxCalories.orElse(1); // 값이 없을 때 기본 최댓값을 명시적으로 설정
```

### 숫자 범위

range 메서드는 시작값과 종료값이 결과에 포함되지 않는 반면 rangeClosed는 시작값과 종료값이 결과에 포함됨

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100).filter(n -> n % 2 == 0);
System.out.println(evenNumbers.count()); // 1부터 100까지에는 50개의 짝수가 있음
```

## 5.8 스트림 만들기

일련의 값, 배열, 파일, 함수를 이용한 무한 스트림 만들기 등 다양한 방식으로 스트림을 만들 수 있음

### 값으로 스트림 만들기

임의의 수를 인수로 받는 정적 메서드 Stream.of를 이용해서 스트림을 만들 수 있음.

```java
Stream<String> stream = Stream.of("Modern ", "Java ", "In ", "Action");
stream.map(String::toUpperCase).forEach(System.out::println);
```

empty 메서드를 이용해서 스트림을 비울 수 있음

```java
Stream<String> emptyStream = Stream.empty();
```

### null이 될 수 있는 객체로 스트림 만들기

**자바 9** 에는 null이 될 수 있는 개체를 스트림으로 만들 수 있는 새로운 메소드가 추가

```java
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
```

### 배열로 스트림 만들기

배열을 인수로 받는 정적 메서드 Arrays.stream을 이용해서 스트림을 만들 수 있음

```java
int[] numbers = {2,3,5,6,11,13}
int sum = Arrays.stream(numbers).sum(); // 합계 41
```

### 파일로 스트림 만들기

java.nio.file.Files의 많은 정적 메서드가 스트림을 반환
예를 들어 Files.lines는 주어진 파일의 행 스트림을 문자열로 반환

```java
long uniqueWords = 0;
try(Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
	uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" "))).distinct().count;
} catch(IOException e) {
	// 파일 열기 예외
}
```

### 함수로 무한 스트림 만들기

Stream.iterate와 Stream.generate를 제공. 두 연산을 이용해서 무한 스트림, 고정된 컬렉션에서 고정된 크기로 스트림을 만들었던 것과 달리 크기가 고정되지 않은 스트림을 만들 수 있음.

- iterate : iterate는 요청할 때마다 값을 생산, 끝이 없으므로 무한 스트림을 만듬
- generate :generate는 Supplier<T>를 인수로 받아서 새로운 값을 생산