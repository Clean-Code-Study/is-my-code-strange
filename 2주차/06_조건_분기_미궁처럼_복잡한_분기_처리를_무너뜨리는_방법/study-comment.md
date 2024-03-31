# 6장 조건 분기: 미궁처럼 복잡한 분기 처리를 무너뜨리는 방법

- 조건분기를 사용하면 복잡한 판단을 빠르고 정확하게 할 수 있다.
- 그러나 조건이 복잡해지면 코드의 동작을 이해하기 어렵다.

## 6.1 조건 분기가 중첩되어 낮아지는 가독성

- 조건 분기 중첩 예시 : RPG 게임에서 마법 발동
- RPG의 전투 상황에서 플레이어가 각 멤버에게 마법을 발동하려면 여러가지 조건을 모두 통과해야한다.

```java
//살아 있는가
if (0 < member.hitPoint) {
	//움직일 수 있는가
	if (member.canAct()) {
		//매직 포인트가 남아 있는가
		if (magic.costMagicPoint <= member.magicPoint) {
			member.consumeMagicPoint(magic.constMagicPoint);
			member.chant(magic);
		}
	}
}
```

- 여러 개의 조건을 판정하기 위해 if 조건문을 중첩하였다.
- 중첩을 하게 되면 코드의 가성이 떨어진다.
- 어디부터 어디까지가 if 조건문의 처리블록으로 감싸진지 이해하기 어렵다.

```java
if(조건) {
	//
	// 수십 ~ 수백 줄의 코드
	//
	if (조건) {
	//
	//수십 ~ 수백 줄의 코드
	}
```

- 중첩 구조 사이사이에 코드가 섞여 있으면, if 조건문의 범위를 찾기가 힘들
- 가독성이 나쁜 코드는 팀 전체의 개발 생산성을 저하시킨다.
- 사양 변경은 더 어렵다. 충분히 이해하지 못한 상태로 로직을 변경하면 버그가 쉽게 숨어듭니다.

### 6.1.1 조기 리턴으로 중첩 제거하기

- 조기 리턴 : 조건을 만족하지 않는 경우 곧바로 리턴하는 방법
- 첫 번째 조건에서는 멤버가 살아있는지 확인하고 있다
- 이를 ‘살아 있지 않다면 곧바로 리턴한다’ 라는 형태로 변경한다.

```java
//살아 있지 않은 경우 리턴하므로 처리를 종료합니다.
// 조기 리턴으로 변경하기 위해 조건을 반전했습니다.

if (member.hitPoint <=0 ) return;

if (member.canAct()) {
	if (magic.costMagicPoint <= member.magicPoint) {
		member.consumeMagicPoint(magic.costMagicPoint);
		member.chant(magic);
	}
}
```

- 이렇게 중첩을 하나 제거했습니다.

```java
if (member.hitPoint <=0) return;
if (!member.canAct()) return;
if (member.magicPoint < magic.costMagicPoint) return;

member.consumeMagicPoint(magic.costMagicPoint);
member.chant(magic);
```

- 조건 로직과 실행 로직을 분리하여 볼 수 있다.
  - 마법을 쓸 수 없는 조건은 앞부분에 조기 리턴으로 모았고, 마법 발동 때 실행할 로직은 뒤로 모았다.
- 중첩이 제거되어 가독성이 좋아졌다.

요구사항 추가

- 멤버가 테크니컬포인트(TP)를 가짐
- 마법 발동에는 일정량의 테크니컬포인트가 필요함

- 발동 불능 조건을 확인하는 부분이 조기리턴으로 앞부분에 모여있다.
- 따라서 매우 간단하게 로직을 추가할 수 있습니다.

```java
if (member.hitPoint <=0) return;
if (!member.canAct()) return;
if (member.magicPoint < magic.costMagicPoint) return;
if (member.technicalPoint < magic.costTechnicalPoint) return; 
//새로 추가함
																															
member.consumeMagicPoint(magic.costMagicPoint);
member.chant(magic);
```

- 실행 로직과 관련된 요구사항 변경도 마찬가지
- ‘마법이 발동되면 일정량의 테크니컬 포인트를 얻는다’라는 조건을 추가
- 마법 발동 실행 로직이 뒷부분에 모여있으므로 간단하게 추가할 수 있다.

```java
if (member.hitPoint <= 0) return;
if (!member.canAct()) return;
if (member.magicPoint < magic.costMagicPoint) return;
if (member.technicalPoint < magic.costTechnicalPoint) return;

member.consumeMagicPoint(magic.costMagicPoint);
member.chant(magic);
member.gainTechnicalPoint(magic.incrementTechnicalPoint); //새로 추가함
```

