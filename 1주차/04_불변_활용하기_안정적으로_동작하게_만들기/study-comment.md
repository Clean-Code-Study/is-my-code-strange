# 4장 불변 활용하기: 안정적으로 동작하게 만들기

불변 활용하기: 안정적으로 동작하게 만들기

- **가변**: 상태를 변경할 수 있는 것 (변수의 값을 변경하는 등)
- **불변**: 상태를 변경할 수 없는 것

⇒ 가변과 불변을 적절하게 설계하지 못하면 동작을 예측하기 어렵다.

가능한 한 상태가 변경되지 않도록 설계해야 한다. 이때 불변의 개념이 사용된다.

요즘 프로그래밍 트렌드는 불변!!!!

## 4.1 재할당

: 변수의 값을 다시 할당하는 것 (=파괴적 할당)

- 변수의 의미를 추측하기 어렵고 언제 어떻게 변경됐는지 추적하기가 힘들다.

```java
int damage(){
//멤버의 힘과 무기 성능을 **기본 공격력**으로 활용합니다.
int **tmp** = member.power() + member.weaponAttack();
//멤버의 **속도로 공격력을 보정**합니다.
**tmp** = (int)(**tmp** * (1f + member.speed() / 100f));
//공격력에서 적의 방어력을 뺀 값을 **대미지**로 사용합니다.
**tmp** = **tmp** - (int)(enemy.defence/2);
//대미지가 음수가 되지 않도록 조정합니다.
**tmp** = Math.max(0,tmp);

return **tmp**;

}
```

→ 기본 공격력, 보정값, 대미지 등이 계속 **재할당** 되면서 값의 의미가 바뀐다

- 중간에 의미가 바뀌면 헷갈려 버그를 만들어 낼 가능성이 높아진다. 따라서 새로운 변수를 만들어 사용해야한다.

### 4.1.1 불변 변수로 만들어서 재할당 막기

- 재할당을 기계적으로 막을 수 있는 방법은 변수에 **`final`** 수식자를 붙이는 것이다.
- final을 붙인 변수는 변경할 수가 없다.

```java
void doSomething() {
**final** int value = 100;
value = 200; //컴파일 오류
}
```

→ **`final`** 키워드는 변수에 할당된 값을 변경할 수 없음을 나타내며, 한 번 초기화되면 다시 할당할 수 없다.

```java
int damage(){
//멤버의 힘과 무기 성능을 기본 공격력으로 활용합니다.
 **final int basicAttackPower** = member.power() + member.weaponAttack();
//멤버의 속도로 공격력을 보정합니다.
**final int** **finalAttackPower** = (int)(**basicAttackPower** * (1f + member.speed() / 100f));
//공격력에서 적의 방어력을 뺀 값을 대미지로 사용합니다.
**final int** **reduction** = (int)(enemy.defence / 2);
//대미지가 음수가 되지 않도록 조정합니다.
**final int damage** = Math.max(0, **finalAttackPower** - **reduction**);

return **damage**;

}
```

### 4.1.2 매개변수도 불변으로 만들기

- 매개변수도 마찬가지로 변경하면 값의 의미가 바뀔 수 있고 코드를 읽기 헷갈려 버그의 원인이 되다.

```java
void addPrice(int **productPrice**) {
	**productPrice** = totalPrice + productPrice;
	if (MAX_TOTAL_PRICE < **productPrice**) {
		throw new IllegalArgumentException("구매 상한 금액을 넘었습니다.");
}
```

→ 재할당을 막으려면 마찬가지로 final 을 붙이면 된다.

```java
void addPrice(**final** int **productPrice**) {
	**final** int **increasedTotalPrice** = totalPrice + productPrice;
	if (MAX_TOTAL_PRICE < **increasedTotalPrice**) {
		throw new IllegalArgumentException("구매 상한 금액을 넘었습니다.");
}

//throw new IllegalArgumentException  자바에서 예외를 발생시키는 구문
//productPrice 매개변수 increasedTotalPrice 지역 변수 모두 final 불변으로 만듬
```

## 4.2 가변으로 인해 발생하는 의도하지 않은 영향

- 인스턴스가 가변일 경우 다른 부분에 의도하지 않은 영향을 주기 쉽다.
- 코드를 변경했을 때 생각하지도 못했던 위치에서 상태가 변화하여 예측하지 못한 동작을 하는 경우가 있다.

