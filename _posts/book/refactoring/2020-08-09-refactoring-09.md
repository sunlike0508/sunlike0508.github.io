---
title:  "메서드 호출 단순화"
excerpt: "메서드 호출 단순화"
classes: wide
categories:
  - Refactoring
tags:
  - [Refactoring]
last_modified_at: 2020-08-11
---



# 메서드 호출 단순화



### 1. 메서드 변경

* 메서드명을 봐도 기능을 알 수 없을 땐 메서드명을 직관적인 이름으로 바꾸자.



#### 1.1 동기

* 코드는 컴퓨터보다 인간이 알아보기 쉽게 작성해야 한다.
* 인간이 알아보기 쉬우려면 코드에 사용된 모든 이름이 적절해야 한다.
* 이름을 잘 짓는 기술이 진정으로 노련한 프로그래머가 되는 열쇠다.



#### 1.2 동기

* 메서드 시그니처가 상위클래스나 하위클래스에 구현되어 있는지 검사하자. 만약 구현되어 있다면 각 구현부를 대상으로 다음 단계들을 실시한다.
* 새 이름으로 새 메서드를 선언하자. 코드의 원래 내용을 새 메서드로 복사하고 적절히 수정한다.
* 새 메서드를 호출하게 원본 메서드의 내용을 수정한다.
* 원본 메서드 참조 부분을 전부 찾아서 새 메서드를 참조하게 수정한다.
* 원본 메서드를 삭제 한다.



#### 1.3 예제

* 메서드 명만 바꾸는거라 패스



### 2. 매개변수 추가

* 메서드가 자신을 호출한 부분의 정보를 더 많이 알아야 할 땐 객체에 그 정보를 전달할 수 있는 매개변수를 추가하자.



#### 2.1 동기

* 매개변수 추가가 항상 좋은 것은 아니다. 충분히 다른 방법이 있을 수 있다.
* 매개변수가 늘어나면 좋지 않다. 그렇다고 절대하지 말라는 것은 아니다.



#### 2.2 방법

* 메서드 시그너처가 상위클래스나 하위클래스에 구현되어 있는지 검사하자. 구현되어 있다면 이 과정을 모든 구현부마다 실시하자.
* 추가한 매개변수를 전달받는 새 메서드를 선언하자. 원본 메서드의 내용을 새 메서드로 복사하자.
* 새 메서드를 호출하게 원본 메서드의 내용을 수정하자.
* 원본 메서드 참조 부분을 찾아서 바꾸자.
* 원본 메서드를 삭제하자.





### 3. 매개변수 제거

* 메서드가 어떤 매개변수를 더 이상 사용하지 않을 땐 그 매개변수를 삭제하자.



#### 3.1 동기

* 매개변수는 필요한 정보를 전달한다.
* 껍데기 매개변수를 제거하자.



#### 3.2 방법

* 메서드 시그니처가 상위클래스나 하위클래스에 구현되어 있는지 검사한다. 사용한다면 하지 말자.
* 제저할 매개변수가 없는 새 메서드를 선언하자.
* 원본 메서드의 내용을 새 메서드로 복사하여 수정하고 참조한 부분들 수정한다.
* 원본 메서드 삭제하자.





### 4. 상태 변경 메서드와 값 반환 메서드를 분리

* 값 반환 기능과 객체 상태 변경 기능이 한 메서드에 들어 있을 댄 질의 메서드와 변경 메서드로 분리하자.



#### 4.1 동기

* 값을 반환하는 모든 메서드는 눈에 띄는 부작용이 없어야 한다.
* 값을 반환하는 메서드가 있는데 그 메서드에 부작용이 있다면 상태 변경 부분과 값 반환 부분을 별도의 메서드로 각각 분리해야 한다.



#### 4.2 방법

* 원본 메서드와 같은 값을 반환하는 값 반환 메서드를 작성하자.
  * 원본 메서드를 관찰하여 무엇을 반환하는지 알아내자. 반환된 값이 임시적이면 임시 할당 위치를 살펴보자.
