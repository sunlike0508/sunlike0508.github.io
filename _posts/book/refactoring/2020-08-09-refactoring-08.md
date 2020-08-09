---
title:  "조건문 간결화"
excerpt: "조건문 간결화"
classes: wide
categories:
  - Refactoring
tags:
  - [Refactoring]
last_modified_at: 2020-08-09
---



# 조건문 간결화



### 1. 조건문 쪼개기

* 복잡한 조건문이 있을 땐 if, then, else 부분을 각각 메서드로 빼내자.



#### 1.1 동기

* 큰 덩어리의 코드를 잘게 쪼개고 각 코드 조각을 용도에 맞는 이름의 메서드 호출로 바꾸면 코드의 용도가 분명히 드러난다.



#### 1.2 방법

* if절을 별도의 메서드로 빼내자
* then절과 else절을 각각의 메서드로 빼내자.

* 조건문이 여러 겹일 땐 먼저 여러 겹의 조건문을 감시 절로 전환기법부터 실시하자.
* 그 결과가 만족스럽지 않으면 조건문을 쪼개자.



#### 1.3 예제

* 겨울인지 여름인지에 따라 달라지는 난방비를 계산하는 다음과 같은 코드가 있다.

```java
double chargeHeatingCost(Date date) {
    if(date.before(SUMMER_START) || date.after(SUMMER_END)) {
        charge = quantity * winterRate + winterServiceCharge;
    } else {
        charge = quantity * summerRate;
    }
    
    return charge;
}
```

* 조건식과 각 분기 절을 다음과 같이 각각의 메서드로 빼낸다.

```java
double chargeHeatingCost(Date date) {
    if(notSummer(date)) {
        charge = winterCharge(quantity);
    } else {
        charge = summerCharge(quantity);
    }
    
    return charge;
}

private boolean notSummer(Date date) {
    return date.before(SUMMER_START || date.after(SUMMER_END));
}

private double summerCharge(int quantity) {
    return quantity * summerRate;
}

private double winterCharge(int quantity) {
    return quantity * winterRate + winterServiceCharge;
}
```



### 2. 중복 조건식 통합

* 여러 조건 검사식의 결과가 같을 땐 하나의 조건문으로 합친 후 메서드로 빼내자.



#### 2.1 동기

* 조건식을 합치면 여러 검사를 OR 연산자로 연결해서 실제로 하나의 검사수행을 표현해서 무엇을 검사하는지 더 확실히 이해할 수 있다.
* 통합하면 메서드 추출을 적용할 기반을 마련한다.



#### 2.2 방법

* 모든 조건문에 부작용이 없는지 검사하자
* 여러 개의 조건문을 논리 연산자를 사용해 하나의 조건문으로 바꾸자.
* 합친 조건문에 메서드 추출 적용을 고려하자.



#### 2.3 예제

```java
double disabilityAmount() {
    if(seniority < 2) {
        return 0;
    }

    if(monthDisabled > 12) {
        return 0;
    }

    if(isPartTime) {
        return 0;
    }

    // 장애인 공제액 산출
}
```

* 다음과 같이 하나로 통합

```java
double disabilityAmount() {
    if(seniority < 2 || monthDisabled > 12 || isPartTime) {
        return 0;
    }

    // 장애인 공제액 산출
}
```

* 메서드 추출

```java
double disabilityAmount() {
    if(isNotEligibleForDisability()) {
        return 0;
    }

    // 장애인 공제액 산출
}

private boolean isNotEligibleForDisability(){
    return seniority < 2 || monthDisabled > 12 || isPartTime;
}
```



### 3. 조건문의 공통 실행 코드 빼내기

* 조건문의 모든 절에 같은 실행 코드가 있을 땐 같은 부분을 조건문 밖으로 빼자.



#### 3.1 방법

* 조건에 상관없이 공통적으로 실행되는 코드를 찾자.
* 공통 코드가 조건문의 앞 절에 있을 땐 조건문 앞으로 빼자.
* 공통 코드가 조건문의 끝 절에 있을 땐 조건문 뒤로 빼자.
* 공통 코드가 조건문의 중간 절에 있을 땐 앞뒤의 코드와 위치를 바꿔도 되는지 판단하자. 그래서 바꿔도 된다면 조건문의 앞이나 끝 절로 뺀 후 앞의 단계처럼 조건문 앞이나 뒤로 빼자.
* 공통 코드 명령이 둘 이상일 땐 메서드로 만들자.



#### 3.2 예제

```java
if(isSpecialDeal()) {
    total = price * 0.95;
    send();
} else {
    total = price * 0.98;
    send();
}
```