### 4.2.1 사례 1: 가변 인스턴스 재사용하기

- 무기의 공격력을 나타내는 AttackPower 클래스를 구현했다. 공격력 값을 저장하는 인스턴스 변수 value에 final 수식자가 따로 붙어있지 않으므로 가변이다.

```java
class AttackPower {
	static final int MIN = 0;
	int **value**; // final을 붙이지 않았으므로 가변(공격력 값을 저장하는 인스턴스변수)

	AttackPower(int value) {
		if (value < MIN) {
			throw new IllegalArgumentException();
		}

		this.value = value;
	}
}
```

- 무기를 나타내는 Weapon 클래스는 AttackPower를 인스턴스 변수로 갖는 구조이다.

```java
class Weapon {
	final AttackPower attackPower;

	Weapon(AttackPower attackPower) {
		this.attackPower = attackPower;
	}
}
```

- 처음 코드 짤 때는 무기의 공격력이 고정적이었다. 그래서 공격력이 같으면 AttackPower 인스턴스를 재사용하였다.

```java
AttackPower **attackPower** = new AttackPower(20);
Weapon weaponA = new Weapon(**attackPower**);
Weapon weaponB = new Weapon(**attackPower**);
```

- 이후 ‘무기 각각의 공격력을 강화 할 수 있도록 조건 변경하자’의견이 나와 수정
- 어떤 무기의 공격력을 강화하려면 다른 무기의 공격력도 강화되는 버그가 발생
- 4.9코드에서 weaponA을 변경했더니 weaponB 공격력도 함께 바뀐다. AttackPower 인스턴스를 재사용했기 때문이다

```java
AttackPower **attackPower** = new AttackPower(20); //공격력을 나타내는 AttackPower 객체를 생성하고, 초기 공격력을 20으로 설정
Weapon weaponA = new Weapon(**attackPower**); //attackPower 객체를 사용하여 첫 번째 무기인 weaponA를 생성합니다.
Weapon weaponB = new Weapon(**attackPower**);

weaponA.attackPower.value = 25;

System.out.println("Weapon A attack power : " +
									weaponA.attackPower.value);

System.out.println("Weapon B attack power : " +
									weaponB.attackPower.value);
```

```java
Weapon A attack power : 25
Weapon B attack power : 25
```

→ 이처럼 가변 인스턴스는 재사용시 한쪽의 변경이 다른 한쪽에 영향을 준다.

이런 상황을 막기 위해서는 AttackPower 인스턴스를 개별적으로 생성하고, 재사용하지 않는 로직으로 변경해야한다.

```java
AttackPower **attackPowerA** = new AttackPower(20);
AttackPower **attackPowerB** = new AttackPower(20);

Weapon weaponA = new Weapon(**attackPowerA**);
Weapon weaponB = new Weapon(**attackPowerB**);

weaponA.attackPower.value += 5;

System.out.println("Weapon A attack power : " +
										weaponA.attackPower.value);
System.out.println("Weapon B attack power : " +
										weaponB.attackPower.value
```

이렇게 하면 한쪽 공격력을 변경해도 공격력은 그대로다

```java
Weapon A attack power : 25
Weapon B attack power : 20
```

### 4.2.2 사례2: 함수로 가변 인스턴스 조작하기

예상하지 못한 동작은 함수 때문에도 발생한다.

AttackPower 클래스에 공격력을 변화시키는 reinfore메서드와 disable 메서드를 추가했다.

```java
class AttackPower {
	static final int MIN = 0;
	int value;  //가변(공격력 값을 저장하는 인스턴스변수)
	AttackPower(int value){
		if (value < MIN) {
			throw new IllegalArgumentException();
		}

		this.value = value;
	}

/**
	공격력 강화하기
	@param increment 공격력 증가량
	*/
	void reinforce(int inrement) {
		value += increament;
	}

	/**무력화 하기 */
	void disable(){
		value = MIN;
	}
}
```

전투 중에 공격력을 강화하는 상황에서 AttackPower.reinforce 메서드를 호출하는 형태라고 생각하면

```java
AttackPower attackPower = new AttackPower(20);
//생략
attackPower.reinforce(15);
System.out.println("attack power : " + attackPower.value);
```