* 메서드 호출의 결과를 반환하게 원본 메서드를 수정하자.
  * 원본 메서드 안의 모든 return문은 다른 것을 반환하게 작성하지 말고 return newQuery()라고 작성해야 한다.
* 각 호출에 대해 한 번의 원본 메서드 호출을 값 반환 메서드 호출로 수정하자. 값 반환 메서드 호출 행 앞에 원본 메서드 호출을 추가하자.
* 원본 메서드를 void 타입으로 수정하고 안의 return문을 삭제하자.



#### 4.3 예제

* 보안 시스템의 침입자 이름을 알려주고 경고 메시지를 보내는 함수는 다음과 같다.
* 이 함수의 규칙은 침입자가 둘 이상일 때도 경고가 한 번만 송신되어야 한다는 점이다.

```java
String foundMiscreant(String[] people) {
    for(int i = 0; i < people.length; i++) {
        if(people[i].equals("Don")) {
            sendAlert();
            return "Don";
        }

        if(people[i].equals("John")) {
            sendAlert();
            return "John";
        }
    }

    return "";
}

void checkSecurity(String[] people) {
    String found = foundMiscreant(people);
    someLaterCode(found);
}
```

* 값 반환 코드를 상태 변경 코드와 분리하려면 우선 변경 메서드와 같은 값을 반환하되 부작용이 없는 적절한 질의 메서드를 다음과 같이 작성해야 한다.

```java
String foundPerson(String[] people) {
    for(int i = 0; i < people.length; i++) {
        if(people[i].equals("Don")) {
            sendAlert();
            return "Don";
        }

        if(people[i].equals("John")) {
            sendAlert();
            return "John";
        }
    }

    return "";
}
```

* 그런 다음, 원본 함수의 모든 return 문을 새 질의 호출로 한 번에 하나씩 수정하자. 각각을 수정할 때마다 테스트하자. 원본 메서드 수정을 모두 완료하면 다음과 같아진다.

```java
String foundMiscreant(String[] people) {
    for(int i = 0; i < people.length; i++) {
        if(people[i].equals("Don")) {
            sendAlert();
            return foundPerson(people);
        }

        if(people[i].equals("John")) {
            sendAlert();
            return foundPerson(people);
        }
    }

    return foundPerson(people);
}
```

* 이제 호출하는 메서드를 전부 수정해서 변경 메서드를 먼저 호출한 후 값 반환 메서드를 호출하게 만들자.

```java
void checkSecurity(String[] people) {
    foundMiscreant(people);
    String found = foundPerson(people);
    someLaterCode(found);
}
```

* 모든 호출 메서드를 수정했으면 다음과 같이 void 타입의 값을 반환하게 상태 변경 메서드를 수정하자.

```java
void foundMiscreant(String[] people) {
    for(int i = 0; i < people.length; i++) {
        if(people[i].equals("Don")) {
            sendAlert();
            return;
        }

        if(people[i].equals("John")) {
            sendAlert();
            return;
        }
    }
}
```

* 이제 원본 메서드명을 수정하자.

```java
void sendAlert(String[] people) {
    for(int i = 0; i < people.length; i++) {
        if(people[i].equals("Don")) {
            sendAlert();
            return;
        }

        if(people[i].equals("John")) {
            sendAlert();
            return;
        }
    }
}
```

* 앞의 코드에서 상태 변경 메서드가 값 반환 메서드의 코드를 이용하므로 코드가 중복되는 부분이 많다.
* 따라서 다음과 같이 상태 변경 메서드에 알고리즘 전환을 적용하여 이러한 중복 코드를 다음과 같이 수정하면 된다.

```java
void sendAlert(String[] people) {
    if(! foundPerson(people).equals("")) {
        sendAlert();
    }
}
```



### 5. 메서드를 매개변수로 전환

* 여러 메서드가 기능은 비슷하고 안에 든 값만 다를 땐 서로 다른 값을 하나의 매개변수로 전달받는 메서드를 하나 작성하자.



#### 5.1 동기

