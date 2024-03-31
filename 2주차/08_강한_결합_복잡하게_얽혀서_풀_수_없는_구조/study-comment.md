# 8장 강한 결합: 복잡하게 얽혀서 풀 수 없는 구조

- 결합도란 ‘클래스 사이의 의존도를 나타내는 지표’. 어떤 클래스가 다른 클래스에 많이 의존하고 있는 구조를 강한 결합이라고 함. 강한 결합 코드는 이해하기도 힘들고, 변경하기도 굉장히 힘들다.
- 느슨한 결합 구조로 개선하면 코드 변경이 쉬워진다. 어떻게 느슨한 결합 구조를 만들 수 있는지 알아보자.

---

## 8.1 결합도와 책무

예를 들어 온라인 쇼핑몰에 할인 기능을 추가한다고 가정해보자.

- 일반 할인의 사양을 아래와 같이 구현
  - 상품 하나당 3,000원 할인
  - 최대 200,000원까지 상품 추가 가능

```java
class DiscountManager {
	List<Product> discountProducts;
	int totalPrice;
	
	boolean add(Product product, ProductDiscount productDiscount) {
		if (product.id < 0) {
			throw new IllegalArgumentException();
		}
		if (product.name.isEmpty()) {
			throw new IllegalArgumentException();
		}
		if (product.price < 0) {
			throw new IllegalArgumentException();
		}
		if (product.id != productDiscount.id) {
			throw new IllegalArgumentException();
		}
		
		int discountPrice = getDiscountPrice(product.price);
		
		int tmp;
		if (productDiscount.canDiscount) {
			tmp = totalPrice + discountPrice;
		} else {
			tmp = totalPrice + product.price;
		}
		if (tmp <= 200000) {
			totalPrice = tmp;
			discountProducts.add(product);
			return true;
		} else {
			return false;
		}
	}
	
	static int getDiscountPrice(int price) {
		int discountPrice = price - 3000;
		if (discountPrice < 0) {
			discountPrice = 0;
		}
		return discountPrice;
	}
}

class Product {
	int id;
	String name;
	int price;
}

class ProductDiscount {
	int id;
	boolean canDiscount;
}
```

- DiscountManager.add 메서드는 다음을 실행한다.
  - 올바른 상품인지 유효성 검사
  - 할인 가격 계산
  - productDiscount.canDiscount를 확인하여 할인 가능한 경우에는 할인 가격을 모두 더하고, 할인이 불가능한 경우에는 원래 상품 가격을 모두 더한다.
  - 가격 총합이 상한가인 200,000원 이내인 경우 상품 리스트에 추가한다.

- 그런데 이후에 일반 할인 이외에 여름 할인 사양이 추가되었다.
  - 상품 하나당 3,000원 할인
  - 최대 300,000원까지 상품 추가 가능
- 다른 담당자가 다음과 같은 SummerDiscountManager 클래스를 구현했다.

```java
class SummerDiscountManager {
	DiscountManager discountManager;
	
	boolean add(Product product) {
		if (product.id < 0) {
			throw new IllegalArgumentException();
		}
		if (product.name.isEmpty()) {
			throw new IllegalArgumentException();
		}
		
		int tmp;
		if (productDiscount.canDiscount) {
			tmp = discountManager.totalPrice + discountManager.getDiscountPrice(product.price);
		} else {
			tmp = discountManager.totalPrice + product.price;
		}
		if (tmp <= 300000) {
			discountManager.totalPrice = tmp;
			discountManager.discountProducts.add(product);
			return true;
		} else {
			return false;
		}
	}

class Product {
	int id;
	String name;
	int price;
	boolean canDiscount; // 여름 할인이 가능한 경우 true
}
```

- SummerDiscountManager.add 메서드는 다음과 같이 실행된다.
  - 올바른 상품인지 유효성 검사
  - 할인 가격 계산
  - productDiscount.canDiscount를 확인하여 할인 가능한 경우에는 할인 가격을 모두 더하고, 할인이 불가능한 경우에는 원래 상품 가격을 모두 더한다.
  - 가격 총합이 상한가인 300,000원 이내인 경우 상품 리스트에 추가한다.