- 조기 리턴은 가독성을 좋게 해서, 로직을 빠르게 이해할 수 있게 해준다.

### 6.1.2 가독성을 낮추는 else 구문도 조기 리턴으로 해결하기

- else 구문도 가독성을 나쁘게 만드는 원인 중 하나이다.
- 멤버의 히트 포인트가 너무 낮아졌을 때, 위험하다는 것을 표시하는 기능을 하기 위해 히트 포인트 비율에 따라서 건강 상태(HealthCondition)을 리턴

기본적인 요구사항 표

| 히트 포인트 비율 | 건강상태 |
| --- | --- |
| 0% | 사망 |
| 30% 미만 | 위험 |
| 50% 미만 | 주의 |
| 50% 이상 | 양호 |

- 설계를 생각하지 않고 단순하게 구현한다면 else 구문을 활용하기가 쉽다.

```java
float hitPointRate = member.hitPoint / member.maxHitPoint;

HealthCondition currentHealthCondition;
if (hitPointRate == 0) { 
	currentHealthCondition = HealthCondition.dead;
}
else if (hitPointRate < 0.3) {
	currentHealthCondition = HealthCondition.danger;
}
else if (hitPointRate < 0.5) {
	currentHealthCondition = HealthCondition.caution;
}
else {
	currentHealthCondition = HealthCondition.fine;
}

return currentHealthCondition;
```

- 중첩된 if 조건문 내부에 else 구문이 섞이면 가독성이 현저히 낮아져서, 코드를 이해하기 어렵다.
- 이러한 else 구문도 조기리턴을 사용해서 해결할 수 있다.

```java
float hitPointRate = member.hitPoint / member.maxHitPoint;

if (hitPointRate == 0) {
	return HealthCondition.dead;
}
else if (hitPointRate < 0.3) {
	return HealthCondition.danger;
}
else if (hitPointRate < 0.5) {
	return HealthCondition.caution;
}
else {
	return HealthCondition.fine;

```

- 이처럼 바로 리턴을 하면 사실 else 절 자체를 사용할 필요가 없다. 따라서 밑에 코드처럼 로직을 개선할 수 있다.

```java
float hitPointRate = member.hitPoint / member.maxHitPoint;

if (hitPointRate == 0) return HealthCondition.dead;
if (hitPointRate < 0.3) return HealthCondition.danger;
if (hitPointRate < 0.5) return HealthCondition.caution;

return HealthCondition.fine;
```

## 6.2 switch 조건문 중복

- 값의 종류에 따라 다르게 처리하고 싶을 때는 switch 조건문을 많이 사용합니다.
- 하지만 switch 조건문은 문제를 일으키기 쉽다.

- ‘게임 회사에서 새로운 RPG를 개발한다’라는 상황을 가정
- 1팀에서 공격 마법 구현

마법의 기본 요구사항

| 항목 | 설명 |
| --- | --- |
| 이름 | 마법의 이름. 어떤 마법인지 표시하는 데 사용합니다. |
| 매직포인트 소비량 | 마법 사용 시 소비되는 매직포인트입니다. |
| 공격력 | 마법의 공격력입니다. 마법별로 서로 다른 계산식을 사용합니다. |

- 개발 초기에는 표 6.3과 같은 마법들을 생각했습니다.

마법 목록

| 마법 | 설명 |
| --- | --- |
| 파이어 | 불 계열의 마법입니다. 사용자의 레벨이 높을수록 공격력이 올라갑니다. |
| 라이트닝 | 번개 계열의 마법입니다. 사용자의 민첩성이 높을수록 공격력이 올라갑니다. |

6.2.1 switch 조건문을 사용해서 코드 작성하기

- 종류에 따라 다른 로직을 구현해야 한다면,switch 조건문을 사용하는 경우가 많다.
- 마법의 종류에 따라 switch 조건문으로 로직을 분기했습니다.

- 일단 enum을 활용해 마법의 종류를 MagicType이라는 이름으로 정의했습니다.

```java
enum MagicType {
	fire, // 불 계열의 마법
	lighting //번개 계열의 마법
}
```

- 마법에는 각각 다음 요구 사항이 설정
1. 이름
2. 매직포인트 소비량
3. 공격력

- 이어서 마법의 이름을 알려주는 getName 메서드를 구현했다.
- switch 조건문을 사용해서 MagicType에 따라 표시 이름을 지정하게 했다.

```java
class MagicManager { 
	String getName(MagicType magicType) {
		String name = "";
		
		switch (magicType) {
			case fire;
				name = "파이어";
				break;
			case lightning;
			name = "라이트닝";
			break;
		}
		
	return name;
	}
}
```