* 기능은 비슷하지만 몇 가지 값에 따라 결과가 달라지는 메서드가 여러 개 있을 때 각 메서드를 전달된 매개변수에 따라 다른 작업을 처리하는 하나의 메서드로 만들면 편리하다.
* 중복코드가 없어지고 매개변수 추가를 통해 다양한 것을 처리할 수 있어서 유연성도 커진다.



#### 5.2 방법

* 여러 메서드를 대체할 수 있는 매개변수 메서드를 작성하자.
* 새 메서드를 호출하도록 원본 메서드 하나를 수정하자.



#### 5.3 예제

##### 5.3.1 예제 1

```java
void tenPercentRaise() {
    salary *= 1.1;
}

void fivePercentRaise() {
    salary *= 1.05;
}
```

```java
void raise(double factor) {
    salary *= (1 + factor);
}
```

##### 5.3.2 예제 2

```java
Dollars baseCharge() {
    double result = Math.min(lastUsage(), 100) * 0.03;

    if(lastUsage() > 100) {
        result += (Math.min(lastUsage(), 200) - 100) * 0.05;
    }

    if(lastUsage() > 200) {
        result += (lastUsage() - 200) * 0.07;
    }

    return new Dollars(result);
}
```

```java
Dollars baseCharge() {
    double result = usageInRange(0,100) * 0.03;		
    result += usageInRange(100,200) * 0.05;
    result += usageInRange(200,Integer.MAX_VALUE) * 0.07;

    return new Dollars(result);
}

double usageInRange(int start, int end) {
    if(lastUsage() > start) {
        return Math.min(lastUsage(), end) - start;
    }

    return 0;
}
```



### 6. 매개변수를 메서드로 전환

* 매개변수로 전달된 값에 따라 메서드가 다른 코드를 실행할 땐 그 매개변수로 전달될 수 있는 모든 값에 대응하는 메서드를 각각 작성하자.



#### 6.1 동기

* 한 매개변수의 값이 여러 개가 될 수 있을 때 조건문 안에서 각 값을 검사하여 다른 기능을 수행하는 메서드에 적용
* 호출하는 부분은 매개변수에 값을 지정하여 무엇을 수행할지 판단해야 하므로, 여러 메서드를 작성하고 조건문은 없애는 것이 좋다.
* 그러면 조건에 따른 실행도 방지하면서 컴파일할 때 검사가 된다는 장점이 있다.
* 인터페이스도 더 명료해진다.
* 그 메서드를 사용하는 프로그래머는 클래스에서 그 메서드가 사용된 부분을 관찰하고 유효한 매개변수 값도 알아내야 한다.
* 매개변수 값이 많이 변할 가능성이 있을 때는 매개변수를 개별 메서드로 전환을 실시하면 안 된다.
* 이럴 때 필드를 그냥 전달받은 매개변수로 지정하려면 간단한 속성 쓰기 메서드를 사용하면 된다.
* 조건에 따라 다른 동작을 실행하게 해야 할 때는 조건문을 재정의로 전환을 실시해야 한다.



#### 6.2 방법

* 매개변수의 각 값에 해당하는 개별 메서드를 작성하자.
* 조건문의 각 절마다 해당되는 새 메서드 호출을 넣자.
* 각 절을 수정할 때마다 테스트를 실시하자.
* 조건문이 든 원본 메서드의 각 호출 부분을 알맞은 새 메서드 호출로 바꾸자.
* 호출 부분을 전부 고쳤으면 조건문이 든 매개변수 메서드를 삭제하자.



#### 6.3 예제

```java
public class Employee {
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	static Employee create(int type) {
		switch(type) {
			case ENGINEER:
				return new Engineer();
			case SALESMAN:
				return new Salesman();
			case MANAGER:
				return new Manager();
			default:
				throw new IllegalArgumentException("없는 분류 부호 값");
		}
	}
}
```

* 앞의 메서드는 팩토리 메서드라서 생성자를 조건문을 재정의로 전환을 적용할 수 없다.
* 왜냐하면 객체를 아직 작성하지 않았기 때문이다.
* 새 하위클래스가 별로 많지 않을 것 같으므로 명시적 인터페이스가 적절하다.
* 다음과 같이 메서드 작성