* send 메서드 밖으로 뺀다.

```java
if(isSpecialDeal()) {
    total = price * 0.95;
} else {
    total = price * 0.98;
}

send();
```

* try구간과 catch 구간 안의 예외 발생 명령 뒤에 공통적으로 들어 있으면, 그 코드를 final 구간으로 옮긴다.



### 4. 제어 플래그 제거

* 논리 연산식의 제어 플래그 역할을 하는 변수가 있을 땐 그 변수를 break 문이나 return 문의로 바꾸자.



#### 4.1 동기

* 제어 플래그를 없애면 의외로 많은 것을 할 수 있고 조건문의 진정한 의도를 쉽게 파악할 수 있다.



#### 4.2 방법

* 논리문을 빠져나오게 하는 제어 플래그 값을 찾자.
* 그 제어 플래그 값을 대입하는 코드를 break나 continue로 바꾸자.



#### 4.3 예제

##### 4.3.1 간단한 제어 플래그를 break문으로 교체

```java
void checkSecurity(String[] people) {
    boolean found = false;

    for(int i = 0; i < people.length; i++) {
        if(!found) {
            if(people[i].equals("Don")) {
                sendAlert();
                found = true;
            }

            if(people[i].equals("John")) {
                sendAlert();
                found = true;
            }
        }
    }
}
```

* 하나씩 break로 바꾼다. 그리고 플래그 참조 삭제. 아래는 최종본

```java
void checkSecurity(String[] people) {
    for(int i = 0; i < people.length; i++) {
        if(people[i].equals("Don")) {
            sendAlert();
            break;
        }

        if(people[i].equals("John")) {
            sendAlert();
            break;
        }
    }
}
```



##### 4.3.2 제어 플래그를 return 문으로 교체

* 리팩토링 기법의 다른 방식은 return 문을 사용하는 것이다.

```java
void checkSecurity(String[] people) {
    String found = "";

    for(int i = 0; i < people.length; i++) {
        if(found.equals("")) {
            if(people[i].equals("Don")) {
                sendAlert();
                found = "Don";
            }

            if(people[i].equals("John")) {
                sendAlert();
                found = "John";
            }
        }
    }

    someLaterCode(found);
}
```

* 여기서 found 변수를 두 가지 역할을 한다. 결과를 나타내기도 하고 제어 플래그 역할도 한다.
* 이럴 땐 found 변수를 알아내는 코드를 메서드로 빼내자.
* 그리고 제어 플래그를 return문으로 고치자.

```java
void checkSecurity(String[] people) {
    String found = foundMiscreant(people);
    someLaterCode(found);
}

String foundMiscreant(String[] people) {
    String found = "";

    for(int i = 0; i < people.length; i++) {
        if(found.equals("")) {
            if(people[i].equals("Don")) {
                sendAlert();
                return "Don";
            }

            if(people[i].equals("John")) {
                sendAlert();
                return "John";
            }
        }
    }

    return "";
}
```

* 값을 반환하지 않을 때도 return 문 방식을 사용할 수 있다.
* 이 메서드는 아직 부작용이 있다.
* 상태 변경 메서드와 값 반환 메서드 분리 기법을 실시해야 한다. 차후에 나온다.



### 5. 여러 겹의 조건문을 감시 절로 전환

* 메서드에 조건문이 있어서 정상적인 실행 경로를 파악하기 힘들 땐 모든 특수한 경우에 감시 절을 사용하자.



#### 5.1 동기

* 조건식은 주로 두 가지 형태를 띤다.

* 어느 한 경로가 정상적인 동작의 일부인지 검사하는 형태
* 조건식 판별의 한 결과만 정상적인 동작을 나타내고 나머지는 비정상적인 동작을 나타내는 형태

* 조건문의 의도는 드러나야 한다.
* 만약 둘다 정상 동작의 일부분이라면 if절과 else절로 구성된 조건문을 사용하고, 조건문이 특이한 조건이라면 그 조건을 검사해서 조건이 true일 경우 반환하자, 이런 식의 검사를 감시 절이라고 한다.
* 여러 겹의 조건문을 감시 절로 전환 기법의 핵심은 강조 부분이다.
* if-then-else문을 사용하면 if 절과 else절의 비중이 동등하다.
* 따라서 코드를 보는 사람은 if절과 else 절의 비중과 같다고 판단하게 된다.
* 그와 달리, 감시 절은 "이것은 드문 경우이니 이 경우가 발생하면 작업을 수행한 후 빠져나와라"하고 명령한다.
* 메서드에 진입점과 이탈점이 하나씩만 있다고 배운 프로그래머와 공동으로 작업할 대는 주로 여러 겹의 조건문을 감시 절로 전환 기법을 사용한다.



