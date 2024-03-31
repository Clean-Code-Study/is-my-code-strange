# 5장 응집도: 흩어져 있는 것들

### 5.1 static 메서드 오용

```java
// 주문을 관리하는 클래스
class OrderManager {
	static int add(int moneyAmount1, int moneyAmount2) {
		return moneyAmount1 + moneyAmount2;
	}
}
```

- 금액을 더하는 add 메서드는 static 메서드로 정의되어 있기 때문에 클래스의 인스턴스를 생성하지 않고도 add 메서드를 호출할 수 있다.

```java
// moneyData1, moneyData2는 데이터 클래스
moneyData1.amount = OrderManager.add(moneyData1.amount, moneyData2.amount);
```

- 이렇게 static 메서드는 데이터 클래스와 함께 사용하는 경우가 많다.
- 이러한 구조의 문제는 데이터는 MoneyData에 있고, 데이터를 조작하는 로직은 OrderManager에 있다는 것이 문제이다.
  - 데이터와 로직이 서로 다른 클래스에 작성됨

**5.1.1. static 메서드는 인스턴스 변수를 사용할 수 없음**

- static 메서드는 인스턴스 변수를 사용할 수 없다.
- 따라서 어떤 메서드를 static 메서드로 만든 시점에서 이미 데이터와 데이터를 조작하는 로직 사이에 괴리가 생기고 응집도가 낮아질 수 밖에 없다.

**5.1.2. 인스턴스 변수를 사용하는 구조로 변경하기**

- 인스턴스 변수와 인스턴스 변수를 사용하는 로직을 같은 클래스에 만드는 것이 응집도는 높이는 방법이다.

**5.1.3. 인스턴스 메서드인 척하는 static 메서드 주의하기**

```java
class PaymentManager {
	private int discountRate; // 할인율
	
	// 생략
	int add(int moneyAmount1, int moneyAmount2){
		return moneyAmount1 + moneyAmount2;
	}
}
```

- add 메서드는 인스턴스 메서드이다.
- 하지만 인스턴스 변수 discountRate를 전혀 사용하지 않는다.
- static 키워드고 붙어 있지 않을 뿐, 같은 문제를 갖고 있는 것이다.

**5.1.4. 왜 static 메서드를 사용할까?**

- C 언어와 같은 절차 지향 언어에서는 데이터와 로직이 따로 존재하도록 설계한다.
- 이런 접근 방식을 객체 지향 언어에 적용하여 설계하기 위해 static 메서드를 활용하는 것이다.

**5.1.5. 어떠한 상황에서 static 메서드를 사용해야 좋을까?**

- 응집도의 영향을 받지 않는 경우 사용
- 예) 로그 출력 전용 메서드, 포맷 변환 전용 메서드 등

### 5.2 초기화 로직 분산

```java
class GiftPoint {
	private static final int MIN_POINT = 0;
	final int value;
	
	GiftPoint(final int point) {
		value = point;
	}
}
```

- 생성자를 public으로 만들면, 결과적으로 관련된 로직이 분산되기 때문에 유지 보수하기 힘들어진다.

```java
GiftPoint standardMembershipPoint = new GiftPoint(3000);
```

- 예를 들어, 표준 회원으로 가입했을 때 3000포인트를 제공하는 코드가 있다고 가정한다.
- 여기서 회원가입 포인트를 변경하고 싶을 때, 소스 코드 전체를 확인해야 한다.

**5.2.1. private 생성자 + 팩토리 메서드를 사용해 목적에 따라 초기화하기**

```java
class GiftPoint {
	private static final int MIN_POINT = 0;
	private static final int STANDARD_MEMBERSHIP_POINT = 3000;
	final int value;
	
	private GiftPoint(final int point) {
		if(point < MIN_POINT) {
			throw new IllegalArgumentException("포인트를 0 이상 입력해야 합니다.");
		}
	
		value = point;
	}
	
	static GiftPoint forStandardMembership() {
		return new GiftPoint(STANDARD_MEMBERSHIP_POINT);
	}
}
```