### 8.1.1 다양한 버그

- 다음과 같은 사양 변경이 발생했다.
  - 일반 할인 가격을 3,000원에서 4,000원으로 변경
- DiscountManager 구현 담당자는 할인가를 계산하는 DiscountManager.getDiscountPrice를 다음과 같이 변경했다.

```java
static int getDiscountPrice(int price) {
		int discountPrice = price - 4000;
		if (discountPrice < 0) {
			discountPrice = 0;
		}
		return discountPrice;
	}
```

- 그런데 이렇게 하면 여름 할인 서비스에서도 할인 가격이 4,000원이 되어버린다. `버그 발생`

### 8.1.2 로직의 위치에 일관성이 없음

- 현재 할인 서비스 로직은 로직 위치 자체에도 문제가 있다.
  - DiscountManager, SummerDiscountManager가 상품 정보 확인, 할인 가격 계산, 할인 적용 여부 판단, 총액 상한 확인 등 너무 많은 일을 하고 있다.
  - Product에서 직접 해야 하는 유효성 검사 로직이 DiscountManager, SummerDiscountManager에 구현되어 있다.
  - ProductDiscount.canDiscount와 Product.canDiscount의 이름이 유사해서 일반 할인과 여름 할인구분하기가 힘들다.
  - 여름 할인 가격 계산을 위해 SummerDiscountManager가 DiscountManager의 일반 할인 로직을 활용하고 있다.
- 이처럼 로직의 위치에 일관성이 없다. 어떤 클래스는 처리해야 할 작업이 집중되어 있는 반면, 어떤 클래스는 특별히 하는 일이 없다.
- 이런 클래스 설계가 바로 책무를 고려하지 않은 설계라고 할 수 있다.

### 8.1.3 단일 책임 원칙

- 책임은 ‘누가 책임을 져야 하는가’라는 적용 범위와 밀접하다고 볼 수 있다.
- 소프트웨워의 책임이란 ‘자신의 관심사와 관련해서, 정상적으로 동작하도록 제어하는 것’ 이다.
- 단일 책임 원칙은 ‘클래스가 담당하는 책임은 하나로 제한해야 한다’는 설계 원칙이다.

### 8.1.4 단일 책임 원칙 위반으로 발생하는 악마

- DiscountManager.getDiscountPrice는 일반 할인 가격 계산을 책임지는 메서드이다. 여름 할인 가격을 책임지기 위해 만들어진 메서드가 아니므로 둘 다 책임지는 것은 단일 책임 원칙을 위반하는 것이다.
- 상품명과 가격이 타당한지 판단하는 책임은 이 데이터를 갖고 있는 Product 클래스가 가지고 있어야 한다. 이렇게 되면 결국 값 확인을 포함해 여러 코드가 중복될 것이다.

### 8.1.5 책임이 하나가 되게 클래스 설계하기

- 상품 가격을 나타내는 RegularPrice 클래스를 만들고, 잘못된 값이 들어오지 않게 유효성 검사 과정을 추가한다. 가격과 관련된 책임을 지는 클래스 구조이다. Money 클래스와 같은 값 객체이다.

```java
class RegularPrice {
	private static final int MIN_AMOUNT = 0;
	final int amount;
	
	RegularPrice(final int amount) {
		if (amount < MIN_AMOUNT) {
			throw new IllegalArgumentException("가격은 0 이상이어야 합니다.");
		}
		
		this.amount = amount;
	}
}
```

- 일반 할인 가격, 여름 할인 가격과 관련된 내용을 개별적으로 책임지는 값 객체를 만들어보자.

```java
class RegularDiscountedPrice {
	private static final int MIN_AMOUNT = 0;
	private static final int DISCOUNT_AMOUNT = 4000;
	final int amount;
	
	RegularDiscountedPrice(final RegularPrice price) {
		int discountedAmount = price.amount - DISCOUNT_AMOUNT;
		if (discountedAmount < MIN_AMOUNT) {
			discountedAmount = MIN_AMOUNT;
		}
		
		amount = discountedAmount;
	}
}
```