### 6.2.2 같은 형태의 switch 조건문이 여러개 사용되기 시작

- 매직포인트 소비량과 공격력 모두 마법의 종류에 따라 다르기 때문에 switch 조건문으로 계산식을 구분하도록 구현했다.

- 매직포인트 소비량을 알려주는 costMagicPoint 메서드

```java
int costMagicPoint(MagicType magicType, Member member) {
	int magicPoint = 0;
	
	switch (magicType) {
		case fire:
			magicPoint = 2;
			break;
		case lightning:
			magicPoint = 5 + (int)(member.level * 0.2);
			break;
	}
		
		return magicPoint;
	}
```

- 공격력을 알려주는 attackPower 메서드

```java
int attackPower(MagicType magicType, Member member) {
	int attackPower = 0;
	
	switch (magicType) {
		case fire:
			attackPower = 20 + (int)(member.level * 0.5);
			break;
		case lightning:
			attackPower = 50 + (int)(member.agility * 1.5);
			break;
		}
		
		return attackPower;
	}
```

- 지금까지 코드를 검토해보면 MagicType에 따른 처리를 switch 조건문으로 구현한 코드가 3번이나 등장했다.
- switch 조건문을 여러번 사용하는 것은 매우 좋지 않다.

### 6.2.3 요구 사항 변경 시 수정 누락 (case 구문 추가 누락)

- 새로운 마법 ‘헬파이어’가 추가되었다.
- ‘헬파이어’라는 마법을 추가할수 있게 case구문을 추가했다

```java
String getName(MagicType magicType) {
	String name = "";
	
	switch (magicType) { 
		//생략
		case hellFire:
			name = "헬파이어";
			break;
		}
		
		return name;
	}

```

```java
int costMagicPoint(MagicType magicType, Member member) {
	int magicPoint = 0;
	
	switch (magicType) {
		//생략
		case hellFire:
			magicPoint = 16;
			break;
	}
	
	return magicPoint;
}
```

- 간단하게 동작을 확인하고 정해진 대로 동작하는 것 같아서 곧바로 출시
- 그런데 ‘헬파이어’ 대미지가 너무 약하다는 불만이 제기되었다.
- 확인해보니 공격력을 계산하는 attack Power 메서드에도 case 구문을 추가해야한다는 사실을 깜빡한 것이다.

```java
int attackPower(MagicType magicType, Member member) {
	int attackPower = 0;
	
	switch (magicType) { 
		// 생략
		// case hellFire: 추가를 깜빢함
	}
	
	return attackPower;
	
}
```

- 문제는 이것만이 아니다. 계속 새로운 요구 사항이 추가되었다.
- 새로운 요구사항
  - 마법 사용시 매직포인트를 소비
  - 동시에 특정행동을 할 때 테크니컬 포인트를 소비
- 이로인해 ‘마법도 테크니컬 포인트를 소비’ 하도록 변경
- 테크니컬 포인트 구현은 1팀이 아닌 다른팀에서 담당하게 되었다.

- 새담당자는 enum MagicType을 조건으로 사용해서, switch 조건문으로 마법을 종류별로 구분해 구현했음을 확인
- 이를 기반으로 테크니컬포인트를 리턴하는 메서드 costTechnicalPoint를 코드 6.17처럼 구현했습니다.

```java
int costTechnicalPoint(MagicType magicType, Member member) {
	int technicalPoint = 0;
	
	switch (magicType) {
		case fire:
			tetchnicalPoint = 0;
			break;
		case lightning:
			technicalPoint = 5;
			break;
	}
	
	return technicalPoint;
}
```

- 담당자는 딱히 문제가 없다고 판단하고 게임 출시
- 그런데 ‘일부 마법의 테크니컬 포인트 소비량이 표시되는 값과 다르다’ 라는 리뷰
- 확인해보니 마법 ’ 헬파이어’ 에 테크니컬 포인트 소비량이 구현되어있지 않았습니다.
- 담당자가 추가된 마법에 대해서 몰랐기 때문에 발생한 문제

### 6.2.4 폭발적으로 늘어나는 switch 조건문 중복

- 이런식으로 코드를 구현한다면 마법 종류만큼 case 구문을 사용해야한다.
- 또한 switch 조건문으로 처리할 대상은 실제로는 훨씬 많을 것이다.
- 그만큼 메서드가 만들어질 것이고 각각의 메서드에 switch 조건문이 작성된다.
- 지금 switch 조건문은 MagicType에 따라 분기되고 있다. 즉, switch 조건문이 중복 코드가 된것이다.
- switch 조건문의 중복이 많아지면 버그가 생기고 가독성이 낮아진다.

