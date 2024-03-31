# 7장 컬렉션: 중첩을 제거하는 구조화 테크닉

### 7.1 이미 존재하는 기능을 다시 구현하지 말기

---

- 아래코드는 for 반복문 내부에 if 조건문이 중첩되어 있어서 가독성이 좋지 않다.

```java
boolean hasPrisonKey = false;
// items는 List<Item> 자료형
for (Item each : items) {
	if (each.name.equals("감옥 열쇠")) {
		hasPrisonKey = true;
		break;
	}
}
```

- 위 기능을 아래처럼 구현할 수 있다.

```java
boolean hasPrisonKey = items.stream().anyMatch(
	item -> item.name.equals("감옥 열쇠")
);
```

- anyMatch 메서드는 자바 표준 라이브러리에 있는 컬렉션 전용 메서드이다.
- 조건을 만족하는 요소가 컬렉션 내부에 하나라도 포함되어 있으면 true를 리턴하는 역할을 한다.
- 따라서 for반복문과 if 조건문을 사용하지 않아도 된다.
- 표준 컬렉션 라이브러리에는 이와 같은 편리한 메서드가 다양하기 때문에 확인해보면 복잡하지 않은 코드를 작성할 수 있다.

### 7.2 반복 처리 내부의 조건 분기 중첩

---

- 컬렉션 내부에서 특정 조건을 만족시키는 요소에 대해서만 어떤 작업을 수행하고 싶은 경우가 있다.
- RPG에서 독 대미지를 받는 사양을 생각해보면 멤버 전원의 상태를 확인하고 중독된 상태인 경우, 히트포인트를 감소시키는 로직이 있을 수 있다.
- 아래 코드는 설계를 고려하지 않고 작성한 코드이다.

```java
for (Member member : members) {
	if (0 < member.hitPoint) {
		if (member.containsState(StateType.poison)){
			member.hitPoint -= 10;
			if (member.hitPoint <= 0) {
				member.hitPoint = 0;
				member.addState(StateType.dead);
				member.removeState(StateType.poison);
			}
		}
	}
}
```

- 먼저 살아있는지 확인하고 살아 있다면 중독상태인지 확인한다.
- 중독 상태이면 히트포인트를 10만큼 감소시키고 히트포인트가 0 이하로 떨어지면 히트포인트를 0으로 고정하고 사망 상태로 만든다.
- 위 코드는 for반복문 내부에 if 조건문이 중첩되어 가독성이 좋지 않다.

### 7.2.1 조기 컨티뉴로 조건 분기 중첩 제거하기

---

- continue는 실행하고 있는 처리를 건너 뛰고, 다음 반복으로 넘어가는 제어 구문입니다.
- 조기 리턴은 ‘조건을 만족하지 않는 경우, return으로 함수를 종료한다’는 기법이다.
- 이 방법을 응용해서 ‘조건을 만족하지 않는 경우, continue로 다음 반복으로 넘어간다’라는 기법을 활용하는 것이다.
- 아래 코드는 생존 상태를 확인하는 if 조건문은 ‘살아 있지 않다면 continue로 다음 반복으로 넘어간다’라는 형태로 변경한다.

```java
for (Member member : members) {
	// 살아 있지 않다면 continue로 다음 반복으로 넘어감.
	// 조기 continue로 변경하려면 조건을 반전해야 함.
	if (member.hitPoint == 0) continue;
	
	if (member.containState(StateType.poison)) {
		member.hitPoint -= 10;
			if (member.hitPoint <= 0) {
				member.hitPoint = 0;
				member.addState(StateType.dead);
				member.removeState(StateType.poison);;
			}
		}
}
```

- 위 코드는 아 있지 않다면 continue로 건너뛰므로, 남아 있는 로직이 실행되지 않고, 곧바로 다음 멤버를 처리한다.

```java
for (Member member : members) {
	if (member.hitPoint == 0) continue;
	if (!member.containState(StateType.poison)) continue;
	
	member.hitPoint -= 10;
	
	if (0 < member.hitPoint) continue;
	
	member.hitPoint = 0;
	member.addState(StateType.dead);
	member.removeState(StateType.poison);
}
```

- 3겹으로 중첩 되었던 if 조건문이 모두 제거되어 가독성이 좋아졌다.

### 7.2.2 조기 브레이크로 중첩 제거하기

---

