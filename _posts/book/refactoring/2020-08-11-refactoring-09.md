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



### 11. 메서드 은폐

* 메서드가 다른 클래스에 사용되지 않을 땐 그 메서드의 반환 타입을 private로 만들자.



#### 11.1 동기

* 클래스 안에 넣은 기능이 많을수록 읽기/쓰기 메서드의 대부분은 더 이상 public으로 놔둘 이유가 없다.

#### 11.2 방법

* 주기적으로 개방도 낮출 수 있는지 확인하자.
* 각 메서드를 가능하면 private 타입으로 만들자.



### 12. 생성자를 팩토리 메서드로 전환

* 객체를 생성할 때 단순한 생성만 수행하게 해야 할 땐 생성자를 팩토리 메서드로 교체하자.



#### 12.1 동기

* 분류부호를 하위클래스로 바꿀 때 발생한다.
* 분류부호를 사용해 작성한 객체가 있는데 현 시점에서 하위클래스가 필요해졌다.
* 어느 하위클래스를 사용할지는 분류 부호에 따라 달라진다.
* 하지만 생성자는 요청된 객체의 인스턴스 반환만 할 수 있다.
* 따라서 생성자를 팩토리 메서드로 바꿔야 한다.
* 생성자가 너무 제한되는 다른 상황에서도 팩토리 메서드를 사용할 수 있다.
* 팩토리 메서드는 값을 참조로 전환을 실시하기 위해 꼭 필요하다.
* 팩토리 메서드는 매개변수의 숫자와 타입을 벗어나는 다른 생성 동작을 나타낼 때도 사용할 수 있다.



#### 12.2 방법

* 팩토리 메서드를 작성하자. 그 메서드의 내용을 기존의 생성자 호출로 수정하자.
* 모든 생성자 호출을 팩토리 메서드 호출로 바꾸자.
* 생성자를 private으로 선언하자.



#### 12.3 예제

```java
public class Employee {
	private int type;
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	Employee(int type) {
		this.type = type;
	}
}
```

* 각 분류 부호에 해당하는 Employee 클래스의 하위클래스를 작성하려 한다.
* 이를 위해선 팩토리 메서드를 다음과 같이 작성해야 한다.

```java
public class Employee {
	private int type;
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	Employee(int type) {
		this.type = type;
	}
	
	static Employee create(int type) {
		return new Employee(type);
	}
}
```

* 다음으로, 생성자 호출 부분을 전부 새 메서드 호출로 고치고 생성자를 private 타입으로 바꾸자.

```java
Employee eng = Employee.create(Employee.ENGINEER);
```



##### 12.3.1 예제: 문자열을 사용하는 하위클래스 작성

* 위에 개선된 점은 생성된 객체의 클래스에서 생성 호출의 결과를 받는 수신 메서드를 분리했다는 것이다.
* 나중에 분류 부호를 하위클래스로 전환을 적용해서 분류 부호를 Employee의 하위클래스로 전환할 경우, 팩토리 메서드를 사용하면 이 하위클래스를 클라이언트가 볼 수 없게 은폐할 수 있다.

```java
static Employee create(int type) {
    switch (type) {
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
```

* 한가지 단점은 switch 문이 생긴다는 것이다. 새 하위클래스를 추가해야 할 땐 이 switch 문을 반드시 수정해야 한다.
* 잊는 것을 방지하려면 Class.forName을 사용하는 것이 좋다.
* 우선 매개변수의 타입을 수정해야 한다.
* 매개변수 타입 변경은 기본적으로 메서드명 변경을 변형한 것이다.
* 먼저 인자로 문자열을 받는 새 메서드를 작성하자.

```java
static Employee create(String name) {
    try {
        return (Employee) Class.forName(name).newInstance();
    } catch (Exception e) {
        throw new IllegalArgumentException("객체 " + name + "를 인스턴스화할 없음");
    }
}
```

* 그런 다음 정수 타입을 매개변수로 받는 create 메서드의 코드를 앞의 새 메서드 호출로 바꾸자.

```java
static Employee create(int type) {
    switch (type) {
        case ENGINEER:
            return create("Engineer");
        case SALESMAN:
            return create("Salesman");
        case MANAGER:
            return create("Manager");
        default:
            throw new IllegalArgumentException("없는 분류 부호 값");
    }
}
```

* 이제 create 메서드 호출 부분에서 다음과 같은 명령문을 찾아 고치자.

```java
Employee eng = Employee.create(Employee.ENGINEER);
```

```java
Employee eng = Employee.create("Engineer");
```

* 여기까지 마치면 정수 타입의 인자를 받는 create 메서드를 없앨 수 있다.

