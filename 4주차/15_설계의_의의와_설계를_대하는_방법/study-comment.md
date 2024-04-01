# 14장 리팩터링: 기존의 코드를 성장시키는 기술

이미 구현되어 있는 코드의 구조가 좋지 않다면 리팩터링해야 한다.

---

## 14.1 리팩터링의 흐름

- 리팩터링은 실질적 동작은 유지하면서, 구조만 정리하는 작업이다.
- 실질적인 동작이 변하지 않았음을 확인할 수 있는 방법으로는 단위 테스트가 있다.
- 예시 - 웹툰 서비스 로직
    - 다음의 조건을 모두 만족해야 결제를 할 수 있다.
        - 구매자의 계정이 유효해야 한다.
        - 구매하려는 만화가 현재 구매 가능한 상태여야 한다.
        - 구매자가 갖고 있는 포인트가 만화 구매 포인트 이상이어야 한다.

```java
public class PurchasePointPayment {
    final CustomerId customerId; // 구매자의 ID
    final ComicId comicId; // 구매할 웹툰의 ID
    final PurchasePoint purchasePoint; // 구매에 필요한 포인트
    final LocalDateTime paymentDateTime; // 구매 일자
    
    PurchasePointPayment(final Customer customer, final Comic comic) {
        if (customer.isEnabled()) {
            customerId = customer.id;
            if(comic.isEnabled()) {
                comicId = comic.id;
                if(comic.currentPurchasePoint.amount <= customer.possessionPoint.amount) {
                    consumptionPoint = comic.currentPurchasePoint;
                    paymentDateTime = LocalDateTime.now();
                } else {
                    throw new RuntimeException("보유하고 있느 포인트가 부족합니다");
                }
            } else {
                throw new IllegalArgumentException("현재 구매할 수 없는 만화입니다.");
            }
        } else {
            throw new IllegalArgumentException("유효하지 않은 계정입니다.");
        }
    }
}
```

### 14.1.1 중첩을 제거하여 보기 좋게 만들기

- 조기 리턴을 활용하여 조건을 반전해서 if 조건문의 중첩을 제거하자.

### 14.1.2 의미 단위로 로직 정리하기

- 결제 조건을 확인하면서 customerId와 comicId에 값을 대입하고 있다. 서로 다른 일이 뒤섞여 있으므로, 로직이 정리되지 않는다.
- 조건 확인과 값 대입 로직을 각각 분리해서 정리하자. 조건 확인을 모두 완료한 후에 값을 대입하는 순서로 바꾸자.

### 14.1.3 조건을 읽기 쉽게 하기

- 유효하지 않은 구매자 계정을 판정 조건문의 논리 부정 연산자 `!` 때문에 가독성이 떨어진다.
- 따라서 Customer 클래스에 유효하지 않은 계정인지 여부를 리턴하는 isDisabled 메서드를 추가하자.

### 14.1.4 무턱대고 작성한 로직을 목적을 나타내는 메서드로 바꾸기

보유 포인트가 부족한지 리턴하는 isShortOfPoint 메서드를 Customer 클래스에 추가하자.

```java
class Customer {
	boolean isShortOfPoint(Comic comic) {
		return possessionPoint.amount < comic.currentPurchasePoint.amount;
	}
}

class PurchasePointPayment {
	PurchasePointPayment(final Customer customer, final Comic comic) {
    if (customer.isDisabled()) {
        throw new IllegalArgumentException("유효하지 않은 계정입니다.");
    }
    if (comic.isDisabled()) {
        throw new IllegalArgumentException("현재 구매할 수 없는 만화입니다.");
    }
    if (customer.isShortOfPoint(comic)) {
        throw new RuntimeException("보유하고 있느 포인트가 부족합니다");
    }
    
    customerId = customer.id;
    comicId = comic.id;
    consumptionPoint = comic.currentPurchasePoint;
    paymentDateTime = LocalDateTime.now();
	}
}
```

## 14.2 단위 테스트로 리팩터링 중 실수 방지하기

- 단위 테스트는 작은 기능 단위로 동작을 검증하는 테스트이다. 리팩터링할 때 단위 테스트는 필수다라는 말이 있다.
- 테스트가 없는 프로덕션 코드가 있다고 가정하고, 테스트 코드를 작성하고 리팩터링하는 과정을 알아보자
- 예시 - 온라인 쇼핑몰에서 배송비를 계산하고, 리턴하는 메서드