### 6.2.5 조건 분기 모으기

- switch 조건문 중복을 해소하려면 단일 책임 선택의 원칙을 생각해봐야한다.

- 단일 책임 선택의 원칙
  - 소프트웨어 시스템이 선택지를 제공해야한다면, 그 시스템 내부의 어떠 한 모듈만으로 모든 선택지를 파악할 수 있어야 한다.
- 즉 간단하게 말해, 조건식이 같은 조건 분기를 여러 번 작성하지 말고 한 번에 작성하자는 뜻

- 단일 책임 선택의 원칙에 따라서 MagicType의 switch 조건문을 하나로 묶어보자.

```java
class Magic {
	final String name;
	final int costMagicPoint;
	final int attackPower;
	final int costTechnicalPoint;
	
	Magic(final MagicType magicType, final Member member) {
		switch (magicType) {
			case fire:
				name = "파이어";
				costMagicPoint = 2;
				attackPower = 20 + (int)(member.level * 0.5);
				costTechnicalPoint = 0;
				break;
			case lightning:
				name = "라이트닝";
				costMagicPoint = 5 + (int)(member.level * 0.2);
				attackPower = 50 + (int)(member.agility * 1.5);
				costTechnicalPoint = 5;
				break;
			case hellFire:
				name = "헬파이어";
				costMagicPoint = 16;
				attackPower = 200 + (int)(member.magicAttack * 0.5 +
																	member.vitality * 2);
				costTechnicalPoint = 20 + (int)(member.level * 0.4);
				break;
			default;
				throw new IllegalArgumentException();
		}
	}
}
```

- switch 조건문 하나로 이름,매직포인트 소비량, 공격력, 테크니컬포인트 소비량을 전환
- switch 조건문이 한곳에 구현되어 있으므로 사양을 변경할 때도 누락 실수를 줄일 수 있다.

### 6.2.6 인터페이스로 switch 조건문 중복 해소하기

- 단일 책임 선택의 원칙으로 switch 조건문을 하나만 사용했다.
- 하지만 변경하고 싶은 부분이 많아지면 코드 6.18 로직이 점점 많아진다.
- 클래스가 거대해지면 데이터와 로직의 관계가 알기 힘들어진다.
- 따라서 클래스가 거대해지면 관심사에 따라 작은 클래스로 분할 하는 것이 중요하다.

- 이러한 문제를 해결 할 때 ‘인터페이스’를 사용한다.
- 인터페이스
  - 자바와 같은 객체 지향 프로그래밍 언어 특유의 문법으로, 기능 변경을 편리하게 할 수 있는 개념

- 사각형과 원을 구하는 Rectangle과 Circle클래스로 정의했다.
- 각 클래스에는 면적을 구하는 area 메서드가 있다.

```java
//사각형
class Rectangle {
	private final double width;
	private final double height;
	//생략
	double area(){
		return width * height;
	}
}

//원
class Circle {
	private final double radius;
	//생략
	double area() {
		return radius * radius * Math.PI;
	}
}
```

- 이코드에서 사각형 과  각각의 면적을 구하려면, area 메서드를 호출하면 된다

```java
rectangle.area();
circle.area();
```

```java
//다른 자료형의 인스턴스는 할당할 수 없습니다. 컴파일 오류가 발생합니다.
//또한 같은 이름의 메서드라도 사용할 수 없습니다.
Rectangle rectangle = new Circle(8);
rectangle.area();
```

- 하지만 면적을 구하는 메서드가 area로 이름이 같아도 Rectangle , Circle  각각 사용하는 클래스가 다르므로 코드 6.21위 처럼 Rectangle 자료형의 변수에 Circle 자료형의 인스턴스를 할당 할 수 없다.

- 이것을 해결해주는 것이 인터페이스다.
- 인터페이스는 서로 다른 자료형을 같은 자료형처럼 사용할 수 있게 해준다.

- 사각형과 원을 프로그램 상에서 같은 도형으로 다룰 수 있게 해보자.
- 도형을 나타내는 Shape이라는 이름의 인터페이스를 만든다.
- 그리고 공통 메서드도 정의한다. 도형의 면적을 나타내는 area 메서드를 정의했다.

```java
interface Shape {
	double area();
	}
```

- 그리고 도형을 다룰 Rectangle 과 Circle은 Shape 인터페이스를 구현하도록 변경한다.

```java
//사각형
class Rectangle **implements Shape**{
	private final double width;
	private final double height;
	//생략
	**public** double area(){
		return width * height;
	}
}

//원
class Circle **implements Shape** {
	private final double radius;
	//생략
	**public** double area() {
		return radius * radius * Math.PI;
	}
}
```