* 이 방법은 Employee 클래스의 새 하위클래스를 추가할 때 create 메서드를 업데이트할 필요가 없어서 좋다.
* 그러나 이 방법은 컴파일할 때의 검사가 너무 부족하므로 오타로 인해 런타임 에러가 발생할 수도 있다.
* 이것이 중요한 문제라면 나는 바로 다음에 설명할 메서드를 작성해 사용하는데, 이때 하위클래스를 하나 추가할 때마다 새 메서드도 추가해야 한다.
* 유연성과 타입 안전성 사이의 절충점을 찾기 위한 선택의 문제다.
* 다행히 선택을 잘못하더라도 메서드를 매개변수로 전환이나 매개변수를 메서드로 전환을 적용하면 잘못된 선택을 되돌릴 수 있다.
* Class.forname을 사용하기가 망설여지는 두 번째 이유는 Class.forName으로 인해 하위클래스 이름이 클라이언트에 노출된다는 것이다.
* 다른 문자열을 사용해서 팩토리 메서드로는 다른 기능을 수행할 수 있으므로 이건 그다지 단점이라고 할 순 없다.
* 메서드 내용 직접 삽입을 적용해서 팩토리 메서드를 없애지 않는 이유가 바로 이 때문이다.



##### 12.3.2 예제: 메서드를 사용하는 하위클래스 작성

* 또 다른 방법은 직접 작성한 메서드를 사용해서 하위클래스를 은폐하는 것이다.
* 이 방법은 변하지 않는 두세 개의 하위클래스만 있을 때 사용 가능하다.
* 예컨대, abstract 타입의 Person 클래스가 있고 그 하위 클래스로 Male과 Female이 있다고 하자.
* 그러면 우선 다음과 같이 각 하위클래스에 해당하는 팩토리 메서드를 정의해야 한다.

```java
public class Person {
	static Person createMale() {
		return new Male();
	}
	
	static Person createFemale() {
		return new Female();
	}
}
```

* 생성자 호출 명령은 다음과 같다. 수정하자.

```java
Person kent = new Male();
```

```java
Person kent = Person.createMale()
```

* 이렇게 하면 상위클래스가 하위클래스 정보를 알고 있는 상태가 유지된다.
* 이를 방지하려면 Product Trader 패턴같은 더 복잡한 기법을 사용해야 한다.
* 그러나 보통 앞에 두 가지로 충분하다.



### 13. 하향 타입 변환을 캡슐화

* 메서드가 반환하는 객체를 호출 부분에서 하향 타입 반환해야 할 땐 하향 타입 변환 기능을 메서드 안으로 옮기자.



#### 13.1 동기

* 하향 타입 변환은 타입을 철저히 따지는 객체지향 언어에서 제일 귀찮은 일이다.
* 왜냐하면 하향 타입 변환은 불필요하단 느낌이 드는데다, 컴파일러가 스스로 파악해야 마땅할 것 같은 정보를 개발자가 일일이 알려줘야 하기 때문이다. 
* 하지만 타입을 알아내기 까다로울 때가 많기에 웬만해선 개발자가 직접 작성해야 한다.
* 사람들이 이터리에터를 어떤 목적으로 사용하는지 살펴보고 그 목적에 맞는 메서드를 작성하자.



#### 13.2 방법

* 메서드 호출의 결과로 반환된 값을 하향 타입 변환해야 하는 각종 상황을 찾자.
  * 이런 상황은 컬렉션이나 이터레이터를 반환하는 메서드에서 자주 볼 수 있다.
* 하향 타입 변환 코드를 그 메서드 안으로 옮기자.
  * 컬렉션을 반환하는 메서드에 컬렉션 캡슐화를 적용하자.



#### 13.2 예제

```java
Object lastReading() {
    return readings.lastElement();
}
```

```java
Reading lastReading() {
	return (Reading) readings.lastElement();
}
```



### 14. 에러 부호를 예외 통지로 교체

* 메서드가 에러를 나타내는 특수한 부호를 반환할 땐 그 부호 반환 코드를 예외 통지 코드로 바꾸자.



#### 14.1 동기

* 뭔가가 잘못되었을 때 개발자는 그 오류에 대응하는 작업을 실시해야 한다.
* 에러 찾기 루틴은 에러를 발견하면 자신을 호출한 부분에 그것을 알리며, 호출 부분이 그 에러를 상위 호출 코드로 보낼 수 있다.



#### 14.2 방법

* 확인된 예외와 미확인 예외 중 어느 것을 사용해야 할지 판단하자.
  * 호출 전에 호출하는 부분이 조건을 검사해야 한다면 미확인 예외로 하자.
  * 예외가 확인된 것이면 새 예외를 작성하거나 기존 예외를 사용하자.