일단 처음에는 정상적으로 동작했습니다.

```java
attack power :35
```

그런데 어느 날 갑자기 제대로 동작하지 않게 되었습니다. 공격력이 0이 되는 일이 종종 발생하게 된 것입니다.

원인을 조사한 결과 AttackPower 인스턴스가 다른 작업에서 사용되었음을 확인.

코드 4.17과 같은 코드를 다른 곳에서 실행하면서 공격력을 0으로 만드는 AttackPower.disable 메서드를 호출한 것입니다.

```java
//다른 스레드의 처리
attackPower.disable();
```

AttackPower의 disable 메서드와 reinforce 메서드는 구조적인 문제를 갖고있습니다.

그것이 바로 부수 효과입니다.

### 4.2.3 부수 효과의 단점

주요 작용: 함수(메서드)가 매개 변수를 전달받고 값을 리턴하는 것

부수 효과:  주요 작용 이외의 상태 변경을 일으키는 것

상태 변경이란? 함수 밖에 있는 상태를 변경하는 것을 의미한다.

- 인스턴스 변수 변경
- 전역 변수 변경(9.5절 참고)
- 매개변수 변경
- 파일 읽고 쓰기 같은 I/O 조작

→부수효과가 있는 함수는 영향 범위를 예측하기 힘들기 때문에 범위를 한정시키는 것이 좋다.

### 4.2.4 함수의 영향 범위 한정하기

- 데이터(상태)는 매개변수로 받습니다.
- 상태를 변경하지 않습니다.
- 값은 함수의 리턴 값으로 돌려줍니다.
- 인스턴스 변수
    - 메서드에서 인스턴스 변수를 사용하는 것도 좋지  않다는 말일까? 라고 생각하는 분도 있을 것입니다. 하지만 이는 괜찮습니다. 인스턴스 변수는 불변으로 만들어 영향이 전달되지 않게 할 수 있다.

### 4.2.5 불변으로 만들어서 예기치 못한 동작 막기

부수 효과의 여지 자체를 없앨 수 있기 인스턴스 변수 value에 final 수식자를 붙여 불변으로 만들기

reinforce 메서드와 disable 메서드처럼 attackPower인스턴스를 새로 생성하고 리턴하는 구조로 변경

```java
class AttackPower {
	static final int MIN = 0;
	**final** int value;  //final로 불변으로 만들었습니다.

	AttackPower(**final** int value){ //final
		if (value < MIN) {
			throw new IllegalArgumentException();
		}

		this.value = value;
	}

/**
	공격력 강화하기
	@param increment 공격력 증가량
	@return 증가된 공격력 
	*/
		void reinforce(**final** **AttackPower** inrement) {//인스턴스 새로생성 매개변수 불변
		**return** **new AttackPower**(this.value + increment.value); //리턴값
	}//메서드가 호출될 때 새로운 AttackPower 인스턴스를 생성하여 값을 갱신하고 반환

	/**무력화 하기 
	 @return 무력화한 공격력
	*/
	**AttackPower** disable(){
		**return** **new AttackPower**(MIN);
	}//AttackPower 인스턴스를 새로 생성하여 반환
}
```

```java
**final** AttackPower attackPower = new AttackPower(20);
//생략
**final** **AttackPower reinforced** = 
		attackPower.reinforce(**new AttackPower(**15));
System.out.println("attack power : " + **reinforced**.value);
```

```java
//다른 스레드에서 처리
**final** **AttackPower** disabled = attackPower.disable();
```

## 4.3 불변과 가변은 어떻게 다루어야할까?

### 4.3.1 기본적으로 불변으로

변수가 불변일 경우 장점

- 변수의 의미가 변하지 않아 혼란을 줄여준다.
- 동작이 안정적이게 되므로 결과를 예측하기 쉽다.
- 코드의 영향 범위가 한정적이므로, 유지 보수가 편리해진다.

→ 최근 등장하는 프로그래밍 언어는 불변이 디폴트가 되도록 만들어진다.

### 4.3.2 가변으로 설계해야하는 경우

<성능이 중요한 경우>