```java
public class DeliveryManager {
    public static int deliveryCharge(List<Product> products) {
        int charge = 0;
        int totalPrice = 0;
        for (Product each : products) {
            totalPrice += each.getPrice();
        }
        if (totalPrice < 20000) {
            charge = 5000;
        } else {
            charge = 0;
        }
        return charge;
    }
}
```

### 14.2.1 코드 과제 정리하기

- 구조 관점에서 몇 가지 과제가 있다. 일단 이 `과제`와 `이상적인 구조`에 대해 생각해 본 다음 리팩터링하자.
    - 우선 이 메서드는 static으로 정의되어 있다. static 메서드는 데이터와 데이터를 조작하는 로직을 분리해서 정의할 수 있는 구조이므로, 응집도가 낮아지기 쉽다.
    - 배송비는 금액을 나타내는 개념이므로, 값 객체로 만들면 좋을 것 같다.
    - 추가로 상품 합계 금액을 메서드 내부에서 계산하고 있는데, 합계 금액은 장바구니를 확인할 때, 실제 주문할 때 등 다양한 유스케이스에 사용된다. 따라서 각각의 메서드에서 따로 계산하면, 로직이 중복될 가능성이 높다.
    - 따라서 이를 별도의 클래스로 빼자. 합계 금액 계산은 List 자료형을 이용할 테니, 일급 컬렉션 패턴으로 설계하자.

### 14.2.2 테스트 코드를 사용한 리팩터링 흐름

1. 이상적인 구조의 클래스 기본 형태를 어느 정도 잡는다.
2. 이 기본 형태를 기반으로 테스트 코드를 작성한다.
3. 테스트를 실패시킨다.
4. 테스트를 성공시키기 위한 최소한의 코드를 작성한다.
5. 기본 형태의 클래스 내부에서 리팩터링 대상 코드를 호출한다.
6. 테스트가 성공할 수 있도록, 조금씩 로직을 이상적인 구조로 리팩터링 한다.

**이상적인 구조의 클래스 기본 형태를 어느 정도 잡기**

```java
class ShoppingCart {
    final List<Product> products;

    ShoppingCart() {
        this.products = new ArrayList<Product>();
    }

    private ShoppingCart(List<Product> products) {
        this.products = products;
    }

    ShoppingCart add(final Product product) {
        final List<Product> adding = new ArrayList<>(products);
        adding.add(product);
        return new ShoppingCart(adding);
    }
}

class Product {
    final int id;
    final String name;
    final int price;
    
    Product(int id, String name, int price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
}

class DeliveryCharge {
    final int amount;
    
    DeliveryCharge(final ShoppingCart shoppingCart) {
        this.amount = -1;
    }
}
```

**테스트 코드 작성하기**

- 배송비 책정 기준은 다음과 같다.
    - 상품 합계 금액이 20,000원 미만이면, 배송비는 5,000원이다.
    - 상품 합계 금액이 20,000원 이상이면, 배송비는 무료이다.

```java

class DeliveryChargeTest {
    //상품 합계 금액이 20,000원 미만이면, 배송비는 5,000원이다.
    @Test
    void payCharge() {
        ShoppingCart emptyCart = new ShoppingCart();
        ShoppingCart oneProductAdded = emptyCart.add(new Product(1, "상품A", 5000))
        ShoppingCart twoProductAdded = emptyCart.add(new Product(2, "상품B", 14990));
        final DeliveryCharge charge = new DeliveryCharge(twoProductAdded);
        assertEquals(5000, charge.amount);
    }

    //상품 합계 금액이 20,000원 이상이면, 배송비는 무료이다.
    @Test
    void chargeFree() {
        ShoppingCart emptyCart = new ShoppingCart();
        ShoppingCart oneProductAdded = emptyCart.add(new Product(1, "상품A", 5000))
        ShoppingCart twoProductAdded = emptyCart.add(new Product(2, "상품B", 15000));
        final DeliveryCharge charge = new DeliveryCharge(twoProductAdded);
        assertEquals(0, charge.amount);
    }
}
```

**테스트 실패시키기**

- 단위 테스트는 프로덕션 코드를 구현하기 전에, 실패와 성공을 확인해야 한다.
- 기대한 대로 실패 혹은 성공하지 않는다면 테스트 코드나 프로덕션 코드가 오류가 있다는 방증이기 때문
- 그래서 일단은 테스트를 실패시킨다. 현재 단계에서 테스트를 실행하면 둘 다 실패할 것이다.