* 호출 부분을 전부 찾아서 그 예외를 사용하게 수정하자.
  * 미확인 예외일 땐 호출 부분이 메서드 호출 전에 적절한 검사를 하게 하자.
  * 확인된 예외일 땐 호출 부분이 try 절 안에서 메서드를 호출하게 하자.
* 메서드 시그너처를 수정해서 새로운 용도를 반영하자.

* 확인된 예외와 미확인 예외 중 어느 것을 사용할지 판단하자.
* 그 예외를 사용하는 새 메서드를 작성하자.
* 원본 메서드의 내용을 수정해서 새 메서드를 호출하게 하자.
* 원본 메서드 호출을 전부 새 메서드 호출로 바꾸자.
* 원본 메서드를 삭제하자.



#### 14.3 예제

```java
public class Account {

	private int balance;
	
	int withdraw(int amount) {
		if(amount > balance) {
			return -1;
		} else {
			balance -= amount;
			return 0;
		}
	}
}
```

* 이 코드가 예외를 사용하게 수정하려면 우선 확인된 예외와 미확인 예외 중 어느 것을 사용할지 정해야 한다.
* 이 결정은 출금 전의 잔액이 검사하는 기능을 호출 코드가 담당하는지 출금 메서드가 담당하는지에 따라 달라진다.
* 계좌 잔액 검사가 호출 부분에서 이뤄진다면 withdraw 메서드에 잔액보다 큰 금액을 전달하면서 호출하는 건 프로그래밍 에러다.
* 프로그래밍 에러, 즉 버그는 미확인 예외를 사용해야 한다.
* 잔액 검사가 withdraw 메서드에서 이뤄진다면 예외를 반드시 인터페이스 안에 선언해야 한다.
* 이런 식으로 호출 부분에 예외를 예상하고 적절한 처리를 하게 통지하는 것이다.



##### 14.3.1 미확인 예외

* 호출 부분이 검사를 담당할 것이다.
* 반환 코드를 사용하는 부분이 없어야 한다.
* 그건 프로그래머 에러이기 때문이다.

```java
if(account.withdraw(amount) == -1) {
    handleOverdrawn();
} else {
    doTheUsualThing();
}
```

* 다음과 같이 수정한다.

```java
if(!account.canWithdraw(amount)) {
    handleOverdrawn();
} else {
    account.withdraw(amount);
    doTheUsualThing();
}
```

* 이제 에러 코드를 삭제하고 해당 에러 상황에 대한 예외를 통지해야 한다.
* 기능은 정의에 따른다는 점에서 예외적이므로 다음과 같이 조건 검사에 감시 절을 넣어야 한다.

```java
void withdraw(int amount) {
    if(amount > balance) {
        throw new IllegalArgumentException("액수가 너무 큽니다");
    } 

    balance -= amount;
}
```

* 이것은 프로그래머 에러이므로 어설션을 넣어 한결 정확하게 표시해야 한다.

```java
void withdraw(int amount) {
    Assert.isTrue("잔액이 충분합니다", amount <= balance);
    balance -= amount;
}

class Assert {
	static void isTrue(String comment, boolean test) {
		if(!test) {
			throw new RuntimeException("어설션 실패: " + comment);
		}
	}
}
```



##### 14.3.2 확인된 예외

* 확인된 예외를 사용할 땐 처리 방법이 약간 다르다.
* 우선 다음과 같이 적당한 새 예외 객체를 작성한다.

```java
class BalanceException extends Exception {
}
```

* 호출 부분을 다음과 같이 수정하자.

```java
try {
    account.withdraw(amount);
    doTheUsualThing();
} catch(BalanceException e) {
    handleOverdrawn();
}
```

* 이제 withdraw 메서드를 수정해서 앞의 예외를 사용하게 하자.

```java
void withdraw(int amount) throws BalanceException{
    if(amount > balance) {
        throw new BalanceException();
    } 

    balance -= amount;
}
```

* 이 과저에서 힘든 점은 메서드와 메서드 호출 부분을 단번에 전부 고쳐야 한다는 것이다.
* 그러지 않으면 컴파일러 에러가 발생하기 때문이다.
* 호출 부분이 많을 땐 컴파일과 테스트를 실시하지 않고 하기엔 수정이 너무 방대할 수 있다.
* 이럴 땐 임시 중개 메서드를 사용하면 된다.

```java
//호출 부분
if(account.withdraw(amount) == -1) {
    handleOverdrawn();
} else {
    doTheUsualThing();
}
//메서드
int withdraw(int amount) {
    if(amount > balance) {
        return -1;
    } else {
        balance -= amount;
        return 0;
    }
}
```