- 반복 처리 제어 구문에는 continue 외에도 break가 있다.
- break는 처리를 중단하고 반복문 전체를 벗어나는 제어 구문이다.
- 조기 브레이크 역시 로직을 단순하게 만들어준다.
- 멤버가 힘을 합쳐 공격하는 ‘연계 공격’이라는 요구 사항이 추가 되었을 때 연계 공격은 공격력이 증폭되는 효과가 있지만, 성공 조건이 까다로워 계속 사용할 수 없다는 특징이 있다.

- 연계 공격의 총 대미지는 다음과 같은 로직으로 계산한다.
  - 첫 멤버부터 차례대로 연계 공격의 성공 여부를 평가
  - 연계 공격에 성공한 경우
    - 해당 멤버의 공격력 x 1.1을 대미지로 추가
  - 연계 공격에 실패한 경우
    - 후속 멤버의 연계를 평가하지 않음
  - 한 멤버의 추가 대미지가 30 이상인 경우
    - 추가되는 대미지까지 총 대미지에 합산
  - 한 멤버의 추가 대미지가 30 미만인 경우
    - 연계 공격 실패로 간주하고, 후속 멤버의 연계를 평가하지 않음

    ```java
    int totalDamage = 0;
    for (Member member : members) {
    	if (member.hasTeamAttackSucceeded()) {
    		int damage = (int)(member.attack() * 1.1);
    		if (30 <= damage) {
    			totalDamage += damage;
    		}
    		else {
    			break;
    		}
    	}
    	else {
    		break;
    	}
    }
    ```

- 아래 코드는 조기 브레이크를 활용해서 정리한 코드이다.

```java
int totalDamage = 0;
for (Member member : members) {
	if (!member.hasTeamAttackSucceeded()) break;
	
	int damage = (int)(member.attack() * 1.1);
	
	if (damage < 30) break;
	
	totalDamage += damage;
}
```

### 7.3 응집도가 낮은 컬렉션 처리

---

- 컬렉션 처리는 응집도가 낮아지기 쉽다.
- 아래 코드는 RPG 게임의 파티를 예시 코드이다.

```java
// 필드 맵과 관련된 제어를 담당하는 클래스
class FieldManager {
	// 멤버를 추가합니다.
	void addMember(List<Member> members, Member newMember) {
		if (members.stream().anyMatch(member -> member.id == newMember.id)) {
			throw new RuntimeException("이미 존재하는 멤버입니다.");
		}
		if (members.size() == MAX_MEMER_COUNT) {
		 throw new RuntimeException("이 이상 멤버를 추가할 수 없습니다.");
		}
		members.add(newMember);
	}
	
	// 파티 멤버가 1명이라도 존재하면 true를 리턴
	boolean partyIsAlive(List<Member> members) {
		return members.stream().anyMatch(member -> member.isAlive());
	}
```

- 위 클래스는 게임에서 필드 맵을 관리하는 클래스이다.
- 파티에 멤버를 추가하는 addMember 메서드와 파티에 살아 있는 멤버가 있는지 리턴하는 partyIsAlive가 정의 되어 있다.
- 필드 맵 말고도 중요한 이벤트가 발생했을 때 멤버를 추가하는 로직이 있을 수 도 있다.
- 이를 아래의 코드로 구현 했다고 해보자

```java
// 게임 중에 발생하는 특별 이벤트를 제어하는 클래스
class SpecialEventManager {
	// 멤버를 추가합니다.
	void addMember(List<Member> members, Member member) {
		members.add(member);
}
```

- 위 클래스에서도 멤버를 추가하는 메소드인 addMember가 있다.
- addMember 메서드 뿐만아니라 FieldManager.partyIsAlive 로직도 다른 클래스에 중복되어 있을 수도 있다.
- 예를 들어 아래의 코드에서 membersAreAlive 메소드는 partyIsAlive와 이름만 다를 뿐 로직의 차이는 없다. 즉 겉모습만 다른 중복 코드라고 할 수 있다.

```java
// 전투를 제어하는 클래스
class BattleManager {
	// 파티 멤버가 1명이라도 존재하는 경우 true를 리턴
	boolean membersAreAlive(List<Member> members) {
		boolean result = false;
		for (Member each : members) {
			if (each.isAlive()) {
				result = true;
				break;
			}
		}
		return result;
	}
}
```

- 이처럼 컬렉션과 관련된 작업을 처리하는 코드가 여기저기에 구현되어 응집도가 낮아져 있을 때 어떻게 해야할까?

