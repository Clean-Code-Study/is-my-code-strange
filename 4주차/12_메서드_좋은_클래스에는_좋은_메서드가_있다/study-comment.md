# 12장 메서드(함수): 좋은 클래스에는 좋은 메서드가 있다

### 12.1 반드시 현재 클래스의 인스턴스 변수 사용하기

- 인스턴스 변수를 안전하게 조작하도록 메서드를 설계하면 클래스 내부가 정상적인 상태인지 보장할 수 있다.
- 예외가 있기는 하지만 메서드는 반드시 현재 클래스의 인스턴스 변수를 사용하도록 설계해야한다.

---

### 12.2 불변을 활용해서 예상할 수 있는 메서드 만들기

- 가변 인스턴스 변수 등을 변경하는 메서드는 의도하지 않게 다른 부분에 영향을 줄 수 있다.
- 이렇게 되면 예상하지 못한 동작이 발생할 수 있으며 유지보수가 어려워진다.
- 따라서 불변을 활용해 예상치 못한 동작 자체를 막을 수 있게 설계해야한다.

---

### 12.3 묻지 말고 명령하라

- 어떤 클래스가 다른 클래스의 상태를 판단하거나, 상태에 따라 값을 변경하는 등 ‘다른 클래스를 확인하고 조작하는 메서드 구조’는 응집도가 낮은 구조이다.
- 인스턴스 변수의 값을 추출하는 메서드를 getter, 값을 설정하는 메서드를 setter라고 부른다.

```java
public class Person {
	private String name;
	
	// getter
	public String getName() {
		return name;
}

	//setter
	public void setName(String newName) {
		name = newName;
	}
```

- getter/setter는 ‘다른 클래스를 확인하고 조작하는 메서드 구조’가 되기 쉽다.
- 일반적으로 개발 생산성이 좋지 않은 소프트웨어의 소스 코드에서 자주 볼 수 있다.

---

### 12.4 커맨드 / 쿼리 분리

- 아래 메서드는 상태 변경과 추출을 동시에 하고 있다.

```java
int gainAndGetPoint {
	point += 10;
	return point;
}
```

- 상태 변경과 추출을 동시에 하는 메서드는 여러 문제의 원인이 되고 사용자가 쓰기 힘든 메서드가 된다.
- 예를 들어 추출만 하고 싶거나 변경만 하고 싶을 때 지원하지 못한다.
- 커맨드•쿼리 분리라는 패턴은 ‘메서드는 커맨드 또는 쿼리 중에 하나만 하도록 설계해야 한다는 패턴이다.

| 메서드 종류 구분 | 설명 |
| --- | --- |
| 커맨드 | 상태를 변경하는 것 |
| 쿼리 | 상태를 리턴하는 것 |
| 모디파이어 | 커맨드와 쿼리를 동시에 하는 것 |
- 현재 코드 12.2의 gainAndGetPoint는 커맨드와 쿼리를 동시에 하는 모디파이어이다.
- 상황에 따라 모디파이어로 만들 수밖에 없는 메서드도 있지만, 그것은 예외이고 최대한 피하는 것이 좋다.
- 커맨드•쿼리 분리 패턴에 따라 gainAndPoint를 커맨드와 쿼리로 분리하면 코드가 굉장히 단순해진다.

```java
/**
* 포인트를 증가(커맨드)
*/
void gainPoint() {
	point += 10;
}

/**
* 포인트를 리턴(쿼리)
* @return 포인트
*/
int getPoint() {
	return point;
}
```

---

### 12.5 매개변수

- 매개변수는 입력 값으로 사용한다. 아래는 매개변수 사용 시 주의점이다.

**12.5.1 불변 매개변수로 만들기**

- 매개변수를 변경하면 값의 의미가 바뀌어 어떤 의미를 나타내는지 유추하기 어렵다.
- 또한 어디서 변경되었는지 찾기도 힘들다.
- 매개변수에 final 수식자를 붙여 불변으로 만들어라
- 매개변수를 변경하고 싶으면 불변 지역 변수를 만들고 여기에 변경 값을 할당하는 형태로 구현해라

**12.5.2 플래그 매개변수 사용하지 않기**

- 플래그 매개변수를 받는 메서드는 코드를 읽는 사람이 메서드가 무슨 일을 하는지 알기 어렵다.
- 무슨일을 하는지 알기 위해서는 메서드 내부 로직을 확인해야 하기때문에 가독성이 낮아진다.
- 전략 패턴을 사용하는 등, 다른 구조로 설계를 개선해라

**12.5.3 null 전달하지 않기**

- null을 활용하는 로직은 NullPointException이 발생할 수 있으며, null을 확인해야 하기때문에 로직이 복잡해지는 문제를 가져온다.
- 매개변수로 null을 전달하지 않도록 설계해라
- null을 전달하지 않게 설계하려면 null에 의미를 부여해서는 안된다.
- EMPTY를 사용하여 구현하라

**12.5.4 출력 매개변수 사용하지 않기**

- 출력 매개변수를 사용하면 응집도가 낮은 구조가 만들어진다.
- 매개변수는 입력 값으로 사용하는 것이 본이다.

**12.5.5 매개변수는 최대한 적게 사용하기**