#### 5.2 방법

* 조건 절마다 감시 절을 넣자.
  * 그 감시 절은 값을 반환하거나 예외를 통지한다.
  * 모든 감시 절의 결과가 같다면 중복 조건식 통합 기법을 실시하자.



#### 5.3 예제

```java
double getPayAmout() {
    double result;

    if(isDead) {
        result = deadAmount();
    } else {
        if(isSeparated) {
            result = separatedAmount();
        } else {
            if(isRetired) {
                result = retiredAmount();
            } else {
                result = normalPayAmount();
            }
        }
    }

    return result;
}
```

* 하나씩 감시절을 사용한다. 아래는 최종본

```java
double getPayAmout() {
    if(isDead) {
        return deadAmount();
    }

    if(isSeparated) {
        return separatedAmount();
    }

    if(isRetired) {
        return retiredAmount();
    }

    return normalPayAmount();
}
```



##### 5.3.1 조건문을 역순으로 만들기

```java
public double getAdjustedCapital() {
    double result = 0.0;

    if(capital > 0.0) {
        if(intRate > 0.0 && duration > 0.0) {
            result = (income / duration) * ADJ_FACTOR;
        }
    }

    return result;
}
```

* 조건문 역순으로 한다.
* 감시 절의 return문에 명시적 값을 붙인다.
* 아래는 최종

```java
public double getAdjustedCapital() {		
    if(capital <= 0.0) {
        return 0.0;
    }

    if(intRate <= 0.0 || duration <= 0.0) {
        return 0.0;
    }

    return (income / duration) * ADJ_FACTOR;
}
```



### 6. 조건문을 재정의로 전환

* 객체 타입에 따라 다른 기능을 실행하는 조건문이 있을 땐 조건문의 각 절을 하위클래스의 재정의 메서드 안으로 옮기고 원본 메서드는 abstract 타입으로 수정하자.



#### 6.1 동기

* 재정의의 본질은 타입에 따라 기능이 달라지는 여러 객체가 있을 때 일일이 조건문을 작성하지 않아도 다형적으로 호출되게 할 수 있다는 것이다.
* 그래서 if문 switch문은 객체지향 프로그램에 별로 사용하지 않는다.
* 다형성 개념을 적용한 재정의 방식을 사용하면 많은 장점이 있다.



#### 6.2 방법

* 조건문을 재정의로 전환기법을 적용하기 전에 필요한 상속 구조부터 만들어야 한다.
* 앞에서 배운 리팩토링 기법의 적용으로 어쩌면 이미 적절한 상속 구조가 형성되어 있을지도 모른다.
* 하지만 그렇지 않다면 적절한 상속 구조로 만들어야 한다.
* 상속 구조를 만드는 방법은 두 가지다.
  1. 분류 부호를 하위클래스로 전환
  2. 분류 부호를 상태/전략 패턴으로 전환
* 하위 클래스로 만드는 것이 가장 간단하다.
* 그러나 객체가 생성된 후 분류 부호를 수정하면 분류 부호를 하위클래스나 상태/전략 패턴으로 바꿀 수 없다.
* 다른 이유로 이 클래스에 이미 하위클래스를 작성했다면, 상태/전략 패턴으로 전환하는 기법을 사용해야 한다.
* 여러 개의 case 문이 하나의 분류 부호 값에 따라 다른 코드를 실행할 땐 그 분류 부호를 대체할 하나의 상속 구조를 만들어야 한다.



#### 6.3 예제

```java
public class Employee {
	private EmployeeType type;
	
	int payAmount() {
		switch(getType()) {
			case EmployeeType.ENGINEER:
				return monthlySalary;
			case EmployeeType.SALESMAN:
				return monthlySalary + commission;
			case EmployeeType.MANAGER:
				return monthlySalary + bonus;
			default:
				throw new RuntimeException("없는 사원입니다");
		}
	}
	
	int getType() {
		return type.getTypeCode();
	} 
}
```

* 하위클래스로 만들 EmployeeType 클래스로 옮겨야 한다.

```java
public class EmployeeType {
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	int payAmount(Employee emp) {
		switch (getTypeCode()) {
			case EmployeeType.ENGINEER:
				return emp.getMonthlySalary();
			case EmployeeType.SALESMAN:
				return emp.getMonthlySalary() + emp.getCommission();
			case EmployeeType.MANAGER:
				return emp.getMonthlySalary() + emp.getBonus();
			default:
				throw new RuntimeException("없는 사원입니다");
		}
	}
}
```