- 생성자를 private로 만들면, 클래스 내부에서만 인스턴스를 생성할 수 있다.
- 인스턴스를 생성하기 위한 static 팩토리 메서드에서 생성자를 호출한다.
- 이렇게 만들면, 신규 가입 포인트와 관련된 로직이 GiftPoint 클래스에 응집된다.

```java
GiftPoint standardMembershipPoint = GiftPoint.forStandardMembership();
```

**5.2.2. 생성 로직이 너무 많아지면 팩토리 클래스를 고려해 보자**

- 생성 로직이 너무 많아지면 해당 클래스가 하는 일이 불분명해진다.
- 그럴땐 생성 전용 팩토리 클래스를 분리하는 방법을 고려하는 것이 좋다.

### 5.3 범용 처리 클래스(Common/Util)

- 똑같은 일을 수행하는 코드가 많아지면 코드를 재사용하기 위해 범용 클래스를 만들어 사용한다. 이때, static 메서드로 구현되는 경우가 많다.
- 범용 클래스는 일반적으로 Common, Util이라는 이름이 붙는다.

**5.3.1. 너무 많은 로직이 한 클래스에 모이는 문제**

- 관련 없는 메서드들이 범용 클래스에 모이는 문제가 발생할 수 있다.
- ‘범용적으로 사용하고 싶은 로직은 Common 클래스에 모아 두면 되겠구나’하는 생각때문이다.

**5.3.2. 객체 지향 설계의 기본으로 돌아가기**

- 꼭 필요한 경우가 아니라면 범용 처리 클래스를 만들지 않는 것이 좋다.
- 다시 기본으로 돌아가 어떤 목적을 수행하는 클래스로 만들어주자.

**5.3.3. 횡단 관심사**

- 다양한 상황에서 넓게 활용되는 기능을 횡단 관심사(cross-cutting concern)라고 부른다.
  - 로그 출력
  - 오류 확인
  - 디버깅
  - 예외 처리
  - 캐시
  - 동기화
  - 분산 처리
- 횡단 관심사에 해당하는 기능이라면 범용 코드로 만들어도 괜찮다.

### 5.4 결과를 리턴하는 데 매개변수 사용하지 않기

```java
class ActorManager {
	// 게임 캐릭터 위치를 이동
	void shift(Location location, int shiftX, int shiftY) {
		location.x += shiftX;
		location.y += shiftY;
	}
}
```

- 이동 대상 인스턴스를 매개변수 location으로 전달받고, 이를 변경하고 있다.
- 이렇게 출력으로 사용되는 매개변수를 출력 매개변수라고 부른다.
- 데이터 조작 대상은 Location, 조작 로직은 ActorManager이다. 응집도가 낮은 구조인 것이다.

```java
class Location {
	final int x;
	final int y;
	
	Location(final int x, final int y) {
		this.x = x; this.y = y;
	}
	
	Location shift(final int shiftX, final int shiftY) {
		final int nextX = x + shiftX;
		final int nextY = y + shiftY;
		return new Location(nextX, nextY);
	} 
}
```

- 데이터와 데이터를 조작하는 논리를 같은 클래스에 배치하도록 수정했다.

### 5.5 매개변수가 너무 많은 경우

- 매개변수가 너무 많은 메서드는 응집도가 낮아지기 쉽다.
- 메서드에 매개변수를 전달하는 것은 해당 매개변수를 사용해서 어떤 기능을 수행하고 싶다는 의미이다.
- 즉, 매개변수가 많다는 것은 많은 기능을 처리하고 싶다는 의미가 된다.
- 처리할 게 많아지면 로직이 복잡해지거나, 중복 코드가 생길 가능성이 높아

**5.5.1. 기본 자료형에 대한 집착**

```java
class Common {
	// 할인된 가격 계산 메서드
	int discountedPrice(int regularPrice, float discountRate) {
		if(regularPrice < 0) {
			throw new IllegalArgumentException();
		}
	}
}
```