```java
class SummerDiscountedPrice {
	private static final int MIN_AMOUNT = 0;
	private static final int DISCOUNT_AMOUNT = 3000;
	final int amount;
	
	SummerDiscountedPrice(final RegularPrice price) {
		int discountedAmount = price.amount - DISCOUNT_AMOUNT;
		if (discountedAmount < MIN_AMOUNT) {
			discountedAmount = MIN_AMOUNT;
		}
		
		amount = discountedAmount;
	}
}
```

- 클래스가 분리되어 할인과 관련된 사양이 변경되어도 서로 영향을 주지 않는다. 이와 같이 관심사에 따라 분리해서 독립되어 있는 구조를 느슨한 결합이라고 부른다.

### 8.1.6 DRY 원칙의 잘못된 적용

- RegularDiscountedPrice와 SummerDiscountedPrice는 DISCOUNT_AMOUNT만 제외하면 로직이 대부분 같다.
- 이를 보고 ‘중복 코드가 작성된 것은 아닐까?’라고 생각할 수도 있다. 그런데 예를 들어 ‘여름 할인 가격은 정가에서 5% 할인한다’라는 사양으로 변경되면 어떨까? SummerDiscountedPrice의 로직이 RegularDiscountedPrice의 로직과 달라질 것이다.
- DRY 원칙은 반복을 피해라라는 의미인데, 코드 중복을 절대로 허용하지 말라는 뜻은 아니다. 비즈니스 개념 관점으로 생각하면 일반 할인과 여름 할인은 서로 다른 개념이다.
- 같은 로직, 비슷한 로직이라도 개념이 다르면 중복을 허용해야 한다. 개념적으로 다른 것까지도 무리하게 중복을 제거하려 하면 강한 결합 상태가 된다.

## 8.2 다양한 강한 결합 사례와 대처 방법

### 8.2.1 상속과 관련된 강한 결합

```java
class PhysicalAttack {
	int singleAttackDamage() {...}
	int doubleAttackDamage() {...}
}
```

```java
class FighterPhysicalAttack extends PhysicalAttack {
	@Override
	int singleAttackDamage() {return super.singleAttackDamage() + 20;}
	@Override
	int doubleAttackDamage() {return super.doubleAttackDamage() + 10;}
}
```

- doubleAttackDamage가 singleAttackDamage를 2회 실행하도록 바꾸면서 FighterPhysicalAttack에서 오버라이드한 singleAttackDamage가 2회 호출되어 격투가의 2회 공격 대미지 값이 50이 증가하는 버그가 발생했다.
- 이처럼 상속 관계에서 서브 클래스는 슈퍼 클래스에 굉장히 크게 의존한다. 슈퍼 클래스의 변화를 놓치는 순간, 버그가 만들어질 수 있다.
- 슈퍼 클래스 의존으로 인한 강한 결합을 피하려면, `상속보다 컴포지션`을 사용해라.
  - 컴포지션이란 사용하고 싶은 클래스를 private 인스턴스 변수로 갖고 사용하는 것이다.
  - 컴포지션 구조를 사용하면 PhysicalAttack의 로직을 변경해도 FighterPhysicalAttack이 영향을 적게 받는다.

    ```java
    class FighterPhysicalAttack {
    	private final PhysicalAttack physicalAttack;
    
    	int singleAttackDamage() {return physicalAttack.singleAttackDamage() + 20;}
    
    	int doubleAttackDamage() {return physicalAttack.doubleAttackDamage() + 10;}
    }
    ```

- 상속받는 쪽에서 차이가 있는 로직만 구현하는 템플릿 메서드 패턴도 있다. 즉, 잘 설계하면 문제없지만 예시 코드처럼 상속은 강한 결합과 로직 분산이 되기 때문에 신중하게 사용해야 한다.

### 8.2.2 인스턴스 변수별로 클래스 분할이 가능한 로직

```java
class Util {
	private int reservationId;
	private ViewSettings viewSettings;
	private MailMagazine mailMagazine;
	
	void cancelReservation() {...} // 예약 취소
	void darkMode() {...} // 다크 모드 표시 전환
	void beginSendMail() {...} // 메일 전송
}
```

