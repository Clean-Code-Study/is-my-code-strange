
# 2장 설계 첫걸음

### 2.1 의도를 분명히 전달할 수 있는 이름 설계하기

---

```java
int d = 0;
d = p1 + p2;
d = d - ((d1 + d2) / 2);
if (d < 0) {
	d = 0;
}
```

- 무언가를 계산하고 있지만, 무엇을 하고 있는지 전혀 이해할 수 없다.
- 이 코드는 게임에서 데미지를 계산하는 로직이다.

| 변수 | 의미 |
| --- | --- |
| d | 데미지 크기 |
| p1 | 플레이어의 기본 공격력 |
| p2 | 플레이어 무기의 공격력 |
| d1 | 적 자체의 방어력 |
| d2 | 적 방어구의 방어력 |
- 이름을 짧게 줄이면 입력해야하는 글자 수가 줄어 조금이라도 빠르게 구현할 수 있을 수 있다.
- 하지만 다른 사람이 읽거나 시간이 지난 후 다시 볼 때는 이해하기 매우 어렵다.
- 따라서 입력할 때 아낀 시간보다 몇 배 이상의 시간이 필요할 수가 있다.
- 그렇기 때문에 의도를 알 수 있는 이름을 사용해야한다.

```java
int damageAmount = 0;
damageAmount = playerArmPower + playerWeaponPower; // 1
damageAmount = damageAmount - ((enemyBodyDefence + enemyArmorDefence) / 2); // 2

if (damageAmount < 0) {
 damageAmount = 0;
}
```

- 자주 바뀔 가능성이 있는 코드를 구현할 때는 변수 이름을 알기 쉽게 붙이는 것이 훌륭한 기본 설계가 될 수 있다.

### 2.2 목적별로 변수를 따로 만들어 사용하기

---

- 데미지 크기를 나타내는 damageAmount에 값이 여러번 할당되고 있다.
- 복잡한 계산을 할 때 이처럼 계산의 중간 결과를 동일한 변수에 계속해서 대입하는 코드가 많이 사용된다.
- 이러한 재할당은 변수의 용도가 바뀌는 문제를 일으키기 쉽다. 따라서 코드를 읽는 사람을 혼란스럽게 만들고 버그를 만들어 낼 가능성이 있다.
- 코드 2.2의 1에서 damageAmount에 할당하는 값은 데미지의 크기가 아닌 플레이어 공격력의 총향이다.
- 1 에는 플레이어의 공격력 총합, 2에는 적 방어력의 총합을 포함해 데미지 양을 계산한다.
- 이처럼 개념이 다른 값들은  totalPlayerAttackPower와 totalEnemyDefence라는 변수를 만들어 할당한다.

```java
int totalPlayerAttackPower = playerArmPower + playerWeaponPower;
int totalEnemyDefence = enemyBodyDefence + enemyArmorDefence;

int damageAmount = totalPlayerAttackPower - (totalEnemyDefence / 2);
if (damageAmount < 0) {
 damageAmount = 0;
}
```

- 위 코드를 보면 어떤 값을 계산하는 데 어떤 값을 사용하는지의 관계 파악이 쉽다.

### 2.3 단순 나열이 아니라 , 의미 있는 것을 모아 메서드로 만들기

---

- 코드 2.3은 일련의 흐름이 모두 그냥 작성되어 있다. 계산 로직들이 단순하게 나열되어 있으면, 로직이 어디에서 시작해서 어디에서 끝나는지, 무슨 일을 하는지 알기 어렵다.
- 계산 로직이 복잡하고 거대해지면, 공격력을 계산할 때 방어력을 실수로 넣을 수도 있다.
- 이러한 상황을 막기 위해서는 의미 있는 로직을 모아 메서드로 구현하는 것이 좋다.