- ‘클래스를 많이 만드는 것이 오히려 이상해 보이는데?’ 라고 생각할 수 있다. 하지만 잘못된 생각이다.

```java
class Util {
	// 적절한 가격인지 확인하는 메서드
	boolean isFairPrice(int regularPrice) {
		if(regularPrice < 0) {
			throw new IllegalArgumentException();
		}
	}
}
```

- discountedPrice와 isFairPrice 모두 regularPrice가 유효한 값인지 검사하는 코드가 존재한다.
- 기본 자료형으로만 구현하면 이처럼 중복 코드가 많이 생긴다.
- 또, 계산 로직이 이곳저곳에 분산되기 쉽다.

```java
// 정가
class RegularPrice {
	final int amount;
	
	RegularPrice(final int amount) {
		if(amount < 0) {
			throw new IllegalArgumentException();
		}
		this.amount = amount;
	}
}
```

- ‘정가’라는 구체적인 자료형을 설계하기 위해 정가 클래스 RegularPrice를 만들고 그 내부에 유효성 검사를 캡슐화한다. (할인율 DiscountRate 클래스도 생성했다고 가정)

```java
class DiscountedPrice {
	final int amount;
	
	DiscountedPrice(final RegularPrice regularPice, final DiscountRate discountRate){
	 (...)
	}
}
```

- 이렇게 하면 관련 있는 로직을 각각의 클래스에 응집할 수 있다.

**5.5.2. 의미 있는 단위는 모두 클래스로 만들기**

- 매개변수가 너무 많아지는 문제를 피하려면, 개념적으로 의미 있는 클래스를 만들어야 한다.
- 매개변수가 많으면 데이터 하나하나를 매개변수로 다루지 말고, 그 데이터를 인스턴스 변수로 갖는 클래스를 만들고 활용하는 설계로 변경하라.

### 5.6 메서드 체인

- A.B().C().D()
- .(점)으로 여러 메서드를 연결해서 리턴 값의 요소에 차례차례 접근하는 방법을 메서드 체인 혹은 열차 사고(train wreck)라고 부른다.
- 메서드 체인은 버그가 발생하면 어디에서 발생한 것인지 모든 코드를 확인해야하고, 하나의 요소를 수정하면 모든 코드를 확인하고 수정해야하기 때문에 좋지 않은 코드이다.
- 데메테르 법칙: 사용하는 객체 내부를 알아서는 안된다는 법칙
- 메서드 체인으로 내부 구조를 돌아다닐 수 있는 설계는 데메테르 법칙을 위반한다고 할 수 있다.

**5.6.1. 묻지 말고 명령하기**

- 다른 객체의 내부 상태(변수)를 기반으로 판단하거나 제어하려고 하지 말고, 메서드로 명령해서 객체가 알아서 판단하고 제어하도록 설계하라는 의미이다.
- 인스턴스 변수를 private으로 변경해서 외부에서 접근할 수 없게 한다.
- 인스턴스 변수에 대한 제어는 외부에서 메서드로 명령하는 형태로 만든다.
- 상세한 판단과 제어는 명령을 받는 쪽에서 담당하게 한다.

```java
// 장비하고 있는 방어구 목록
class Equipments {
	private boolean canChange;
	private Equipment head;
	private Equipment armor;
	private Equipment arm;
	
	// 갑옷 장비하기
	void equipArmor(final Equipment newArmor) {
		if(canChange) {
			armor = newArmor;
		}
	}
	
	// 전체 장비 해제하기
	void deactivateAll() {
		head = Equipment.EMPTY;
		armor = Equipment.EMPTY;
		arm = Equipment.EMPTY;
	}
	
}
```

- 이렇게 하면 방어구의 탈착과 관련된 로직이 Equipments에 응집된다.
- 따라서 방어구와 관련된 요구 사항이 변경되었을 때, Equipments만 보면 된다.