- 매개변수는 최대한 적게 설계하는 것이 좋다.
- 메서드에 매개변수가 많다는 것은 메서드가 여러 가지 기능을 처리한다는 의미이다.
- 메서드가 처리할 것이 많아지면 그만큼 로직이 복잡해져 이는 다양한 악마들을 불러온다.
- 매개변수가 많아질 것 같으면 별도의 클래스를 만들어 사용하는 것을 기억해라

### **12.6 리턴 값**

- 리턴 값을 설계할 때 주의할 점들이 있다.

**12.6.1 ‘자료형’을 사용해서 리턴 값의 의도 나타내기**

- 아래 코드의 Price.add 메서드는 가격을 리턴하고 있다. 근데 가격의 자료형이 int이다.

```java
class Price {
	//생략
	int add(final Price other) {
		return amount + other.amount;
	}
}
```

- int 자료형처럼 단순한 기본 자료형으로는 리턴한 값의 의미를 호출하는 쪽에 전달할 수 없다.
- 왜 그럴까? 아래 코드를 살펴보자

```java
	int price = productPrice.add(otherPrice); //상품 가격 합계
	int discountedPrice = calcDiscountedPrice(price); //할인 금액
	int deliveryPrice = calcDeliveryPrice(discountedPrice); //배송비
```

- productPrice는 Price 자료형이며, add 메서드로 int 자료형의 가격을 리턴한다.
- 그런데 가격뿐만 아니라 할인 금액과 배송비까지 모두 int 자료형을 사용하고 있다.
- 금액을 계산할 때는 어떤 가격이 다른 가격 계산에 사용되는 등, 다양한 종류의 가격을 함께 다루는 경우가 많다.
- int 자료형을 리턴하게 만들면, 어떤 값이 어떤 금액을 의미하는지 알기 힘들기 때문에 매개변수를 잘못 전달하는 실수가 발생할 수 있다.

```java
// 배송 수수료 DeliveryCharge에는 배송비가 전달되어야 하는데
// 상품 가격 합계가 전달되고 있음.
DeliveryCharge deliveryCharge = new DeliveryCharge(price);
```

- 따라서 기본 자료형을 사용하지 말고, 독자적인 자료형을 사용해서 의도를 명확하게 하는 것이 좋다.
- 예를 들어 아래의 코드에서 add 메서드는 Price 자료형을 리턴한다. 가격을 리턴한다는 의도가 명확해 진 것이다.

```java
class Price {
	// 생략
	Price add(final Price other) {
		final int added = amount + other.amount;
		return new Price(added);
	}
}
```

- 마찬가지로 다른 금액 계산에서도 독자적인 자료형을 사용하면, 의도를 보다 명확하게 나타낼 수 있다.
- 자료형 자체가 다르기 때문에 실수로 값을 섞어서 계산하면 컴파일 오류가 발생할 것이다.

```java
Price price = productPrice.add(otherPrice);
DiscountedPrice discountedPrice = new DiscountedPrice(price);
DeliveryPrice deliveryPrice = new DeliveryPrice(discountedPrice);
```

**12.6.2 null 리턴하지 않기**

- 매개변수로 null을 전달하지 않는 것이 좋듯이 null을 리턴하지 않아야 좋다.

**12.6.3 오류는 리턴 값으로 리턴하지 말고 예외 발생시키기**

- 아래 코드는 문제가 있는 오류 처리이다.

```java
// 위치를 나타내는 클래스
class Location {
	// 생략
	
	// 위치 이동하기
	Location shift(final int shiftX, final int shiftY) {
		int nextX = x + shiftX;
		int nextY = y + shiftY;
		if (valid(nextX, nextY)) {
			return new Location(nextX, nextY);
		}
		// (-1, -1)은 오류 값
		return new Location(-1, -1);
	}
```

- 위 코드의 Location.shift는 위치를 이동시키는 메서드이다.
- 그런데 이동 후의 좌표가 잘못된 경우, 오류 값으로 Location(-1, -1)을 리턴한다.
- 이러한 구현은 호출하는 쪽에서 ‘오류가 있을 때, 오류 값으로 Location(-1, -1)을 리턴한다.’라는 사실을 알고 있어야 하고 이를 활용해서 오류 처리를 확실하게 구현해야 한다.
- 만약 오류 처리를 잊으면 Location(-1, -1)이라는 값이 후속 로직에서 정상 값처럼 사용되어 버그가 생길 것이다.
- Location(-1, -1)을 좌표가 아니라 오류로 다루는 것을 중의적이라고 할 수 있다.
- 중의적인 것은 사람에게 혼란을 준다. 또한 상황 판단을 위해 조건 분기를 사용해야 하므로 여기저기 조건분기가 늘어난다.
- 따라서 중의적으로 해석될 만한 코드는 피하는 것이 좋다.
- 그러므로 오류 값을 리턴하지 말고 예외를 발생시켜야한다.

```java
// 위치를 나타내는 클래스
class Location {
	//생략
	
	Location(final int x, final int y) {
	if (!valid(x, y)) {
	 throw new IllegalArgumentException("잘못된 위치입니다.");
	}
	
	this.x = x;
	this.y = y;
}

// 위치 이동하기
Location shift(final int shiftX, final int shiftY) {
	int nextX = x + shiftX;
	int nextY = y + shiftY;
	
	return new Location(nextX, nextY);
}
```