- 클래스를 잘 분리하려면 각각의 인스턴스 변수와 메서드가 무엇과 관련 있는지 잘 파악해야 한다.
  - 의존 관계 그림을 그려주는 Jig 같은 도구를 사용해보자.

### 8.2.3 특별한 이유 없이 public 사용하지 않기

- 특별한 이유 없이 public을 붙이면 강한 결합 구조가 될 가능성이 있다.
- 적절한 접근제어자를 사용하자.
  - public : 모든 클래스에서 접근 가능
  - protected : 같은 클래스와 서브 클래스에서 접근 가능
  - private : 같은 클래스에서만 접근 가능
  - 없음(package private) : 같은 패키지에서만 접근 가능

### 8.2.4 private 메서드가 너무 많다는 것은 책임이 너무 많다는 것

```java
class OrderService {
	private int calcDiscountPrice(int price) {...} // 할인 가격 계산
	private List<Product> getProductBrowsingHistory(int userId) {...} // 최근 본 상품 리스트 조회
}
```

- 책임의 관점에서 생각해보면 가격 할인과 최근 본 상품 리스트 확인은 주문과 다른 책임이다.
- 책임이 다른 메서드는 다른 클래스로 분리하는 것이 좋다.

### 8.2.5 높은 응집도를 오해해서 생기는 강한 결합

- 관련 깊은 데이터와 논리를 한 곳에 모은 구조를 응집도가 높은 구조라고 하는데 이런 높은 응집도를 잘못 이해해서 강한 결합이 발생하는 경우가 있다.

```java
class SellingPrice {
	int calcSellingCommission() {...} // 판매 수수료 계산
	int calcDeliveryCharge() {...} // 배송비 계산
	int calcShoppoingPoint() {...} // 추가할 쇼핑 포인트 계산
}
```

- 이 메서드는 판매 가격을 사용해서 판매 수수료와 배송비를 계산한다. ‘판매 수수료와 배송비는 판매 가격과 관련이 깊을 것이다’ 라고 생각해서 SellingPrice 클래스에 계산 메서드를 추가할 수도 있다. 하지만 판매 가격에 판매 가격과 다른 개념이 섞여 있으므로 강한 결합에 해당한다.
- 응집도가 높다는 개념을 염두에 두고, 관련이 깊다고 생각되는 로직을 한 곳에 모으려고 했지만, 결과적으로 강한 결합 구조를 만드는 상황은 매우 자주 일어난다.

### 8.2.6 스마트 UI

- 화면 표시를 담당하는 클래스 중에서 화면 표시와 직접적인 관련이 없는 책무가 구현되어 있는 클래스
- 서로 다른 클래스로 분할하자.

### 8.2.7 거대 데이터 클래스

- 데이터 클래스가 커지면 거대 데이터 클래스가 된다. 수많은 인스턴스 변수를 갖는다.
- 온라인 쇼핑몰에는 발주, 예약, 배송 등 다양한 유스케이스가 있다. 각각의 유스케이스에서 필요한 데이터만 접근하고 사용할 수 있게 구성하는 것이 좋다.

### 8.2.8 트랜잭션 스크립트 패턴

- 메서드 내부의 일련의 처리가 하나하나 길게 작성되어 있는 구조
- 데이터 클래스와 데이터를 처리하는 클래스를 나누어 구현할 때 자주 발생한다.

### 8.2.9 갓 클래스

- 하나의 클래스 내부에 수천에서 수만 줄의 로직을 담고 있으며, 수많은 책임을 담당하는 로직이 난잡하게 섞여 있는 클래스이다.

### 8.2.10 강한 결합 클래스 대처 방법

- 객체 지향 설계와 단일 책임 원칙에 따라 제대로 설계하자.
- 거대한 강한 결합 클래스는 책임별로 클래스를 분할해야 한다. 단일 책임 원칙에 따라 설계된 클래스는 아무리 많아도 200줄 정도, 일반적으로는 100줄 정도이다.