**테스트 성공시키기**

- 이어서 테스트를 성공시킨다. 성공시킨다고 처음부터 본격적으로 구현하는 것은 아니다.
- 테스트를 성공시키기 위한 최소한의 코드만 구현한다. DeliveryCharge 클래스를 다음과 같이 변경한다.

```java
class DeliveryCharge {
    final int amount;

    DeliveryCharge(final ShoppingCart shoppingCart) {
        int totalPrice = shoppingCart.products.get(0).price + shoppingCart.products.get(1).price;
        if (totalPrice < 20000) {
            this.amount = 5000;
        } else {
            this.amount = 0;
        }
    }
}
```

**리팩터링하기**

- 테스트 코드가 의도대로 동작하는지 확인했다면, 이제 리팩터링한다. DeliveryCharge 생성자에서 리팩터링 대상인 DeliveryManager.deliveryCharge 메서드를 호출하도록 수정한다.

```java
class DeliveryCharge {
    final int amount;

    DeliveryCharge(final ShoppingCart shoppingCart) {
        amount = DeliveryManager.deliveryCharge(shoppingCart.products);
    }
}
```

- DeliveryManager.deliveryCharge 메서드에서 하던 합계 금액 계산 로직을 그대로 수행하는 totalPrice 메서드를 ShoppingCart 클래스에 추가한다.

```java
public class ShoppingCart {
    ...
    int totalPrice() {
        int amount = 0;
        for (Product each : products) {
            amount += each.price;
        }
        return amount;
    }
}
```

- DeliveryManager.deliveryCharge 메서드는 ShoppingCart 인스턴스를 전달받도록 수정한다. 그런 다음 계산을 직접 수행하지 말고 ShoppingCart.totalPrice 메서드를 호출하도록 변경한다.

```java
public class DeliveryManager {
    public static int deliveryCharge(ShoppingCart shoppingCart) {
        int charge = 0;
        if (shoppingCart.totalPrice() < 20000) {
            charge = 5000;
        } else {
            charge = 0;
        }
        return charge;
    }
}

class DeliveryCharge {
    final int amount;

    DeliveryCharge(final ShoppingCart shoppingCart) {
        amount = DeliveryManager.deliveryCharge(shoppingCart);
    }
}
```

- ShoppingCart 클래스의 인스턴스 변수 products 클래스 외부의 어디에서도 참조되지 않고 있고, 앞으로도 외부에서 마음대로 변경되면 위험할 수도 있다. 따라서 private 으로 변경한다.

```java
public class ShoppingCart {
  private final List<Product> products;
}
```

- DeliveryManager.deliveryCharge 메서드 로직을 DeliveryCharge 클래스의 생성자로 복사한다. 추가로 배송비 금액 할당 부분을 인스턴스 변수 amount로 변경한다.
- 이제 DeliveryManager.deliveryCharge 메서드는 필요없으므로 제거한다.
- DeliveryCharge는 좀더 수정해볼 수 있다. 20000, 5000, 0과 같은 매직 넘버는 상수로 만들어 활용하자
- 조건 분기를 삼항 연산자로 변경해보자
- 최종 프로덕션 코드

```java
/**
* 배송비
*/
class DeliveryCharge {
    private static final int CHARGE_FREE_THRESHOLD = 20000;
    private static final int PAY_CHARGE = 5000;
    private static final int CHARGE_FREE = 0;
    final int amount;

    DeliveryCharge(final ShoppingCart shoppingCart) {
        this.amount = (shoppingCart.totalPrice() < CHARGE_FREE_THRESHOLD) ? PAY_CHARGE : CHARGE_FREE;
    }
}

/**
* 장바구니
*/
class ShoppingCart {
    private final List<Product> products;

    ShoppingCart() {
        this.products = new ArrayList<Product>();
    }

    private ShoppingCart(List<Product> products) {
        this.products = products;
    }

    ShoppingCart add(final Product product) {
        final List<Product> adding = new ArrayList<>(products);
        adding.add(product);
        return new ShoppingCart(adding);
    }

    int totalPrice() {
        int amount = 0;
        for (Product each : products) {
            amount += each.price;
        }
        return amount;
    }
}
```

- 리팩터링 중간에 실수로 로직을 작성하면, 테스트가 실패하므로 곧바로 알아차릴 수 있어 안전하게 로직을 변경할 수 있다.

## 14.3 불확실한 사양을 이해하기 위한 분석 방법