- 이렇게 하면 Rectangle 과 Circle 을 shape라는 자료형으로 다룰수 있다.
- Shape 자료형의 변수에 Rectangle 과 Circle 자료형의 변수를 할당 할 수 있는 것이다.
- 그리고 Shape 인터페이스에 공통 메서드로 정의해둔 area 메서드도 호출 할 수 있다.

```java
//Shape 인터페이스를 구현하는 Rectangle, Circle 모두를 할당할 수 있음.
Shape shape = new Circle(10);
System.out.println(shape.area()); // 원의 면적 출력
shape = new Rectangle(20, 25);
System.out.println(shape.area()); //사각형의 면적 출력
```

- 매개변수의 자료형을 Shape로 해두면, Shape 인터페이스를 구현하는 모든 클래스를 매개변수로 받을 수 있다.

```java
void showArea(**Shape** shape) {
	System.out.println(shape.area());
}

...
Rectangle rectangle = new Rectangle(8,12);
showArea(rectangle); //사각형의 면적을 출력
```

- 면적을 구하는 코드는 Retangle 과 Circle 클래스가 서로 다르다.
- 하지만 인터페이스를 사용하면, 조건 분기를 따로 작성하지 않고도 각각의 코드를 적절하게 실행할 수 있다.
- 이처럼 ‘각각의 코드를 간단하게 실행할수 있게 하는 것’ 이 인터페이스의 큰 장점 중 하나이다.

### 6.2.7 인터페이스를 switch 조건문 중복에 응용하기(전략 패턴)

---

**switch 조건문 중복 문제 해결에 인터페이스 응용하기**

- 종류별로 다르게 처리해야 하는 기능 인터페이스의 메소드로 정의
  - 인터페이스의 큰 장점 중 하나는 다른 로직을 같은 방식으로 처리할 수 있다는 것이다.
  - 마법 별로 다르게 처리해야 하는 기능

    ```java
    	String name(); //이름
    	int costMagicPoint(); //매직포인트 소비량
    	int attackPower(); //공격력
    	int costTechnicalPoint(); //테크니컬포인트 소비량
    ```

- 인터페이스의 이름을 결정하는 방법: ‘어떤 부류에 속하는가?’
  - 인터페이스를 구현하는 클래스들이 어떤 부류인가 생각해보기
  - 코드 6.23에서 사각형, 원형이 도형에 속하므로 Shape라는 이름을 붙인 것처럼 ‘파이어’, ‘라이트닝’, ‘헬파이어’를 마법으로 분류한다. 따라서 인터페이스 이름을 아래와 같이 Magic으로 짓는다.

      ```java
      // 마법 자료형
      interface Magic {
          String name();
          int costMagicPoint();
          int attackPower();
          int costTechnicalPoint();
      }
      ```

- 종류별로 클래스로 만들기
  - 코드 6.24에서 사각형, 원을 Rectangle, Circle 클래스로 만들고 영역 계산 메서드 area를 각각 구현 한 것과 같이 마법의 종류를 각각의 클래스로 만든다.

  | 마법 | 클래스 |
      | --- | --- |
  | 파이어 | Fire |
  | 라이트닝 | Lightning |
  | 헬파이어 | HellFire |

- 각각의 클래스에 인터페이스 구현하기
  - 각 마법 클래스에서 Magic 인터페이스 구현
  - Fire 클래스에서  마법 ‘파이어’의 이름, 매직포인트 소비량, 공격력, 테크니컬포인트 소비량을 확인할 수 있도록 아래와 같이 각각의 메서드를 구현한다.

    ```java
    // 마법 '파이어'
    class Fire implements Magic {
    	private final Member member;
    	
    	Fire(final Member member) {
    		this.member = member;
    	}
    	
    	public String name() {
    		return "파이어";
    	}
    	
    	public int costMagicPoint() {
    		return 2;
    	}
    	
    	public int attackPower() {
    		return 20 + (int)(member.level * 0.5);
    	}
    	
    	public int costTechnicalPoint() {
    		return 0;
    	}
    }
    ```

  - 다른 마법도 같은 방법으로 구현(생략)
  - 그랬을 때 Fire, Lightning, HellFire를 모두 Magic 자료형으로 활용할 수 있다