```java
static Employee createEngineer() {
    return new Engineer();
}

static Employee createSalesman() {
    return new Salesman();
}

static Employee createManager() {
    return new Manager();
}
```

* case 문 안의 각 case를 새 개별 메서드 호출로 하나씩 바꾸자.

```java
static Employee create(int type) {
    switch(type) {
        case ENGINEER:
            return Employee.createEngineer();
        case SALESMAN:
            return Employee.createSalesman();
        case MANAGER:
            return Employee.createManager();
        default:
            throw new IllegalArgumentException("없는 분류 부호 값");
    }
}
```

* 이제 원본 create 메서드를 호출하는 부분을 작업하자.

```java
Employee kent = Employee.create(ENGINEER);
```

```java
Employee kent = Employee.createEngineer();
```

* create메서드를 호출하는 부분을 모두 수정했으면 create 메서드 삭제하고 상수들 모두 삭제 하자.

```java
public class Employee {
	static Employee createEngineer() {
		return new Engineer();
	}
	
	static Employee createSalesman() {
		return new Salesman();
	}
	
	static Employee createManager() {
		return new Manager();
	}
}
```



### 7. 객체를 통째로 전달

* 객체에서 가져온 여러 값을 메서드 호출에서 매개변수로 전달할 땐 그 객체를 통째로 전달하게 수정하자.



#### 7.1 동기

* 객체가 한 객체에 든 여러 값을 메서드 호출할 때 매개변수로 전달하고 있다면 이 리팩토링 기법을 적용해야 한다.
* 이럴 땐 호출된 객체가 나중에 새 데이터 값을 필요로 할 때마다 이 메서드를 호출하는 모든 부분을 찾아서 수정해야 한다는 문제가 있다.
* 데이터를 넘겨주는 객체 자체를 넘기면 이 문제를 방지할 수 있다.
* 객체를 통째로 전달을 실시하면 매개변수 세트 변경의 편의성뿐 아니라 코드를 알아보기도 쉬워진다.
* 객체를 통째로 전달하는 방식의 단점은 값을 전달할 때 호출되는 객체가 그 값들에 의존하게 되지만 값이 추출된 객체에는 의존하지 않게 되는 점이다.
* 통 객체를 전달하면 통 객체와 호출된 객체가 서로 의존하게 된다.
* 의존성 구조를 망가뜨릴 것 같으면 객체를 통째로 전달을 실시하지 말아야 한다.



#### 7.2 방법

* 데이터가 속한 통 객체에 새 매개변수를 작성하자.
* 통 객체에서 가져와야 할 매개변수를 파악하자.
* 한 매개변수를 선택해서 메서드 안에서 그 매개변수를 참조하는 부분을 넘겨받은 통 객체 안의 적절한 메서드 호출로 바꾸자.
* 매개변수를 삭제하자.
* 통 객체에서 가져올 수 있는 모든 매개변수를 대상으로 위의 과정을 반복
* 삭제한 매개변수들을 가져오는 호출 메섣드 안의 코드를 삭제한다.



#### 7.3 예제

* 하루 동안의 최고기온과 최저기온을 기록하는 Room 객체는 다음과 같다.
* 이 온도 범위를 미리 정의한 난방 계획의 온도 범위와 비교해야 한다.

```java
public class Room {
	boolean withinPlan(HeatingPlan plan) {
		int low = daysTempRange().getLow();
		int high = daysTempRange().getHight();
		return plan.withinRange(low, high);
	}
}

class HeatingPlan {
	private TempRange range;
	
	boolean withinRange(int low, int high) {
		return (low >= range.getLow() && high <= range.getHigh());
	}
}
```

* 범위 정보를 일일이 전달할 것이 아니라 범위 객체를 통째로 전달하면 된다.
* 이런 간단한 상황일 땐 통 객체를 전달하게 수정하는 작업이 단번에 끝난다.
* 그러나 매개변수가 더 많이 필요할 때는 더 작은 단계들로 나눠서 해야 한다.
* 우선 매개변수 나열 부분에 다음과 같이 통 객체를 추가하자.