* Employee 클래스의 데이터가 필요하므로 Employee 클래스를 인자로 전달해야 한다.
* 이 데이터 중 일부는 Employee 타입의 객체로 옮기게 될 수도 있지만 그건 다른 리팩토링 기법으로 처리할 문제다.
* 이것을 컴파일할 때 Employee 클래스 안의 payAmount 메서드를 수정해서 다음과 같이 새 클래스로 위임하게 만들자.

```java
public class Employee {
	private EmployeeType type;
	
	int payAmount() {
		return type.payAmount(this);
	}
}
```

* case 문을 정리한다.
* case 문의 ENGINEER 절 코드를 Engineer 클래스로 복사한다.

```java
public class Engineer extends EmployeeType{
	int payAmount(Employee emp) {
		return emp.getMonthlySalaray();
	}
}
```

* 새로 만든 Engineer 메서드는 ENGINEER에 해당하는 case 문 전체를 재정의한다.
* 나머지도 다 똑같이 한다.

```java
public abstract class EmployeeType {
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	abstract int payAmount(Employee emp);
}
```



### 7. Null 검사를 널 객체에 위임

* null 값을 검사하는 코드가 계속 나올 땐 null 값을 널 객체로 만들자.



#### 7.1 동기

* 재정의의 본질은 어떤 종류인지를 객체에 일일이 물어서 그 응답에 따라 실행할 기능을 호출하는 것이 아니라, 묻지도 따지지도 않고 기능을 곧바로 호출하는 것이다.
* 객체는 타입에 따라 그에 맞는 기능을 수행한다.
* null 값이 저장된 필드가 있을 땐 재정의하면 비교적 이해하기 힘들다.



#### 7.2 방법

* 원본 클래스 안에 널 객체 역할을 할 하위클래스를 작성하자. 원본 클래스와 널 클래스 안에 isNull 메서드를 작성하자. 원본 클래스의 isNull 메서드는 false를 반환해야 하고, 널 클래스의 isNull 메서드는 true를 반환해야 한다.
  * isNull 메서드를 넣을 Nullable 인터페이스를 작성하면 좋을 때도 있다.
  * 아니면, 검사 인터페이스로 null 여부를 검사하는 방법도 있다.
* 원본 객체에 요청하면 null을 반환할 수 있는 부분을 전부 찾자. 그래서 그 부분을 널 객체로 바꾸자.
* 원본 타입의 변수를 null과 비교하는 코드를 전부 찾아서 isNull 호출로 바꾸자.
  * 원본 클래스와 클라이언트를 한 번에 하나씩 수정하고 그때마다 컴파일과 테스트를 실시하면 된다.
  * null이 나오지 말아야 할 곳에 null을 검사하는 어설셜을 몇 개 넣으면 좋다.
* 클라이언트가 null이 아니면 한 메서드를 호출하고 null이면 다른 메서드를 호출하는 case문을 전부 찾자.
* 각 case 문마다 널 클래스 안의 해당 메서드를 다른 기능의 메서드로 재정의하자.
* 재정의 메서드를 사용하는 부분에서 조건문을 삭제하고 컴파일과 테스트를 실시하자.



#### 7.3 예제

* 공공설비 업체는 공공 설비 서비스를 이용하는 주택가와 아파트 단지 등의 지역을 파악하고 있다.
* 한 지역에 있는 고객은 반드시 하나다.

```java
public class Site {
	Customer customer;
	
	Customer getCustomer() {
		return customer;
	}
}

public class Customer {
	String name;
	
	public String getName() {
		return name;
	}
	
	public BillingPlan getPlan() {
	}
	
	public PaymentHistory getHistory() {
		
	}
}

public class PaymentHistory {
	int getWeeksDelinquentInLastYear() {
	}
}
```

* 클라이언트는 다음과 같은 데이터에 접근할 수 있다.
* 그러나 고객에 없는 지역도 간혹 있다.
* 어떤 사람이 다른 지역으로 이사한 후 거기로 누가 이사해 왔는지 아직 파악하지 않았을 수도 있다.
* 이런 상황이 발생할 수 있기에 Customer 클래스를 사용하는 코드에 다음과 같이 null처리를 넣어야 한다.