- switch 조건문이 아니라, Map으로 변경하기
  - 인터페이스를 사용한다고해서 switch 조건문에 의존하지 않고 다른 형태로 전환하기는 어렵다.
  - 따라서 키가 MagicType, 값이 Magic인 Map을 만든다.

    ```java
    final Map<MagicType, Magic> magics = new HashMap<>();
    //생략
    final Fire fire = new Fire(member);
    final Lightning lightning = new Lightning(member);
    final HellFire hellFire = new HellFire(member);
    
    magics.put(MagicType.fire, fire);
    magics.put(MagicType.lightning, lightning);
    magics.put(MagicType.hellFire, hellFire);
    ```

  - 대미지 계산을 위해 마법 공격력을 확인해야할 때 Map에서 MagicType에 대응하는 인스턴스를 추출한다. 이후 Magic 인터페이스의 attackPower를 호출한다.

    ```java
    void magicAttack(final MagicType magicType) {
     final Magic usingMagic = magic.get(magicType);
     usingMagic.attackPower();
    }
    ```

  - 위 코드의 magicAttack 메서드의 매개변수로 MagicType.hellFire를 전달하면, usingMagic.attackPower()로 HellFire.attackPower()가 호출된다.
  - Map이 switch 조건문 처럼 경우에 따라 구분을 하게된다.
  - 위 6.33 코드 처럼 이름, 마법 공격력, 공격력, 테크니컬포인트 소비량 처리 모두 Map을 사용해서 변경할 수 있다.

- 메서드를 구현하지 않으면 오류로 인식하게 만들기
  - 인터페이스를 활용하는 전략 패턴은 switch 조건문을 사용하지 않아도 된다 것 외에도 여러 장점이 있다.
  - 인터페이스를 활용하면 메서드 구현을 깜빡하면 컴파일조차 실패하기 때문에 실수 자체를 방지할 수 있다.
- 값 객체화하기
  - 현재 int 자료형을 리턴하는 메서드가 3개나 있는데 실수로 의미가 다른 값을 전달할 수도 있다. 따라서 값 객체를 만들어 두자.

    ```java
    interface Magic {
    	String name();
    	MagicPoint costMagicPoint();
    	AttackPower attackPower();
    	TechnicalPoint costTechnicalPoint();
    }
    ```

    ```java
    class Fire implements Magic {
    	private final Member member;
    
    	Fire(final Member member) {
    	this.member = member;
    	}
    	
    	public String name() {
    	return "파이어";
    	}
    	
    	public MagicPoint costMagicPoint() {
    		return new MagicPoint(2);
    	}
    	
    	public AttackPower attackPower() {
    		final int value = 20 + (int)(member.level * 0.5);
    		return new AttackPower(value);
    	}
    	
    	public TechnicalPoint costTechnicalPoint() {
    		return new TechnicalPoint(0);
    	}
    }
    ```


### 6.3 조건 분기 중복과 중첩

---

- 코드 6.41은 온라인 쇼핑몰에서 우수 고객인지 판정하는 로직이다.
- 고객의 구매 이력을 확인하고, 다음 조건을 모두 만족하는 경우, 골드 회원으로 판정한다.

```java
/**
* @return 골드 회원이라면 true
* @param history 구매 이력
*/
boolean isGoldCustomer(PurchaseHistory history) {
	if (1000000 <= history.totalAmount) {
		if (10 <= history.purchaseFrequencyPerMonth) {
			if (history.returnRate <= 0.001) {
				return true;
			}
		}
	}
	return false;
}
```

- 위 코드는 if 조건문이 중첩되어 있다.
- 이는 조기 리턴으로 제거할 수 있다.
- 아래는 실버 회원인지 판정하는 메서드이다.

```java
/**
* @return 실버 회원이라면 true
* @param history 구매 이력
*/
boolean isGoldCustomer(PurchaseHistory history) {
		if (10 <= history.purchaseFrequencyPerMonth) {
			if (history.returnRate <= 0.001) {
				return true;
			}
		}
	return false;
}
```

- 판정 조건이 골드 회원과 거의 같다.
- 이러한 같은 판정 로직을 재사용하기 위해서는 어떻게 해야할까?

### 6.3.1 정책 패턴으로 조건 집약하기

---

- 정책패턴
  - 조건을 부품처럼 만들고, 부품으로 만든 조건을 조합해서 사용하는 패턴
- 먼저 코드 6.43처럼 인터페이스를 만든다.

```java
interface ExcellentCustomerRule {
	/**
	* @param history 구매 이력
	* @return 조건을 만족하는 경우 true
	*/
	boolean ok(final PurchaseHistory history);
}
```

- 골드 회원이 되기 위해서는 3개의 조건을 만족해야한다.

```java
class GoldCustomerPurchaseAmountRule implements ExcellentCustomerRule {
 public boolean ok(final PurchaseHistory history) {
	 return 1000000 <= history.totalAmount;
 }
}
```