* 우선 예외 통지를 사용하는 newWithdraw 메서드를 새로 작성하자.

```java
void newWithdraw(int amount) throws BalanceException {
    if(amount > balance) {
        throw new BalanceException();
    }

    balance -= amount;
}
```

* 그리고 원본 메서드 호출을 새 메서드 호출로 고치자.

```java
try {
    account.newWithdraw(amount);
    doTheUsualThing();
} catch (BalanceException e) {
    handleOverdrawn();
}
```

* 완료했으면 원본 메서드 삭제하고 메서드명을 원래로 바꾸자.



### 15. 예외 처리를 테스트로 교체

* 호출 부분에 사전 검사 코드를 넣으면 될 상황인데 예외 통지를 사용했을 땐 호출 부분이 사전 검사를 실시하게 수정하자.



#### 15.1 동기

* 예외를 너무 많이 적용하면 좋지 않다.
* 예외처리는 예외적인 기능, 즉, 예기치 못한 에러에 사용해야 한다.
* 예외 처리를 조건문 대용으로 사용해선 안 된다.
* 호출 부분이 메서드를 호출하기 전에 당연히 조건을 검사할 것으로 예상한다면, 개발자는 테스트틀 작성해야 하고 호출 부분은 그 테스트를 사용해야 한다.



#### 15.2 방법

* 테스트를 앞에 넣고 catch 절의 코드를 if문의 적절한 절로 복사하자.
* catch 절이 실행되는지 여부가 표시되게 catch 절에 어설션을 넣자.
* catch절을 삭제하고, 다른 catch 절이 없으면 try 절도 삭제하자.



#### 15.3 예제

* 새로 생성하기엔 많은 비용이 들지만 재사용할 수 있는 각종 리소스를 관리하는 객체를 사용하겠다.
* 데이터 베이스 접속이 좋은 예
* 리소스 관리 객체에는 두 개의 리소스 풀이 있다.
* 하나는 가용 리소스 풀이고 다른 하는 할당 리소스 풀이다.
* 클라이언트가 리소스를 요청하면 리소스 관리 객체는 리소스는 넘겨주고 가용 풀에 있던 리소스를 할당 풀로 전달한다.
* 클라이언트가 리소스를 해제하면 관리 객체는 거꾸로 할당 풀의 리소스를 가용풀로 전달한다.
* 클라이언트가 리소스를 요청했는데 사용 가능한 리소스가 없으면 관리 객체는 새 리소스를 생성한다.

```java
public class ResourcePool {
	
	Stack<Resource> available;
	Stack<Resource> allocated;
	
	Resource getResource() {
		Resource result;
		
		try {
			result = available.pop();
			allocated.push(result);
			return result;
		} catch(EmptyStackException e) {
			result = new Resource();
			allocated.push(result);
			return result;
		}
	}
}
```

* 여기서 리소스 고갈은 예기치 못한 일이 아니므로 예외 처리를 사용하면 안 된다.
* 예외를 없애려면 우선 적절한 사전 테스트를 넣고 거기에 가용 리소스가 없을 때의 처리 코드를 넣는다.

```java
Resource getResource() {
    Resource result;

    if(available.isEmpty()) {
        result = new Resource();
        allocated.push(result);
        return result;
    } else {
        try {
            result = available.pop();
            allocated.push(result);
            return result;
        } catch(EmptyStackException e) {
            result = new Resource();
            allocated.push(result);
            return result;
        }
    }
}
```

* 앞의 코드에선 예외가 절대로 발생하지 않아야 한다.
* 거걸 확실히 하고자 다음과 같이 어설션을 넣는다.

```java
public class ResourcePool {
	
	Stack<Resource> available;
	Stack<Resource> allocated;
	
	Resource getResource() {
		Resource result;
		
		if(available.isEmpty()) {
			result = new Resource();
			allocated.push(result);
			return result;
		} else {
			try {
				result = available.pop();
				allocated.push(result);
				return result;
			} catch(EmptyStackException e) {
				Assert.shouldNeverReachHere("pop 실행 시에 available이 비어 있음");
				result = new Resource();
				allocated.push(result);
				return result;
			}
		}
	}
}

class Assert {
	static void shouldNeverReachHere(String message) {
		throw new RuntimeException(message);
	}
}
```

* 컴파일 후 예외가 발생하지 않으면 try절 삭제한다.

```java
public class ResourcePool {
	
	Stack<Resource> available;
	Stack<Resource> allocated;
	
	Resource getResource() {
		Resource result;
		
		if(available.isEmpty()) {
			result = new Resource();
		} else {
			result = available.pop();
		}
		
		allocated.push(result);
		return result;
	}
}
```