```java
Site site = new Site();
Customer customer = site.getCustomer();
BillingPlan plan;

if(customer == null) {
    plan = BillingPlan.basic();
} else {
    plan = customer.getPlan();
}

String customerName;

if(customer == null) {
    customerName = "occupant";
} else {
    customerName = customer.getName();
}

int weeksDelinquent;

if(customer == null) {
    weeksDelinquent = 0;
} else {
    weeksDelinquent = customer.getHistory().getWeeksDelinquentInLastYear();
}
```

* 이때 Site 클래스와 Customer 클래스가 많은 부분에 사용되고, 그 모든 부분에서 null을 검사해서 null을 발견할 때마다 같은 작업을 수행해야 할 수도 있다.
* 따라서 널 객체가 필요하다.

* 우선 다음과 같이 NullCustomer 클래스를 작성하고 널 검사 메서드를 호출하게 Customer 클래스를 수정하자.

```java
public class NullCustomer extends Customer{
	public boolean isNull() {
		return true;
	}
}

public class Customer {
	String name;
	protected Customer() {
	}
	
	public boolean isNull() {
		return false;
	}
}
```

* Customer 클래스를 수정할 수 없으면 null 검사 인터페이스를 사용하면 된다.
* 괜찮다면 널 객체를 사용한다는 사실을 다음과 같이 인터페이스를 통해 나타내자.

```java
interface Nullable {
	boolean isNull();
}

class Customer implements Nullable
```

* NullCustomer 인스턴스를 생성하는 팩토리 메서드를 추가하자.
* 이렇게 하면 클라이언트가 널 클래스 정보를 알 필요가 없다.

```java
public class Customer {
	static Customer newNull() {
		return new NullCustomer();
	}
}
```

* null이 예상될 때마다 새 널 객체를 반환하고 foo==null 형태의 null 검사 코드를 foo.isNull() 형태의 코드로 수정해야 한다.
* Customer 클래스가 사용되는 부분을 모두 찾아서 null 대신 NullCustomer 클래스를 반환하게 수정하자.

```java
public class Site {
	Customer customer;
	
	Customer getCustomer() {
		return (customer == null) ? Customer.newNull() : customer;
	}
}
```

* 이 값을 사용하는 부분도 null 검사 코드 대신 isNull()로 수정하자.

```java
Site site = new Site();
Customer customer = site.getCustomer();
BillingPlan plan;

if(customer.isNull()) {
    plan = BillingPlan.basic();
} else {
    plan = customer.getPlan();
}

String customerName;

if(customer.isNull()) {
    customerName = "occupant";
} else {
    customerName = customer.getName();
}

int weeksDelinquent;

if(customer.isNull()) {
    weeksDelinquent = 0;
} else {
    weeksDelinquent = customer.getHistory().getWeeksDelinquentInLastYear();
}
```

* 수정하는 null의 각 원본 객체마다 null 여부를 검사하는 부분을 전부 찾아서 교체해야 한다.
* Customer 타입의 모든 변수를 찾고 그 변수가 사용되는 곳을 전부 찾아야 한다.
* 널 검사 형태의 == null 코드를 isNull 호출로 수정했지만 아직은 장점이 와닿지 않는다.
* 그 장점은 NullCustomer로 기능을 옮기고 조건문을 삭제해야 느낄 수 있다.
* NullCustomer 클래스에 다음과 같이 적합한 name 읽기 메서드를 추가하자.

```java
String customerName;

if(customer.isNull()) {
    customerName = "occupant";
} else {
    customerName = customer.getName();
}
```

```java
String customerName = customer.getName
    
if(customer.isNull()) {
    customer.setPlan(BillingPlan.special());
}

customer.setPlan(BillingPlan.special());

public class NullCustomer extends Customer{
	public boolean isNull() {
		return true;
	}
	
	public String getName() {
		return "occupant";
	}
	
	public void setPlan(BillingPlan arg) {
	}
}
```



### 8 어설션 넣기

* 일부 코드가 프로그램의 어떤 상태를 전제할 땐 어설션을 넣어서 그 전체를 확실하게 코드로 작성하자.



#### 8.1 동기

* 어설션이란 항상 참으로 참제되는 조건문을 뜻한다.
* 어설션이 실패하면 그건 프로그래머가 오류를 범한 것이다.
* 그래서 어설션이 실패할 경우 반드시 에외를 통지하게 해야 한다.
* 시스템의 다른 부분에는 절대로 어설션을 사용하면 안 된다.
* 어설션은 대개 제품화 단게에서 삭제한다.
* 따라서 코드에서 어설션을 넣은 부분을 꼭 표시해둬야 한다.



#### 8.2 방법

* (이번 절은 일단 딱히 중하지 않아 보여서 패스)