```java
public class Room {
	boolean withinPlan(HeatingPlan plan) {
		int low = daysTempRange().getLow();
		int high = daysTempRange().getHight();
		return plan.withinRange(daysTempRange(), low, high);
	}
}

class HeatingPlan {
	private TempRange range;
	
	boolean withinRange(TempRange roomRange, int low, int high) {
		return (low >= range.getLow() && high <= range.getHigh());
	}
}
```

* 그런 다음, 매개 변수 중 하나를 택해서 그것을 통 객체에 있는 메서드로 교체하자.
* 아래는 최종본

```java
public class Room {
	boolean withinPlan(HeatingPlan plan) {
		return plan.withinRange(daysTempRange());
	}
}

class HeatingPlan {
	private TempRange range;
	
	boolean withinRange(TempRange roomRange) {
		return (roomRange.getLow() >= range.getLow() 
                && roomRange.getHigh() <= range.getHigh());
	}
}
```

* 객체를 통째로 전달하는 방법에 익숙해지면, 나중에 통 객체로 기능을 옮겨야 작업하기 편하다.

```java
public class Room {
	boolean withinPlan(HeatingPlan plan) {
		return plan.withinRange(daysTempRange());
	}
}

class HeatingPlan {
	private TempRange range;
	
	boolean withinRange(TempRange roomRange) {
		return range.includes(roomRange);
	}
}

class TempRange {
	boolean includes(TempRange arg) {
		return arg.getLow() >= this.getLow() && arg.getHigh() <= this.getHigh();
	}
}
```



### 8. 매개변수 세트를 메서드로 전환

* 객체가 A 메서드를 호출해서 그 결과를 B 메서드에 매개변수로 전달하는데, 결과를 매개변수로 받는 B 메서드도 직접 A 메서드를 호출할 수 있을 땐 매개변수를 없애고 A 메서드를 B 메서드가 호출하게 하자.



#### 8.1 동기

* 메서드가 매개변수로 전달받는 값을 다른 방법으로도 가져올 수 있다면, 그 방법을 택해야 한다.
* 매개변수 나열이 길면 코드가 복잡해지므로 가능하면 매개변수를 줄여야 한다.
* 전달할 매개변수를 줄이려면 같은 계산을 수신 메서드도 할 수 있는지 검사해야 한다.
* 객체가 자신의 메서드를 호출하지만 호출한 메서드의 매개변수가 계산에 전혀 사용되지 않는다면, 그 계산을 별도의 메서드로 만들고 매개변수를 삭제할 수 있다.
* 호출하는 객체를 참조하는 다른 객체에 있는 메서드를 호출할 때도 마찬가지다.
* 나중에 메서드를 매개변수 메서드로 바꿀 가능성에 대비해 매개변수를 두는 경우도 가끔 있는데 이럴 때는 삭제하자.



#### 8.2 방법

* 필요한다면 매개변수를 사용한 계산 부분을 별도의 메서드로 빼내자.
* 메서드 안의 매개변수 사용 부분을 추출한 메서드 호출로 수정하자.
* 매개변수를 대상으로 매개변수 제거를 실시하자.



#### 8.3 예제

* 할인 주문 에제를 현실성 없게 변형한 코드는 다음과 같다.

```java
public class Test {
	private int quantity;
	private int itemPrice;

	public double getPrice() {
		int basePrice = quantity * itemPrice;
		int discountLevel;
		
		if(quantity > 100) {
			discountLevel = 2;
		} else {
			discountLevel = 1;
		}
		
		double finalPrice = discountedPrice(basePrice, discountLevel);
		
		return finalPrice;
	}

	private double discountedPrice(int basePrice, int discountLevel) {
		if(discountLevel == 2) {
			return basePrice * 0.1;
		}
		
		return basePrice * 0.05;
	}
}
```

* 우선 할인 등급 계산 부분을 메서드로 추출하자.