- 단위 테스트를 사용한 리팩터링은 처음부터 사양을 알고 있다는 전제가 있기에 테스트를 작성할 수 있다
- 하지만 실제 개발을 하다보면 사양을 제대로 모르는 경우가 꽤 많다.
- 사양을 제대로 모른다면, 리팩터링을 위한 테스트 코드를 작성할 수 없다. 이럴 때는 어떻게 해야 할까?

### 14.3.1 사양 분석 방법 1: 문서화 테스트

```java
public class MoneyManager {
    public static int calc(int v, boolean flag) { ... }
}
```

- calc 메서드가 무슨 기능을 하는지 알 수 없는 상태에서는 테스트 코드를 작성할 수 없으므로, 안전하게 리팩터링하기 어렵다.
- 이때 활용할 수 있는 기법이 바로 **문서화 테스트**이다. 문서화 테스트는 메서드의 사양을 분석하는 방법이다.
- 일단 적당한 값을 입력해서 테스트를 작성한다.

```java
@Test
void characterizationTest() {
    int actual = MoneyManager.calc(1000, false);
    assertEquals(0, actual);
}
```

- 테스트는 실패하지만 아래의 결과를 얻을 수 있다.

```java
org.opentest4j.AssertionFailedError:
Expected :0
Actual   :1000
```

- 일단 매개변수 v의 값이 그대로 리턴된다는 사실을 알 수 있고, 입력에 대한 답을 하나 알 수 있게 되었다.
- 테스트가 성공할 수 있게 값을 변경해보자.

```java
@Test
void characterizationTest() {
    int actual = MoneyManager.calc(1000, false);
    assertEquals(1000, actual);
}
```

- 이와 같은 방법으로 calc 메서드가 어떤 값을 리턴하는지 알 수 있게, 테스트를 추가로 작성한다.
- 이 결과로 다음과 같은 사실을 유추할 수 있다.
    - 매개변수 flag가 false라면, 매개변수 v를 그대로 리
    - 매개변수 flag가 true라면, 매개변수 v를 활용해서 어떤 계산을 수행한 뒤 리턴
- 이처럼 문서화 테스트는 분석하고 싶은 메서드의 테스트를 작성해서 해당 메서드가 어떤 동작을 하는지 확인하는 방법이다.

### 14.3.2 사양 분석 방법 2: 스크래치 리팩터링

- 앞에서 배송비 코드를 리팩터링하는 예는 이상적인 구조가 머리속에 먼저 떠오를 수 있는 코드였다.
- 하지만 실제 프로덕션 코드는 굉장히 복잡하고 기괴해서, 이상적인 구조를 유추하기 어려운 경우도 많다.
- 그리고 이러한 코드는 일반적으로 사양 자체가 불분명한 경우가 많다.
- 이러한 상황에 유횽하게 쓸 수 있는 분석 방법이 바로 스크래치 리팩터링이다. 이는 정식 리팩터링이 아니라, 로직의 의미와 구조를 분석하기 위해 시험 삼아 리팩터링하는 것이다.
- 일단 대상 코드를 리포지터리에서 체크아웃하고 테스트 코드를 작성하지 않고 코드를 리팩터링한다. 코드가 정리되어 가독성이 좋아지면 다음과 같은 장점이 생긴다.
    - 코드의 가독성이 좋아져 로직의 사양을 이해할 수 있게 된다.
    - 이상적인 구조가 보인다. 쓸데없는 코드가 보인다.
    - 테스트 코드를 어떻게 작성해야 할지 보인다.
- 스크래치 리팩터링으로 분석한 결과를 기반으로, 이상적인 구조를 떠올릴 수 있다. 스크래치 리팩터링은 어디까지나 분석용이므로, 역할을 마쳤다면 그냥 파기해야 한다.

## 14.4 IDE의 리팩터링 기능

### 14.4.1 리네임

- 한 번에 클래스, 메서드, 변수의 이름을 전부 변경하는 리팩터링 기능

### 14.4.2 메서드 추출

- 로직 일부를 메서드로 추출해 주는 기능

## 14.5 리팩터링 시 주의 사항

### 14.5.1 기능 추가와 리팩터링 동시에 하지 않기

### 14.5.2 작은 단계로 실시하기

- 커밋은 어떻게 리팩터링했는지 차이를 알 수 있는 단위로 한다.

### 14.5.3 불필요한 사양은 제거 고려하기