```java
class PurchaseFrequencyRule implements ExcellentCustomerRule {
	public boolean ok(final PurchaseHistory history) {
	return 10 <= history.purchaseFrequencyPerMonth;
	}
}
```

```java
class ReturnRateRule implements ExcellentCustomerRule {
	public boolean ok(final PurchaseHistory history) {
	return history.returnRate <= 0.001;
	}
```

- 이어서 정책 클래스를 만들고 add 메서드로 규칙을 집약한 뒤 complywithAll 메서드 내부에서 규칙을 모두 만족하는지 판정한다.

```java
class ExcellentCustomerPolicy {
	private final Set<ExcellentCustomerRule> rules;
	
	ExcellentCustomerPolicy() {
		rules = new HashSet();
	}
	
	/**
	* 규칙 추가
	*
	* @param rule 규칙
	*/
	void add(final ExcellentCustomerRule rule) {
		rules.add(rule);
}

/**
* @param history 구매 이력
* @return 규칙을 모두 만족하는 경우 true
*/
boolean complyWithAll(final PurchaseHistory history) {
	for (ExcellentCustomerRule each : rules) {
		if (!each.ok(history)) return false;
	}
	return true;
}
```

- if 조건문이 ExcellentCustomerPolicy.complyWithAll 메서드 내부에 하나만 있게 되어 로직이 단순해졌다.
- 아래 코드는 Rule과 Policy를 사용해서 골드 회원 판정 로직을 개선한 코드이다.

```java
ExcellentCustomerPolicy goldCustomerPolicy = new ExcellentCustomerPolicy();
goldCustomerPolicy.add(new GoldCustomerPurchaseAmountRule());
goldCustomerPolicy.add(new PurchaseFrequencyRule());
goldCustomerPolicy.add(new ReturnRateRule());

goldCustomerPolicy.complyWithAll(purchaseHistory);
```

- 위처럼 코드를 클래스에 그냥 작성하면, 골드 회원과 무관한 로직을 삽입할 가능성이 있다.
- 따라서 골드 회원 정책을 코드 6.49처럼 확실하게 클래스로 만든다.

```java
class GoldCustomerPolicy {
	private fianal ExcellentCustomerPolicy policy;
	
	GoldCustomerPolicy() {
		policy.add(new GoldCustomerPurchaseAmountRule());
		policy.add(new PurchaseFrequencyRule());
		policy.add(new REturnRateRule());
	}

/**
* @param history 구매 이력
* @return 규칙을 모두 만족하는 경우 true
*/
	boolean complyWithAll(final PurchaseHistory history) {
		return policy.complyWithAll(history);
	}
}
```

- 이후 골드 회원 조건이 달라질 경우, 이 GoldCustomerPolicy만 변경하면 된다.

### 6.4 자료형 확인에 조건 분기 사용하지 않기

---

- 인터페이스는 조건 분기를 제거할 때 활용할 수 있다.
- 하지만 인터페이스를 사용해도 조건 분기가 줄어들지 않는 경우가 있다.
- 전략 패턴으로 전환할 수 있게 인터페이스를 만든다.

```java
interface HotelRates {
	Money fee(); // 요금
	
	add(Money money);
}
```

- fee 메서드는 숙박 요금을 리턴한다.
- 일반 객실 요금과 프리미엄 객실 요금은 코드 6.52, 6.53처럼 HotelRates 인터페이스를 구현하는 형태로 만든다.

```java
class RegularRates implements HotelRates {
 public Money fee() {
	 return new Money(70000);
 }
}
```

```java
class PremiumRates implements HotelRates {
	public Money fee() {
		return new Money(120000);
}
```

- 위와 같이 인터페이스를 사용하면 숙박 요금을 상황에 따라 전환할 수 있다.
- 하지만 성수기 처럼 수요가 높은 기간에는 숙박 요금을 높게 설정하는 경우가 있기 때문에 성수기 때의 요금을 상향하는 로직을 다음과 같이 구현 했다.

```java
Money busySeasonFee;
if (hotelRates instanceof RegularRates) {
	busySeasonFee = hotelRates.fee().add(new Money(30000));
}
else if (hotelRates instanceof PremiumRates) {
	busySeasonFee = hotelRates.fee().add(new Money(50000));
}
```

- instanceof는 자료형 판정 연산자이다.
- 인터페이스 구현 클래스의 자료형을 확인해서 조건에 따라 나누고 있다.
- 인터페이스를 사용했는데도 조건분기가 있는 것이다.
- 이와 같은 로직은 리스코프 치환 원칙이라는 소프트웨어 원칙을 위반한다.
- 기반 자료형을 하위 자료형으로 변경해도 코드는 문제 없이 작동해야 한다는 것이다.