```java
public class Test {
	private int quantity;
	private int itemPrice;

	public double getPrice() {
		int basePrice = quantity * itemPrice;
		int discountLevel = getDiscountLevel();
		
		double finalPrice = discountedPrice(basePrice, discountLevel);
		
		return finalPrice;
	}

	private int getDiscountLevel() {
		if(quantity > 100) {
			return 2;
		} 
		
		return 1;
	}

	private double discountedPrice(int basePrice, int discountLevel) {
		if(discountLevel == 2) {
			return basePrice * 0.1;
		}
		
		return basePrice * 0.05;
	}
}
```

* 그런 다음 discountedPrice 메서드 안의 매개변수 사용 부분을 전부 바꾼다.

```java
public class Test {
	private int quantity;
	private int itemPrice;

	public double getPrice() {
		int basePrice = quantity * itemPrice;
		double finalPrice = discountedPrice(basePrice);
		
		return finalPrice;
	}

	private int getDiscountLevel() {
		if(quantity > 100) {
			return 2;
		} 
		
		return 1;
	}

	private double discountedPrice(int basePrice) {
		if(getDiscountLevel() == 2) {
			return basePrice * 0.1;
		}
		
		return basePrice * 0.05;
	}
}
```

* 나머지 basePrice에 대해서도 변경

```java
public class Test {
	private int quantity;
	private int itemPrice;
	
	private double getPrice() {
		if(getDiscountLevel() == 2) {
			return getBasePrice() * 0.1;
		}
		
		return getBasePrice() * 0.05;
	}

	private int getDiscountLevel() {
		if(quantity > 100) {
			return 2;
		} 
		
		return 1;
	}
	
	private int getBasePrice() {
		return quantity * itemPrice;
	}
}
```



### 9. 매개변수 세트를 객체로 전환

* 여러 개의 매개변수가 항상 붙어 다닐 땐 그 매개변수들을 객체로 바꾸자.



#### 9.1 동기

* 특정 매개변수들이 늘 함께 전달되는 경우를 흔히 볼 수 있다.
* 여러 메서드가 한 클래스나 여러 클래스에서 이 매개변수 집합을 사용할 가능성이 있다.
* 이런 클래스들은 데이터 뭉치이므로 그 모든 데이터가 든 객체로 바꿀 수 있다.
* 데이터를 그룹으로 묶으려면 이 매개변수들을 객체로 바꾸는 것이 좋다.
* 새 객체에 정의된 속성 접근 메서드로 인해 코드의 일관성도 개선되고, 결과적으로 코드를 알아보거나 수정하기도 쉬워진다.
* 더불어 매개변수를 한 덩이로 만들면 기능을 새 클래스로 옮길 수 있어서 훨씬 좋다.
* 메서드 안에 매개변수 값에 대한 공통적인 조작을 넣는 경우가 많다.
* 이 동작을 새 객체로 옮기면 상당량의 중복 코드를 없앨 수 있다.



#### 9.2 방법

* 대체할 매개변수 그룹에 해당하는 새 클래스를 작성하고, 그 클래스를 변경불가로 만들자.
* 새 데이터 뭉치에 매개변수 추가를 적용하자. 새 매개변수에 기본 값을 사용하자.
  * 호출 부분이 많으면 기존 시그너처를 그대로 두고 새 메서드를 호출하게 하자. 그 리팩토링을 기존 메서드에 먼저 적용하자. 그런 다음 호출 부분들을 하나씩 옮긴 후 기존 메서드는 삭제하면 안된다.
* 데이터 뭉치 안의 각 매개변수마다 시그너처에서 해당 매개변수를 삭제하자. 그 값 대신 매개변수 객체를 사용하게 호출 부분과 메서드 내용을 수정하자.
* 매개변수 삭제를 전부 완료했으면, 메서드 이동을 적용하여 매개변수 객체로 옮길 수 있는 기능을 찾자.
  * 매개변수 객체로 옮길 수 있는 기능은 메서드 전체일 수도 있고 일부분일 수도 있다. 메서드의 일부분일 겨우에는 우선 메서드 추출을 실시한 후 새로 빼낸 메서드를 옮기면 된다.



