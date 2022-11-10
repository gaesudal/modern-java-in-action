## 2.1 변화하는 요구사항에 대응하기

```java
// 녹색 사과 필터링
// 사과 색을 정의
enum Color { RED, GREEN }

// 첫번째 시도 결과 코드
public static List<Apple> filterGreenApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>(); // 사과 누적 리스트
	for (Apple apple : inventory) {
		if (GREEN.eqauls(apple.getColor())) { // 녹색 사과만 선택
				result.add(apple);
		}
	}
	return result;
}
```

```java
// 색을 파라미터화
// 두번째 시도 결과 코드
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
	List<Apple> result = new ArrayList<>(); 
	for (Apple apple : inventory) {
		if (apple.getColor().eqauls(color)) {
				result.add(apple);
		}
	}
	return result;
}

///////////////////////////////////////////////////////////////
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

```java
// 무게를 파라미터화
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
	List<Apple> result = new ArrayList<>(); 
	for (Apple apple : inventory) {
		if (apple.getWeight() > weight) {
				result.add(apple);
		}
	}
	return result;
}
```

```java
// 가능한 모든 속성으로 필터링
// 세번째 시도 결과 코드
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
	List<Apple> result = new ArrayList<>(); 
	for (Apple apple : inventory) {
		if ((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) {
				result.add(apple);
		}
	}
	return result;
}

///////////////////////////////////////////////////////////////
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```

세번째 시도 결과코드에서 true와 false는 뭘 의미하는지 알 수 없음

앞으로 요구사항이 바뀌었을 때 유연하게 대응할 수도 없음

filterApples에 어떤 기준으로 사과를 필터링할 것인지 효과저긍로 전달할 수 있다면 더 좋을 것 → **동작 파라미터화**

## 2.2 동작 파라미터화

참 또는 거짓을 반환하는 함수를 **프레디케이트**라고 한다

```java
// 선택 조건을 결정하는 인터페이스
public interface ApplePredicate {
	boolean test (Apple apple);
}

// 선택 조건을 대표하는 여러 버전의 ApplePredicate
public class AppleHeavyWeightPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) { 
        return apple.getWeight() > 150; // 무거운 사과만 선택
    }
}

public class AppleGreenColorPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return GREEN.equals(apple.getColor()); // 녹색 사과만 선택
    }
}
```

ApplePredicate는 사과 선택 전략을 캡슐화 → 이를 strategy design pattern 이라 한다

인터페이스가 알고리즘 패밀리고 구체화 한것들이 전략

```java
// 추상적 조건으로 필터링
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory) {
        if (p.test(apple)) { // 프레디케이트 객체로 사과 검사 조건 캡슐화
            result.add(apple);
        }
    }
    return result;
}

// filterApples 메서드의 동작 파라미터화
public class AppleRedAndHeavyPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor()) && apple.getWeight() > 150;
    }
}

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```

## 2.3 복잡한 과정 간소화

자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있는 **익명 클래스**를 제공한다

익명 클래스는 자바의 지역 클래스와 비슷한 개념, 익명 클래스를 이용하면 클래스 선언과 인스턴스화를 동시에 할 수 있다

```java
List<Apple> redApples = fileterApples(inventory, new ApplePredicate() { // filterApples 메서드의 동작을 직접 파라미터화
    @Override
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
});
```

동작의 파라미터화를 람다를 통하여 코드를 간결하게 정리가 가능

```java
List<Apple> result = fileterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

```java
// 리스트 형식으로 추상화
public static <T> List<T> filter(List<T> list, Predicate<T> p) { // 형식 파라미터 T 등장
    List<T> result = new ArrayList<>();
    for (T e: list) {
        if (p.test(e)) {
            result.add(e);
        }
    }
    return result;
}

List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

## 2.4 실전 예제

```java
// java.util.Comparator
public interface Comparator<T> {
	int compare(T o1, T o2);
}

// 익명 클래스를 이용해서 무게가 적은 순서로 목록 정렬
inventory.sort(new Comparator<Apple>() {
	pulbic int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
});

inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```