```java
// 플레이어의 공격력 합계 계산
int sumUpPlayerAttackPower(int playerArmPower, int playerWeaponPower) {
	return playerArmPower + playerWeaponPower;
}

// 적의 방어력 합계 계산
int sumUpEnemyDefence(int enemyBodyDefence, int enemyArmorDefence) {
	return enemyBodyDefence + enemyArmorDefence;
}

// 데미지 평가
int estimateDamage(int totalPlayerAttackPower, int totalEnemyDefence) {
	int damageAmount = totalPlayerAttackPower - (totalEnemyDefence / 2);
	if (damageAmount < 0) {
		return 0;
	}
	return damageAmount;
}
```

- 위 코드는 코드 2.3에서 공격력 계산, 방어력 계산, 데미지 계산 코드를 메서드로 추출한 코드이다.

```java
int totalPlayerAttackPower = sumUpPlayerAttackPower(playerBodyPower, 
																										playerWeaponPower);
int totalEnemyDefence = sumUpEnemyDefence(enemyBodyDefence,
																					enemyArmorDefence);
int damageAmount = estimateDamage(totalPlayerAttackPower,
																	totalEnemyDefence);
```

- 세부 계산 로직을 메서드로 감싸 흐름이 훨씬 쉽게 읽힌다.
- 또한 서로 다른 계산 작업을 각각의 메서드로 분리하여 쉽게 구분할 수 있다.
- 이와 같이 유지 보수와 변경이 쉽도록 변수의 이름과 로직을 신경 써서 작성하는 것이 설계이다.

### 2.4 관련된 데이터와 로직을 클래스로 모으기

---

```java
int hitPoint;
```

- 데미지를 입으면 히트포인트를 감소시키는 로직이 필요하다. 이는 코드 2.7처럼 어딘가에 구현되어 있을 것이다.

```java
hitPoint = hitPoint - damageAmount;
if (hitPoint < 0) {
	hitPoint = 0;
}
```

- 회복 아이템을 써서 히트포인트를 높이는 기능을 추가하고 싶을 때는 코드 2.8과 같은 로직이 어딘가에 구현 될 것이다.

```java
hitPoint = hitPoint + recoveryAmount;
if (999 < hitPoint) {
	hitPoint = 999;
}
```

- 변수와 변수를 조작하는 로직이 이곳저곳에 만들어지고 있다.
- 이러한 경우 큰 프로그램에서는 관련 로직을 찾기 위해 들이는 시간이 엄청날 것이다.
- 문제를 해결해 주는 것이 바로 클래스이다.
- 클래스는 데이터를 인스턴스 변수로 갖고, 인스턴스 변수를 조작하는 메서드를 함께 모아 놓은 것이다.
- 코드 2.9는 히트포인트와 관련한 데이터와 로직을 묶은 클래스이다.

```java
// 히트포인트(HP)를 나타내는 클래스
class HitPoint {
	private static final int MIN = 0;
	private static final int MAX = 999;
	final int value;

	HitPoint(final int value) {
		if (value < MIN) throw new IllegalArgumentException(MIN + "이상을 지정해 주세요.");
		
		if (MAX < value) throw new IllegalArgumentException(Max + "이하를 지정해 주세요.");
		
		this.value = value;	
	}

	// 데미지를 받음.
	HitPoint damage(final int damageAmount) {
		final int damaged = value - damageAmount;
		final int corrected = damaged < MIN ? MIN : damaged;
		return new HitPoint(corrected);
	}

	// 회복
	HitPoint recover(final int recoveryAmount) {
		final int recovered = value + recoveryAmount;
		final int corrected = MAX < recovered ? MAX : recovered;
		return new HitPoint(corrected);
	}
}
```

- HitPoint 클래스는 히트포인트와 관련된 로직을 담고 있다.
- 서로 밀접한 데이터와 로직을 한곳에 모아 두어 이곳저곳 찾아 다니지 않아도 된다.
- 생성자에는 0~999 범위를 벗어나는 값을 거부하는 로직을 두었다. 처음부터 잘못된 값이 유입되지 않게 만들어 조금이나마 버그로부터 안전한 클래스 구조를 만든 것이다.
- 의도를 가지고 적절하게 설계했을 때 유지 보수와 변경이 쉬워진다.