#### 9.3 예제

```java
public class Entry {
	private double value;
	private Date chargeDate;
	
	Entry (double value, Date chargeDate) {
		this.value = value;
		this.chargeDate = chargeDate;
	}
	
	Date getDate() {
		return chargeDate;
	}
	
	double getValue() {
		return value;
	}
}
```

* Account 클래스엔 입금액 컬렉션이 들어 있고, 두 날짜 사이의 계좌 입출금 현황을 알아내는 매서드가 들어 있다.

```java
public class Account {
	private Vector<Entry> entries = new Vector<Entry>();
	
	double getFlowBetween(Date start, Date end) {
		double result = 0;
		Enumeration<Entry> e = entries.elements();
		
		while(e.hasMoreElements()) {
			Entry each = e.nextElement();
			
			if(each.getDate().equals(start) || each.getDate().equals(end)
					|| (each.getDate().after(start) && each.getDate().before(end))) {
				result += each.getValue();
			}
		}
		
		return result;
	}
}
```

```java
/* 클라이언트 코드 */
double flow = anAccount.getFlowBetween(startDate, endDate);
```

* 개시일과 폐쇄일, 큰 금액과 작은 금액 같이 범위를 나타내는 값 쌍이 얼마나 많은 부분에 들어 있을지 모른다.
* 이럴 때 범위패턴을 사용한다.
* 다음과 같이 범위를 처리하는 단순 데이터 클래스를 선언한다.

```java
public class DateRange {
	private final Date start;
	private final Date end;
	
	DateRange (Date start, Date end) {
		this.start = start;
		this.end = end;
	}
	
	Date getStart() {
		return start;
	}
	
	Date getEnd() {
		return end;
	}
}
```

* DateRange 클래스를 변경불가로 만든다.
* 이러면 별칭 버그가 방지되어 좋다.
* 자바의 매개변수는 값을 이용한 전달 방식을 사용하는데 클래스를 변경불가로 만들면 그러한 자바 매개변수의 원리를 흉내낼 수 있으므로, 이 리팩토링 기법을 적용하려면 반드시 DateRange 클래스를 변경불가로 만들어야 한다.

* 다음으로 getFlowBetween 메서드의 매개변수 나열 부분에 다음과 같이 DateRange 클래스를 추가하자.
* 그리고 매개변수 start, end를 삭제하고 호출 부분 변경하자.

```java
public class Account {
	private Vector<Entry> entries = new Vector<Entry>();
	
	double getFlowBetween(DateRange range) {
		double result = 0;
		Enumeration<Entry> e = entries.elements();
		
		while(e.hasMoreElements()) {
			Entry each = e.nextElement();
			
			if(each.getDate().equals(range.getStart()) 
			|| each.getDate().equals(range.getEnd())
			|| (each.getDate().after(range.getStart()) 
				&& each.getDate().before(range.getEnd()))) {
				result += each.getValue();
			}
		}
		
		return result;
	}
}
/* 클라이언트 */
double flow = anAccount.getFlowBetween(new DateRange(startDate, endDate));
```

* 이것으로 매개변수 나열 부분을 객체로 전환하는 작업은 마쳤지만, 기능을 새 객체의 다른 메서드로 옮기는 작업을 추가로 실시한다면 이 리팩토링의 결과가 더욱 빛날 것이다.
* 메서드 추출, 메서드 이동을 적용하면 코드가 다음과 같아 진다.

```java
public class Account {
	private Vector<Entry> entries = new Vector<Entry>();
	
	double getFlowBetween(DateRange range) {
		double result = 0;
		Enumeration<Entry> e = entries.elements();
		
		while(e.hasMoreElements()) {
			Entry each = e.nextElement();
			
			if(range.includes(each.getDate())) {
				result += each.getValue();
			}
		}
		
		return result;
	}
}

public class DateRange {
	...
	
	boolean includes(Date arg) {
		return arg.equals(start) || arg.equals(end) 
					|| (arg.after(start) && arg.before(end));
	}
}
```