1. 대량의 데이터를 빠르게 처리해야하는 경우
2. 이미지를 처리하는 경우
3. 리소스에 제약이 큰 임베디드 소프트웨어를 다루는 경우
4. 스코프가 국소적(일부분)인 경우(변수나 함수의 스코프가 특정한 범위에 국한 반복문의 변수 i같은 경)
- 스코프가 국소적인 경우

  , 반복문 내에서 사용되는 변수가 그 반복문 블록 내에서만 유효하고, 반복문 외부에서는 접근할 수 없는 경우가 있습니다. 이런 경우에 해당 변수는 "국소적인 스코프(local scope)"를 가지고 있다고 말할 수 있습니다.


→ 크기가 큰 인스턴스를 새로 생성하면서 시간이 오래걸려 성능에 문제가 생길경우

### 4.3.3 상태를 변경하는 메서드 설계하기

인스턴스 변수를 가변으로 만들었다면 메서드를 만들 때는 올바른 상태로만 변경하도록 주의해서 설계해야한다.

<조건>

- 히트 포인트는 0 이상
- 히트 포인트가 0이 되면, 사망 상태로 변경

```java
class HitPoint {
	int amount;
}

class Member {
	final Hitpoint hitPoint;
	final States states;
	//생략
	
	/***
	대미지 받기
	@parm damageAmount 대미지 크기
	*/
	void damage(int damageAmount){
		hitPoint.amount -= damageAmount;
	}
}
```

→Member.damage로직은 HitPoint.amount가 음수가 될 수 있다. 또한 0이 되어도 사망상태로 변하지않는다.

상태를 변화시키는 메서드를 ‘뮤테이터(mutater)’라고 한다. 조건에 맞는 상태로 변경하는 뮤테이터로 바꿔보자

```java
class HitPoint {
	**private static final int MIN = 0; //체력 최소 값 설정**
	int amount; //현재 체력값을 저장하는 변수 

HitPoint(final int amount) {
	if (amount < MIN) {
		throw new IllegalArgumentException(); 
	}
	
	this.amount = amount; //주어진값 할당
	}
	
 /**
	*대미지 받기
	*@parm damageAmount 대미지 크기
	*/
	void damage(int damageAmount){
		**final int nextAmount** = amount - damageAmount;
		**amount = Math.max(MIN, nextAmount);**
	}

/** @return 히트포인트가 0이라면 true */
**boolean isZero(){
	return amount == MIN;
	}**
}

class Member {  //게임 캐릭터를 나타내는클래
	final HitPoint hitPoint;  .//객체 참조 
	final States states;
	//생략
	
	/**
	 * 대미지 받는 처리
	 * param damageAmount 대미지 크기
	 */
	 //damage 메서드는 캐릭터가 대미지를 받을 때 호출되는 메서드입니다. 
	 //주어진 대미지만큼 hitPoint 객체의 체력을 감소시키고, 
	 //만약 체력이 0이라면 states 객체에 'dead' 상태를 추가합니다.
	**void damage(final int damageAmount){
		hitPoint.damage(damageAmount);
		if (hitPoint.isZero()) {
			state.add(StateType.dead);**
		**}
	}**
} 
```

### 4.3.4 코드 외부와 데이터 교환은 국소화하기

불변을 활용해서 신중하게 설계하더라도, 코드 외부와의 데이터 교환은 주의해야한다.

파일이나 데이터베이스는 코드 외부에 있는 상태이다. 예를 들면 파일의 내용은 다른 시스템에 의해서 덮어쓰일 수 있다.  코드 내부에서는 이러한 외부 동작을 제어할 수 없다.

특별한 이유없이 외부 상태에 의존하는 코드를 작성하면 동작 예측이 힘들어지므로 문제가 발생할 가능성이 높아진다.

국소화 하는 방법

- 리포지터리 패턴
    - 데이터 베이스의 영속화(데이터베이스에 데이터를 저장하는것) 를 캡슐화하는 디자인 패턴
    - 특정 클래스 내부에 데이터베이스 관련 로직을 격리하므로, 애플리케이션 로직이 데이터베이스 관련 로직과 섞이지 않습니다. 리포지터리 패턴은 집합체(정합성 유지가 필요한 여러 클래스의 집합체)라는 단위로 데이터를 읽고 쓰게 설계하는 것이 일반적이다.

데이터 출처(로컬 DB인지 API응답인지 등)와 관계 없이 동일 인터페이스로 데이터에 접속할 수 있도록 만드는 것을 Repository 패턴이라고 합니다.