- 기반 자료형은 인터페이스를 의미하며 하위 자료형은 클래스를 의미한다.
- instanceof로 분기한 조건문 내부의 3만원과 5만원을 추가하는 로직에서 hotelRates는 다른 하위 자료형으로 변경하면 로직 자체가 깨진다.
- 따라서 유지보수가 어려운 코드가 되어버리는 것이다.

- 리스코프 치환 원칙은 부모 클래스를 자식 클래스로 대체해도 시스템이 정상 작동해야 한다는 원칙입니다. 예제에서 **`hotelRates`**의 타입 체크로 분기 처리한 로직은, 만약 **`hotelRates`**가 다른 하위 타입으로 변경되면 해당 로직이 깨질 수 있습니다. 즉, 인터페이스(기반 자료형)를 사용하더라도 구현 클래스(하위 자료형)에 따라 동작이 달라지므로, 원칙을 위반합니다. 이를 해결하기 위해선 다형성을 활용하여 각 클래스 내에 적절한 로직을 구현하는 것이 좋습니다.

---

- 성수기 요금도 인터페이스로 변경해보자
- HotelRates 인터페이스에 성수기 요금을 리턴하는 busySeasonFee 메서드를 추가한다.

```java
interface HotelRates {
	Money fee();
	Money busySeasonFee(); //성수기 요금
}
```

- 인터페이스 구현 클래스에서도 해당 메서드를 구현

```java
class RegularRates implements HotelRates {
	public Money fee() {
		return new Money(70000);
	}
	
	public Money busySeasonFee() {
		return fee().add(new Money(30000));
	}
}
```

```java
class PremiumRates implements HotelRates {
	public Money fee() {
		return new Money(70000);
	}
	
	public Money busySeasonFee() {
		return fee().add(new Money(30000));
	}
}
```

- 따라서 instanceof로 자료형을 판정하지 않아도 된다.

```java
Money busySeasonFee = hotelRates.busySeasonFee();
```

### 6.5 인터페이스 사용 능력이 중급으로 올라가는 첫걸음

---

- 인터페이스를 잘 사용하는 것은 곧 설계 능력의 전환점이다.
- 조건 분기를 써야 하는 상황에 일단 인터페이스 설계를 떠올려 본다.

### 6.6 플래그 매개변수

---

```java
damage(true, damageAmount);
```

```java
void damage(boolean damageFlag, int damageAmount) {
	if (damageFlag == true) {
	// 물리 대미지(히트포인트 기반 대미지)
		member.hitPoint -= damageAmount;
		if(0 < member.hitPoint) return;
		
		member.hitPoint = 0;
		member.addState(StateType.dead);
	}	
	else {
		// 마법 대미지(매직포인트 기반 대미지)
		member.magicPoint -= damageAmount;
		if (0 < member.magicPoint) return;
		
		membr.magicPoint = 0;
	}
}
```

- 첫번째 매개변수인 damageFlag로 물리 대미지인지 마법 대미지인지 구분한다.
- 이처럼 메서드의 기능을 전환하는 boolean 자료형의 매개 변수를 플래그 매개변수라고 한다.
- 플래그 매개변수의 단점
  - 어떤 일을 하는지 예측하기 힘들다.
  - 예측을 위해 메서드 내부 로직을 확인해야 하기때문에 가독성이 낮아지고 개발 생산성이 저하된다.
- boolean 뿐만아니라 int 자료형을 사용해도 같은 문제가 발생한다.

### 6.6.1 메서드 분리하기

---

- 메서드는 하나의 기능만 하도록 설계하는 것이 좋다.
- 따라서 플래그 매개변수를 받는 메서드는 기능별로 분리하는 것이 좋다.

```java
void hitPointDamage(final int damgeAmount) {
	member.hitPoint -= damageAmount;
	if (0 < member.hitPoint) return;
	
	member.hitPoint = 0;
	member.addState(StateType.dead);
}

void magicPointDamage(final int damageAmount) {
	member.magicPoint -= damageAmount;
	if (0 < member.magicPoint) return;
	
	member.magicPoint = 0;
}
```

- 위 코드 처럼 기능별로 나누고 메서드 이름을 잘 지어주면 가독성이 높아진다.

### 6.6.2 전환은 전략 패턴으로 구현하기

---

- 요구 사항이 달라져 히트포인트 대미지와 매직포인트 대미지를 전환 해야할 수도 있다.
- 이때 전환하기 위해 boolean 자료형을 사용하면 또 다시 플래그 매개변수로 돌아간다.
- 따라서 전환할 땐 인터페이스를 만들도록 한다.