### 10. 쓰기 메서드 제거

* 생성할 때 지정한 필드 값이 절대로 변경되지 말아야 할 땐 그 필드를 설정하는 모든 쓰기 메서드를 삭제하자.



#### 10.1 동기

* 쓰기 메서드가 있다는 건 필드 값을 변경할 수 있다는 얘기다. 
* 객체가 생성된 후에는 필드가 변경되지 말아야 한다면, 쓰기 메서드를 작성하지 않아야 한다.
* 그렇게 하면 확실히 의도가 달성되고 필드가 수정될 가능성을 차단할 수 있다.
* 이 기법은 프로그래머가 간접적인 변수 접근을 맹목적으로 이용할 때 실시해야 한다.



#### 10.2 방법

* 쓰기 메서드가 생성할 때나 생성자가 호출하는 메서드에서만 호출되는지 검사하자.
* 쓰기 메서드가 생성자 안이나 생성자가 호출한 메서드 안에서만 호출되는지 검사하자.
* 변수에 직접 접근할 수 있게 생성자를 수정하자.
  * 상위클래스의 private 필드를 설정하는 하위클래스가 있으면 생성자를 변수에 직접 접근하게 수정할 수 없다. 이럴 땐 상위클래스에 private 필드 값을 설정하는 protected 메서드를 넣어야 한다. 상위클래스 메서드명은 쓰기 메서드와 혼동되지 않는 이름으로 정하자.
* 쓰기 메서드를 삭제하자.



#### 10.3 예제

```java
public class Account {
	private String id;
	
	Account (String id) {
		setId(id);
	}
	
	void setId(String arg) {
		id = arg;
	}
}
```

* 다음과 같이 바꾼다.

```java
public class Account {
	private String id;
	
	Account (String id) {
		this.id = id;
	}
}
```

* 문제는 코드를 조금 변형하면 드러난다.
* 첫 번째 문제는 매개변수로 계산을 수행할 때 드러난다.

```java
public class Account {
	private String id;
	
	Account (String id) {
		setId(id);
	}
	
	void setId(String arg) {
		id = "ZZ" + arg;
	}
}
```

* 앞의 코드처럼 수정이 간단할 때는 생성자를 수정해도 되지만, 수정이 복잡하거나 별도의 메서드에서 호출해야 할 때는 따로 메서를 하나 작성해야 한다.
* 이때 메서드 이름은 의도가 확실히 드러나게 정해야 한다.

```java
public class Account {
	private String id;
	
	Account (String id) {
		initializeId(id);
	}
	
	void initializeId(String arg) {
		id = "ZZ" + arg;
	}
}
```

* 이번에는 하위클래스가 상위클래스의 private 변수를 초기화하는 다음 까다로운 예제를 보자.

```java
public class InterestAccount extends Account{
	private double interestRate;
	
	InterestAccount(String id, double rate) {
		setId(id);
		interestRate = rate;
	}
}
```

* 문제는 id 변수에 직접 접근해서 값을 설정할 수 없다는 것이다.
* 최선의 해결 방법은 상위클래스 생성자를 사용하는 것이다.

```java
public class InterestAccount extends Account{
	private double interestRate;
	
	InterestAccount(String id, double rate) {
		super(id);
		interestRate = rate;
	}
}
```

* 혹시 그 방법을 사용할 수 없을 땐, 다음과 같이 이름이 잘 지어진 메서드를 사용하는 게 최선이다.

```java
public class InterestAccount extends Account{
	private double interestRate;
	
	InterestAccount(String id, double rate) {
		initializeId(id);
		interestRate = rate;
	}
}
```

* 까다로운 또 한 가지 예는 다음과 같이 컬렉션 값을 설정하는 것이다.

```java
public class Person {
	private Vector courses;
	
	Vector getCourses() {
		return courses;
	}
	
	void setCourses(Vector arg) {
		courses = arg;
	}
}
```

* 쓰기메서드를 추가 및 삭제 기능으로 바꿔야 한다.
* 그렇게 하는 방법은 컬렉션 캡슐화를 참고한다.

















