### 7.3.1 컬렉션 처리를 캡슐화하기

---

- 컬렉션과 관련된 응집도가 낮아지는 문제는 일급 컬렉션 패턴을 사용해 해결할 수 있다.
- 일급 컬렉션이란 컬렉션과 관련된 로직을 캡슐화하는 디자인 패턴이다.
- 클래스에는 인스턴스 변수와 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정상적으로 조작하는 메서드가 있어야한다.
- 이 원리를 반영하면 일급 컬렉션은 컬렉션 자료형의 인스턴스 변수와 컬렉션 자료형의 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정삭적으로 조작하는 메서드로 구성 된다.
- 멤버 컬렉션 List<Member>를 인스턴스 변수로 가지는 클래스로 설계를 해보자

```java
class Party {
	private final List<Member> members();
	
	Party() {
		members = new ArrayList<Member>();
	}
}
```

- 멤버를 추가하는 메서드를 add라는 이름으로 구현한다.
- 아래 코드는 members의 요소가 변화하는 부수 효과가 발생한다.

```java
class Party {
	// 생략
	void add(final Member newMember) {
		members.add(newMember);
	}
}
```

- 아래 코드는 부수 효과를 막기 위해 새로운 리스트를 생성하고 해당 리스트에 요소를 추가하는 형태로 구현했다.
- 최종적으로 새로운 Party 클래스의 인스턴스를 리턴하는 것이다.

```java
class Party {
	// 생략
	Party add(final Member newMember) {
		List<Member> adding = new ArrayList<>(members);
		adding.add(newMember);
		return new Party(adding);
	}
}
```

- 위 코드를 실행하면 원래 members를 변화시키지 않아 부수 효과를 막을 수 있다.
- 아래 코드는 멤버가 1명이라도 살아 있는지 판정하는 메소드 isAlive, 멤버를 추가할 수 있는지 확인하는 메서드인 exists, isFull 을 추가한 최종 코드이다.

```java
class Party{

    static final int MAX_MEMBER_COUNT = 4;
    private final List<Member> members;

    Party() {
        members = new ArrayList<Member>();
    }

    private Party(List<Member> members){
        this.members = members;
    }

    Party add(final Member newMember){
        if(exists(newMember)){
            throw new RuntimeException("이미 파티에 참가되어 있습니다.");
        }

        if(isFull()){
            throw new RuntimeException("이 이상 멤버를 추가할 수 없습니다.");
        }

        final List<Member> adding = new ArrayList<>(members);

        adding.add(newMember);

        return new Party(adding);
    }

    boolean isAlive(){
        return members.stream().anyMatch(each -> each.isAlive());
    }

    boolean exists(final Member member){
        return members.stream().anyMatch(each -> each.id == member.id);
    }

    boolean isFull(){
        return members.size() == MAX_MEMBER_COUNT;
    }
  }
 
```

- 컬렉션과 컬렉션을 조작하는 로직을 한 클래스에 응집한 구조로 만든 것이다.

### 7.3.2 외부로 전달할 때 컬렉션의 변경 막기

---

- 파티 멤버 전원의 상태 화면을 표시하는 기능을 추가한다면 List<Member>에 접근해서 전체 데이터를 참조할 수 있어야 한다.
- 일급 컬렉션으로 설계한 Party 클래스에서 멤버 전원을 참조하려고 할 때 아래와 같이 메서드를 정의해도 될까?

```java
class Party {
	// 생략
	List<Member> members() {
		return members;
	}
}
```

- 위 처럼 인스턴스 변수를 그대로 외부에 전달하면 외부에서도 마음대로 멤버를 추가하고 제거할 수 있다.
- 따라서 Party 클래스를 응집도 높게 설계했지만 응집도가 낮을 때와 큰 차이가 없는 것이다.

```java
members = party.members();
members.add(newMember);
...
members.clear();
```

- 외부로 전달할 때에는 컬렉션 요소를 변경하지 못하도록 unmodifiableList 메서드를 사용하는 것이 좋다.

```java
class Party {
	// 생략
	/** @return 멤버 리스트(다만 요소를 외부에서 변경할 수 없다.) */
	List<Member> members() {
		return members.unmodifiableList();
	}
}
```

- unmodifiableList로 리턴되는 컬렉션은 요소를 추가하거나 제거할 수 없다.
- 따라서 Party 클래스 외부에서 마음대로 컬렉션을 조작하는 상황 자체를 방지할 수 있다.
