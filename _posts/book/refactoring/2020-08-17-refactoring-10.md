---
title:  "일반화 처리"
excerpt: "일반화 처리"
classes: wide
categories:
  - Refactoring
tags:
  - [Refactoring]
last_modified_at: 2020-08-17
---



# 일반화 처리



### 1. 필드 상향

* 두 하위클래스에 같은 필드가 들어 있을 땐 필드를 상위클래스로 옮기자.



#### 11.1 동기

* 각각의 하위 클래스를 따로 개발 중이거나 리팩토링을 적용해서 연결할 경우, 여러 하위클래스에 중복된 기능이 들어 있는 경우가 많다.
* 특히 특정한 몇 갱의 필드가 중복될 수 있다.
* 중복된 필드는 이름이 서로 비슷할 때도 있다.
* 어느 필드가 무슨 용도로 쓰이는지 알려면, 필드를 살펴보면서 다른 메서드에 어떤 식으로 사용되는지 보면 된다.
* 중복된 필드가 서로 비슷한 방식으로 사용된다면 그 필드를 일반화하면 된다.
* 일반화란 상위클래스로 옮기는 작업을 말한다.



#### 11.2 방법

* 상위클래스로 옮길 필드가 사용된 모든 부분을 검사해서 같은 방식으로 사용되는지 확인하자.
* 필드들의 이름이 같지 않다면 필드명을 상위클래스 필드로 사용할 이름으로 변경하자.
* 상위클래스 안에 새 필드를 작성하자.
  * 필드가 private이면 상위클래스 필드를 protected로 수정해서 하위클래스가 참조할 수 있게 하자.
* 하위 클래스의 필드는 삭제하자.
* 새 필드에 필드 자체 캡슐화 적용을 고려하자.



### 2. 메서드 상향

* 기능이 같은 메서드가 여러 하위클래스에 들어 있을 땐 그 메서드를 상위클래스로 옮기자.



#### 2.1 동기

* 메서드의 내용이 서로 같을 때 메서드 상향을 적용한다.
* 메서드 상향은 다른 리팩토링 단계를 마친 후 적용하는 것이 일반적이다.
* 다른 클래스에 들어 있는 두 개의 메서드가 매개변수로 전환되어 결국 같은 메서드가 될 수도 있다.

* 특수한 상황으로는 하위클래스 메서드가 상위클래스 메서드를 재정의함에도 불구하고 기능이 같을 때이다.
* 메서드 상향을 적용하기 난감할 때는 하위 클래스에는 있고 상위클래스에는 없는 기능을 참조할 때이다.
* 그 기능이 아마 다른 메서드를 상위클래스로 옮기는 것이 가능할 것이다.
* 이를 위해서는 메서드의 시그너처를 수정하거나 위임하는 메서드를 작성해야 할 수도 있다.
* 두 메서드가 똑같지 않고 비슷한 부분이 있다면 템플릿 메서드 형성을 실시하는 방법도 있다.



#### 2.2 방법

* 메서드가 서로 같은지 검사하자.
  * 거의 비슷한데 똑같진 않다면 한 메서드에 알고리즘 전환을 적용해서 메서드를 똑같게 만들자.
* 메서드의 시그너처가 서로 다르다면 모든 시그너처를 상위클래스에 사용하고자 하는 시그너처로 수정하자.
* 상위클래스 안에 새 메서드를 작성하고 새 메서드 안에 같은 메서드의 내용을 복사한 후 적절히 수정하고 컴파일하자.
  * 철저한 타입 선언을 요하는 언오로 작업 중일 때 그 메서드가 두 하위클래스엔 있고 상위클래스엔 없는 다른 메서드를 호출한다면 상위클래스에 abstract 타입의 메서드를 선언하자.
  * 메서드가 하위클래스의 필드를 사용한다면 필드 상향이나 필드 자체 캡슐화를 적용하고 abstract타입의 읽기 메서드를 선언해 사용하자.
* 하위 클래스의 메서드를 하나 삭제하자.
* 상위클래스의 메서드만 남을 때까지 하위클래스 메서드를 계속 삭제하자.
* 상위클래스 메서드를 호출하는 부분을 보변서 필요한 타입을 상위클래스로 수정할 수 있는지 파악하자.



#### 2.3 예제

![]({{site.url}}/assets/images/ref21.PNG)

* createBill 메서드는 다음과 같다.

```java
void createBill (Date date) {
    double chargeAmout = chargeFor (lastBillDate, date);
    addBill(date, charge);
}
```

* 앞의 메서드는 상위클래스로 올려보낼 수 없다.
* 왜냐하면 chargeFor 메서드가 하위클래스마다 다르기 때문이다.
* 우선 상위클래스에서 chargeFor 메서드를 abstract로 선언해야 한다.

```java
class Customer {
	abstract double chargeFor(Date start, Date end);
}
```

* 그 다음 하위클래스 중 하나에서 createBill 메서드를 복사하고 컴파일한다.
* 그리고 하위클래스 중 하나에서 createBill 메서드를 삭제하고 테스트한다.
* 그 다음, 나머지 하위클래스에서 createBill 메서드를 삭제하고 테스트한다.

* 그러면 아래와 같이 된다.

![]({{site.url}}/assets/images/ref22.PNG)



### 3. 생성자 내용 상향

* 하위클래스마다 거의 비슷한 내용의 생성자가 있을 땐 상퀴클래스에 생성자를 작성하고, 그 생성자를 하위클래스의 메서드에서 호출하자.



#### 3.1 동기

* 생성자는 일반 메서드와 전혀 달라서 일반 메서드에 비해 할 수 있는 것이 제한적이다.
* 개발자는 여러 하위클래스에 같은 기능의 메서드가 든 것을 발견하면 우선적으로 공통적인 부분을 메서드로 빼낸 후 상위클래스로 옮길 생각을 하게 된다.
* 그러나 하위클래스는 대체로 공통적인 생성 기능이다.
* 이럴 땐 하위클래스에서 생성자 메서드를 작성하고 상위클래스로 올려서 하위클래스들이 호출하게 해야 한다.
* 대부분 공통적인 기능은 생성자 내용 전체다.
* 생성자는 상속이 불가능하기 때문에 메서드 상향을 적용할 수 없다.
* 리팩토링이 복잡해지면 이 기법 대신 생성자를 팩토리 메서드로 전환을 적용해야 할 수 도 있다.



#### 3.2 방법

* 상위클래스에 생성자를 정의하자.
* 하위클래스 생성자에서 앞 부분의 공통적인 코드를 상위클래스 생성자 안으로 옮기자.
  * 어쩌면 하위클래스 생성자의 코드 전체를 옮겨야 할지도 모른다.
  * 공통 코드를 생성자 앞쪽에 넣자.
* 하위클래스 생성자 안의 맨 앞에 상위클래스 생성자 호출 코드를 넣자.
  * 코드 전체가 공통적이라면 하위클래스 생성자 안엔 바로 이 상위클래스 생성자 호출 코드만 남게 된다.
* 컴파일과 테스트를 실시하자.
  * 나중에 공통적인 코드가 발견되면 메서드 추출을 적용해서 공통적인 코드를 별도의 메서드로 만들고 메서드 상향을 적용해서 상위클래스로 올리자.



#### 3.3 예제

```java
public class Employee {
	protected String name;
	protected String id;
}

public class Manager extends Employee {
	private int grade;
	
	public Manager (String name, String id, int grade) {
		this.name = name;
		this.id = id;
		this.grade = grade;
	}
}
```

* Employee 클래스의 두 필드는 Employee 클래스의 생성자 안에서 값 지정이 이뤄져야 한다.
* 따라서 다음과 같이 생성자 메서드를 정의하고 하위클래스가 호출해야 함을 명시하고자 protected 타입으로 선언하자.

* 그리고 Emlpoyee 생성자 메서드를 하위클래스에서 호출하게 하자.

```java
public class Employee {
	protected String name;
	protected String id;
	
	public Employee (String name, String id) {
		this.name = name;
		this.id = id;
	}
}

public class Manager extends Employee {
	private int grade;
	
	public Manager (String name, String id, int grade) {
		super(name, id);
		this.grade = grade;
	}
}
```

* 나중에 공통적인 코드가 발견될 땐 조금 달라진다.

```java
public class Employee {	
    protected String name;
	protected String id;
	
	public Employee (String name, String id) {
		this.name = name;
		this.id = id;
	}
    
	boolean isPriviliged() {
	}
	void assignCar() {
	}
}

public class Manager extends Employee {
	private int grade;
	
	public Manager (String name, String id, int grade) {
		super(name, id);
		this.grade = grade;
		
		if(isPriviliged()) { // 모든 하위클래스의 공통 기능
			assignCar();
		}
	}
	
	boolean isPriviliged() {
		return grade > 4;
	}
}
```

* assignCar 메서드의 기능은 상위클래스 생성자 안으로 옮길 수 없다.
* 왜냐하면 grade가 grade 필드에 대입된 후 실행되어야 하기 때문이다.
* 따라서 메서드 추출과 메서드 상향을 적용해야 한다.

```java
public class Employee {
	protected String name;
	protected String id;
	
	public Employee (String name, String id) {
		this.name = name;
		this.id = id;
	}
	
	boolean isPriviliged() {
	}
	void assignCar() {
	}
	
	void initialize() {
		if(isPriviliged()) {
			assignCar();
		}
	}
}

public class Manager extends Employee {
	private int grade;
	
	public Manager (String name, String id, int grade) {
		super(name, id);
		this.grade = grade;
		initialize();
	}
}
```



### 4. 메서드 하향

* 상위 클래스에 있는 기능을 일부 하위클래스만 사용할 땐 그 기능을 관련된 하위클래스 안으로 옮기자.



#### 4.1 동기

* 메서드 하향은 메서드 상향과 반대되는 기법이다.
* 이 기법은 상위클래스의 기능을 특정 하위클래스로 옮겨야 할 때 사용한다.
* 이 기법은 하위클래스 추출을 실시할 때 흔히 사용한다.



#### 4.2 방법

* 모든 하위클래스에 메서드를 하나 선언하고 그 메서드의 내용을 각 하위클래스로 복사하자.
  * 메서드가 필드에 접근할 수 있으려면 필드를 protected로 선언해야 한다.
  * 그 필드를 나중에 하위클래스로 내릴 예정일 때 주로 이렇게 한다.
  * 다른 방법으로 상위클래스에 읽기 메서드를 사용하자.
  * 이 읽기 메서드를 public으로 선언할 필요가 없다면 protected로 선언해야 한다.
* 상위클래스의 메서드를 삭제하자.
  * 변수와 매개변수 선언에 하위클래스를 사용하게 호출 부분을 수정해야 할 수도 있다.
  * 상위클래스 변수를 통해 그 메서드에 접근하는 것이 적절하고, 어느 하위클래스에서든 그 메서드를 삭제할 생각이 없고, 상위클래스가 abstract로 선언되어 있다면, 그 메서드를 상위클래스 안에 abstarct로 선언할 수 있다.
* 그 메서드가 필요 없어진 하위클래스에선 그 메서드를 삭제하자.



### 5. 필드 하향

* 일부 하위클래스만이 사용하는 필드가 있을 땐 그 필드를 사용하는 하위클래스로 옮기자.



#### 5.1 동기

* 필드 하향은 필드 상향과 정반대이다.
* 이 기법은 필드가 상위클래스엔 필요 없고 하위클래스에만 필요할 때 사용하자.



#### 5.2 방법

* 모든 하위클래스 안에 그 필드를 선언하자.
* 상위클래스에서 그 필드를 삭제하자.
* 그 필드를 사용하지 않는 모든 하위클래스에서 삭제하자.



### 6. 하위클래스 추출

* 일부 인스턴스에만 사용되는 기능이 든 클래스가 있을 땐 그 기능 부분을 전담하는 하위클래스를 작성하자.



#### 6.1 동기

* 하위클래스 추출은 주로 클래스의 기능을 그 클래스의 일부 인스턴스만 사용할 때 적용한다.
* 간혹 이런 신호는 분류 부호에 나타기도 하는데, 이럴 때는 분류 부호를 하위클래스로 전환이나 분류 부호를 상태/전략 패턴으로 전환 중 하나를 실시하면 된다.
* 하지만 하위클래스에 사용하게 하려고 분류 부호를 넣을 필요는 없다.
* 하위클래스 추출 대신 클래스 추출을 사용할 수 있다.
* 이 두 기법은 위임이냐 상속이냐의 차이다.
* 하위클래스 추출 기법이 보통은 더 간단하지만 단점이 있다.
* 첫째, 객체가 생성된 후에는 객체의 클래스 기반 기능을 수정할 수 없다.
* 클래스 기반 기능을 수정하려면 단순히 다른 각종 컴포넌트를 연결해서 클래스 추출을 실시하면 된다.
* 둘째, 하위클래스를 사용해서 한가지 변형만을 표현할 수도 있다.
* 클래스가 다양하게 변하게 하려면 하나가 아닌 모두를 대상으로 위임을 적용해야 한다.


#### 6.2 방법

* 원본 클래스에 새 하위클래스를 정의하자.
* 그 하위클래스에 생성자 메서드를 작성하자.
  * 간단할 땐 상위클래스의 인자를 복사하고 super를 사용해서 상위클래스의 생성자를 호출하자.
  * 하위클래스를 사용한다는 사실을 클라이언트가 모르게 은닉하려면 생성자를 팩토리 메서드로 전환을 적용하자.
* 상위클래스의 생성자를 호출하는 부분을 전부 찾자. 그 분이 하위클래스를 사용한다면 새로 작성한 생성자 호출로 고치자.
  * 하위클래스의 생성자가 다른 인자를 받아야 할 땐 메서드명 변경을 적용해서 생성자명을 바꾸자.
  * 상위클래스의 생성자 매개변수 중 일부가 더 이상 필요 없어졌을 때도 마찬가지로 메서드명 변경을 적용하자.
  * 상위클래스를 직접 인스턴스화될 수 없게 됐을 땐 상위클래스를 abstract타입으로 바꾸자.
* 메서드 하향과 필드 하향을 차례로 적용해서 기능을 하위클래스로 옮기자.
  * 클래스 추출과 달리 보통은 메서드에 먼저 적용하고 데이터엔 나중에 적용하는 것이 더 간편하자.
  * public 메서드를 하위클래스로 옮길 때, 호출 부분의 변수나 매개변수 타입을 새로 정의해서 새 메서드를 호출해야 할 수도 있다.
  * 이렇게 할 경우는 컴파일러가 잡아낸다.
* 계층구조가 현재 나타내는 정보(보통 부울값이나 분류 부호)가 저장되는 필드가 있는지 찾자. 
* 필드 자체 캡슐화를 실시해서 그 필드를 제거하고 속성 읽기 메섣를 다형적인 상수 메서드로 교체하자. 
* 이 필드를 사용하는 모든 부분을 대상으로 조건문을 재정의로 전환을 실시해야 한다.
  * 읽기 메서드를 사용하는 클래스 밖의 모든 메서드에 메서드 이동을 적용해서 그 클래스에 옮길지 고려하자.
  * 그 다음, 조건문을 재정의로 전환을 실시하자.
* 각 메서드를 하위클래스로 내릴 때마다 컴파일과 테스트를 실시하자.



#### 6.3 예제

* 동네 정비소에서 작업 종류별 가격을 구하는 JobItem 클래스는 다음과 같다.

```java
public class Employee {
	private int rate;
	
	public Employee (int rate) {
		this.rate = rate;
	}
	
	public int getRate() {
		return rate;
	}
}

public class JobItem {
	private int unitPrice;
	private int quantity;
	private Employee employee;
	private boolean isLabor;
	
	public JobItem (int unitPrice, int quantity, boolean isLabor, Employee employee) {
		this.unitPrice = unitPrice;
		this.quantity = quantity;
		this.isLabor = isLabor;
		this.employee = employee;
	}
	
	public int getTotalPrice() {
		return getUnitPrice() * quantity;
	}

	public int getUnitPrice() {
		return (isLabor) ? employee.getRate() : unitPrice;
	}
	
	public int getQuantity() {
		return quantity;
	}
	
	public Employee getEmployee() {
		return employee;
	}
}
```

* 앞의 클래스에서 LaborItem 하위클래스를 추출하자.
* 그래야 하는 이유는 기능과 데이터 중 일부가 LaborItem 클래스에만 사용되기 때문이다.
* 새 클래스를 작성한다.

```java
class LaborItem extends JobItem {}
```

* JobItem 클래스엔 인자를 받지 않는 생성자가 없으니 우선 LaborItem 클래스에 생성자부터 작성해야 한다.
* LaborItem의 생성자를 작성하고자 JobItem 생성자의 시그너처를 다음과 같이 복사하자.

```java
public class LaborItem extends JobItem{
	public LaborItem (int unitPrice, int quantity, boolean isLabor, Employee employee) {
		super(unitPrice, quantity, isLabor, employee);
	}
}
```

* 이렇게만 해도 하위클래스는 충분히 성공적으로 컴파일된다.
* 하지만 생성자가 지저분해진다.
* LaborItem에 어떤 인자들은 필요하고 어떤 인자들은 필요하지 않다. 이 문제는 나중에 처리하자.
* 그 다음, JobItem의 생성자를 호출하는 부분을 찾은 후 LaborItem의 생성자가 대신 호출되어야 할 경우들을 찾아야 한다.

```java
JobItem j1 = new JobItem(0, 5, true, kent);
```

* 앞의 코드를 LaborItem의 생성자를 호출하게 다음과 같이 수정하자.

```java
JobItem j1 = new JobItem(0, 5, true, kent);
```

* 이 단계에서 변수의 타입은 수정하지 않고 생성자의 타입만 수정했다.
* 새로운 타입은 꼭 필요한 곳에만 사용해야 하기 때문이다.
* 아직 하위클래스에 특정 인터페이스가 없으니, 아직 다른 변형 하위클래스를 선언하면 안 된다.
* 이제 생성자 매개변수 세트를 정리해야 한다.
* 각 메서드에 메서드명 변경을 적용하자.
* 우선 상위클래스부터 작업하자.
* 새 생성자를 작성하고 기존의 생성자는 protected 타입으로 바꾸자.
* 아직 하위클래스가 기존 생성자를 사용하기 때문이다.

```java
public class JobItem {
	private int unitPrice;
	private int quantity;
	private Employee employee;
	private boolean isLabor;
	
	protected JobItem (int unitPrice, int quantity, boolean isLabor, Employee employee) {
		this.unitPrice = unitPrice;
		this.quantity = quantity;
		this.isLabor = isLabor;
		this.employee = employee;
	}
	
	public JobItem (int unitPrice, int quantity) {
		this (unitPrice, quantity, false, null);
	}
	
	public int getTotalPrice() {
		return getUnitPrice() * quantity;
	}

	public int getUnitPrice() {
		return (isLabor) ? employee.getRate() : unitPrice;
	}
	
	public int getQuantity() {
		return quantity;
	}
	
	public Employee getEmployee() {
		return employee;
	}
}
```

* 외부에 있는 호출 코드엔 이제 다음과 같이 새 생성자를 사용해야 한다.

```java
JobItem j1 = new JobItem(10, 15);
```

* 컴파일과 테스트를 실시한 후 하위클래스 생성자를 대상으로 다음과 같이 메서드명 변경을 실시하자.

```java
public class LaborItem extends JobItem{
	public LaborItem (int quantity,Employee employee) {
		super(0, quantity, true, employee);
	}
}
```

* 당분간은 protected 타입의 상위클래스 생성자를 사용하자.
* 이제 JobItem의 기능들을 하위클래스로 내릴 수 있다. 메서드부터 옮기자.
* 우선 읽기 메서드 getEmployee에 메서드 하향을 다음과 같이 적용하자.

```java
public class JobItem {
	protected Employee employee;
}

public class LaborItem extends JobItem{
	public Employee getEmployee() {
		return employee;
	}
}
```

* employee 필드는 나중에 하위클래스로 내릴 예정이므로, 일단 protected 타입으로 선언하자.
* employee 필드를 protected 타입으로 고쳤으면 employee 필드가 내려간 하위클래스에서만 값이 설정되게 생성자를 모두 정리하자.

```java
public class JobItem {
	private int unitPrice;
	private int quantity;
	protected Employee employee;
	private boolean isLabor;
	
	protected JobItem (int unitPrice, int quantity, boolean isLabor) {
		this.unitPrice = unitPrice;
		this.quantity = quantity;
		this.isLabor = isLabor;
	}
	
	public JobItem (int unitPrice, int quantity) {
		this (unitPrice, quantity, false);
	}
}	

public class LaborItem extends JobItem{
	public LaborItem (int quantity,Employee employee) {
		super(0, quantity, true);
		this.employee = employee;
	}
	
	public Employee getEmployee() {
		return employee;
	}
}
```

* isLabor 필드는 상위클래스에서 상속되는 정보를 참조하는데 쓰인다
* 따라서 이 필드는 삭제해도 된다.
* 이를 위한 최선의 방법은 필드 자체 캡슐화를 실시한 후 재정의 상수 메서드를 이용하게끔 접근 메서드를 수정하는 것이다.
* 재정의 상수 메서드는 각 구현부가 서로 다른 고정 값을 반환하는 메서드다.

```java
public class JobItem {
	protected boolean isLabor() {
		return false;
	}
}

public class LaborItem extends JobItem{
	protected boolean isLabor() {
		return true;
	}
}
```

* 이제 isLabor 필드를 없애자.

* isLabor 메서드를 호출하는 부분을 조건문을 재정의로 전환을 적용하자.

```java
public int getUnitPrice() {
    return (isLabor()) ? employee.getRate() : unitPrice;
}
```

* 그럼 앞의 메서드가 다음과 같이 수정된다.

```java
public int getUnitPrice() {
	return unitPrice;
}
```

* 일부 데이터를 사용하는 메서드들을 하위클래스로 내린 후 데이터에 필드 하향을 적용하자.
* 어떤 메서드가 그 데이터를 사용해서 개발자가 그 메서드를 사용할 수 없다면, 추가로 메서드 하향이나 조건문을 재정의로 전환을 실시해야 한다.
* unitPrice 필드는 근로와 관련 없는 항목들(JobItem)에만 사용되므로, JobItem을 대상으로 하위클래스 추출을 다시 실시해서 PartItem클래스를 만들면 된다.
* 작업을 완료하면 JobItem 클래스는 abstract 타입이 된다.



### 7. 상위클래스 추출

* 기능이 비슷한 두 클래스가 있을 땐 상위클래스를 작성하고 공통된 기능들을 그 상위클래스로 옮기자.



#### 7.1 동기

* 비슷한 작업을 같은 방식이나 다른 방식으로 수행하는 두 클래스가 있다면 상위클래스를 추출하자.
* 상위클래스 추출을 적용할 수 없을 땐 클래스 추출을 적용하면 된다.
* 둘은 기본적으로 상속이냐 위임이냐의 차이다.
* 상위클래스 추출을 적용한 게 잘못된 선택이었다면, 나중에 상속을 위임으로 전환을 실시하면 된다.



#### 7.2 방법

* 빈 abstract 타입의 상위클래스를 작성하자. 원본 클래스들은 이 상위클래스의 하위클래스로 만들자.
* 필드 상향, 메서드 상향, 생성자 내용 상향을 차례로 적용해서 공통된 요소를 상위 클래스로 옮기자.
  * 필드부터 옮기는 것이 대체로 더 간편하자.
  * 하위클래스의 여러 메서드가 시그너처는 다르지만 기능이 같다면 메서드명 변경을 적용해서 그 메서들의 이름을 같게 만든 후 메서드 상향을 적용하자.
  * 하위클래스의 여러 메서드가 시그너처는 같지만 내용이 다르다면, 공통된 시그너처를 상위클래스에 abstract 타입의 메서드로 선언하자
  * 하위클래스의 여러 메서드가 내용이 다르지만 기능이 같다면 알고리즘 전환을 적용해서 한 메서드의 내용을 다른 메서드로 복사하자. 문제없이 마쳤으면 메서드 상향을 적용하자.
* 메서드를 상위클래스로 옮길 때마다 테스트한다.
* 하위클래스에 남아 있는 메서드마다 검사하면서 공통된 부분이 있는지 확인하고, 만약 있으면 그 공통 부분에 메서드 추출과 메서드 상향을 차례로 적용하면 된다. 전체적인 흐름이 비슷하다면 템플릿 메서드 형성을 실시하는 것도 괜찮다.
* 모든 공통 요소를 상위클래스로 옮긴 후 하위클래스를 사용하는 각 부분을 검사하자. 하위클래스 사용 부분이 공통 인터페이스만 사용한다면 필요한 그 인터페이스의 타입을 상위클래스의 타입으로 바꾸자.



#### 7.3 예제

```java
public class Employee {
	private String name;
	private int annualCost;
	private String id;
	
	public Employee (String name, String id, int annualCost) {
		this.name = name;
		this.id = id;
		this.annualCost = annualCost;
	}

	public String getName() {
		return name;
	}

	public int getAnnualCost() {
		return annualCost;
	}

	public String getId() {
		return id;
	}
}

public class Department {
	private String name;
	private Vector<Employee> staff = new Vector<Employee>();
	
	public Department (String name) {
		this.name = name;
	}
	
	public int getTotalAnnualCost() {
		Enumeration<Employee> e = getStaff();
		int result = 0;
		
		while(e.hasMoreElements()) {
			Employee each = e.nextElement();
			result += each.getAnnualCost();
		}
		
		return result;
	}
	
	public int getHeadCount() {
		return staff.size();
	}
	
	public void addStaff(Employee arg) {
		staff.addElement(arg);
	}

	public Enumeration<Employee> getStaff() {
		return staff.elements();
	}
	
	public String getName() {
		return name;
	}
}
```

* 앞의 두 클래스는 공통된 부분이 두 가지다.
* 첫 번째 공통점은 사원과 부서는 둘다 이름이 있다는 점이다.
* 두 번째 공통점은 계산 메서드가 약간 다르긴 하지만 두 클래스 모두 연간 경비가 있다.
* 우선, 다음과 같이 새 상위클래스를 작성하고 기존 상위클래스는 새 상위클래스 안에 하위클래스로 넣자.

```java
abstract class Party {}
class Employee extends Party ...
class Department extends Party ...
```

* 이제 기능을 상위클래스에 올리자.
* 주로 필드 상향부터 적용하는 것이 더 간편하다. 그리고 메서드 상향도 적용하자.

```java
abstract class Party {
	protected String name;
	
	public String getName() {
		return name;
	}
}
```

* name 필드를 private 타입으로 만들어야 한다.
* 이를 위해 생성자 내용 상향을 적용하여 이름을 할당하자.

```java
abstract class Party {
	private String name;
	
	protected Party (String name) {
		this.name = name;
	}
	
	public String getName() {
		return name;
	}
}

public class Employee extends Party{
	public Employee (String name, String id, int annualCost) {
		super(name);
		this.id = id;
		this.annualCost = annualCost;
	}
}

public class Department extends Party{
	public Department (String name) {
		super(name);
	}
}
```

* getTotalAnnualCost, getAnnualCost 메서드는 기능이 같으므로 이름이 같아야 한다.
* 메서드명 변경을 적용해서 두 메서드의 이름을 같게 만들자.
* 두 메서드의 내용이 아직 다르므로 메서드 상향을 적용할 수 없다.
* 그러나 다음과 같이 상위클래스에 abstract 메서드를 선언할 순 있다.

```java
abstract public int getAnnualCost()
```

* 확실한 부분을 모두 수정하고 나서 두 클래스가 사용되는 부분을 보면서 새 상위클래스를 사용하게 수정해도 되는지 검사하자.
* 두 클래스의 사용 부분 중 하나는 Department 클래스 자체인데, 이 클래스엔 사원 컬렉션이 들어 있다.
* getAnnualCost 메서드는 이제 Party 클래스로 옮긴 getAnnualCost 메서드만 사용한다.

* 앞의 기능은 새로운 가능성을 시사한다.
* Department 클래스와 Employee 클래스를 콤퍼지트 패턴으로 취급할 수도 있을 것이다.
* 그러면 Department 클래스 안에 다른 Department 클래스를 넣을 수 있다.
* 이것은 새로운 기능이므로 엄밀히 말하면 리팩토링이 아니다. 콤퍼지트 패턴을 사용하려 했다면 staff 필드의 이름을 더 적절한 것으로 수정했을 것이다.
* 이렇게 수정하려면 addStaff 메서드명도 그에 맞게 수정하고 매개변수를 Party 클래스로 수정해야 한다.
* 끝으로 headCount 메서드를 수정해서 재귀 메서드로 만들어야 한다.
* 그러려면 Employee 클래스에 단순히 1을 반환하는 headCount 메서드를 작성하고, 부서에 속한 사원 수를 합산하는 Department 클래스의 headCount 메서드에 알고리즘 전환을 적용하면 된다.



### 8. 인터페이스 추출

* 클래스 인터페이스의 같은 부분을 여러 클라이언트가 사용하거나, 두 클래스에 인터페이스의 일부분이 공통으로 들어 있을 땐 공통 부분을 인터페이스로 빼내자.



#### 8.1 동기

* 클래스는 서로를 다양한 방식으로 사용한다.
* 클래스를 사용한다는 건 주로 클래스의 담당 기능 전반을 사용함을 뜻한다.
* 하지만 여러 클라이언트가 클래스 기능 중 특정 부분만 사용할 때도 있고, 한 클래스가 각 요청을 처리할 수 있는 특정한 클래스들과 공조해야 할 때도 있다.
* 일부분만 사용하는 경우와 특정 기능의 여러 클래스를 함께 사용하는 경우엔, 클래스 기능 중 사용되는 부분을 분리해서 시스템을 사용할 때 사용되는 부분을 확실히 알 수 있게 하는 것이 좋다.
* 그렇게 하면 해당 기능들이 어떤 식으로 나뉘는지 더욱 쉽게 알 수 있다.
* 새 클래스들이 해당 부분을 보조해야 할 경우엔 해당 부분에 무슨 클래스가 적절할지 정확히 알아내기가 더 쉽다.
* 클래스가 서로 다른 상황에서 서로 다른 역할을 담당할 때 인터페이스를 사용하면 좋다.



#### 8.2 방법

* 빈 인터페이스를 작성하자
* 공통 기능을 인터페이스 안에 선언하자.
* 그 인터페이스를 상속구현하는 관련 클래스들을 선언하자.
* 그 인터페이스를 사용하게 클라이언트의 타입 선언 코드를 수정하자.



#### 8.3 예제

* TimeSheet 클래스는 사원비를 산출한다.
* 이를 위해 TimeSheet 클래스는 사원 평점과 특수 기술 보유 여부를 알아야 한다.

```java
public class TimeSheet {
	double charge(Employee emp, int days) {
		int base = emp.getRate() * days;
		
		if(emp.hasSpecialSkill()) {
			return base * 1.05;
		} else {
			return base;
		}
	}
}
```

* Employee 클래스는 평점과 특수 기술 보여 여부 외에도 많은 데이터가 있지만, 이 프로그램에 필요한 정보는 평점과 특수 기술 보유 여부뿐이다.
* 이 두 가지 정보만이 필요함을 강조하려면 그 두 정보를 알아내는 인터페이스를 다음과 같이 정의하면 된다.

```java
public interface Billable {
	public int getRate();
	public boolean hasSpecialSkill();
}

public class Employee implements Billable{}
```

* 그 다음 charge부분을 수정하자.

```java
public class TimeSheet {
	double charge(Billable emp, int days) {
		int base = emp.getRate() * days;
		
		if(emp.hasSpecialSkill()) {
			return base * 1.05;
		} else {
			return base;
		}
	}
}
```



### 9. 계층 병합

* 상위클래스와 하위클래스가 거의 다르지 않을 땐 둘을 합치자.



#### 9.1 동기

* 상속을 과용하면 클래스 관계가 복잡해진다.
* 이때 하위 클래스가 쓸모가 없어지면 클래스를 합친다.



#### 9.2 방법

* 상위클래스와 하위클래스 중 무엇을 삭제할지 선택하자.
* 필드, 메서드 상향 or 하향을 적용한다.
* 참조 부분도 수정하자.
* 빈 클래스 삭제하자.



### 10. 템플릿 메서드 형성

* 하위클래스 안의 두 메서드가 거의 비슷한 단계들을 같은 순서로 수행할 땐 그 단계들을 시그너처가 같은 두 개의 메서드로 만들어서 두 원본 메서드를 같게 만든 후, 두 메서드를 상위클래스로 옮기자.



#### 10.1 동기

* 상속은 중복된 기능을 없애는 강력한 수단이다.
* 하위클래스에 들어 있는 두 메서드가 비슷하다면 둘을 합쳐서 하나의 상위클래스로 만드는 것이 좋다.
* 그러나 두 메서드가 완전히 똑같지 않다면 어떻게 해야 할까?
* 그래도 중복된 부분은 가능한 정부 없애고 차이가 있는 필수 부분만 그대로 둬야 한다.
* 두 메서드가 똑같지는 않지만 거의 비슷한 단계를 같은 순서로 수행하는 경우가 제일 흔하다.
* 이럴 땐 그 순서를 상위클래스로 옮기고 재정의를 통해 각 단계가 고유의 작업을 다른 방식으로 수행하게 하면 된다.
* 이런 메서드를 템플릿 메서드라고 한다.



#### 10.2 예제

* 1장에서 살펴보다 만 예제를 이어 설명하겠다.

```java
public String statement() {
    Enumeration<Rental> rentals = _rentals.elements();
    String result = getName() + " 고객님의 대여 기록\n";

    while (rentals.hasMoreElements()) {
        Rental each = rentals.nextElement();
        result += "\t" + each.getMovie().getTitle() 
            + "\t" + String.valueOf(each.getCharge()) + "\n";
    }

    result += "누적 대여료: " + String.valueOf(getTotalCharge()) + "\n";
    result += "적립 포인트: " + String.valueOf(getTotalFrequentRenterPoints());

    return result;
}
```

* 한편 htmlStatement 메서드는  다음과 같이 고객의 대여료 내역을 HTML로 출력한다.

```java
public String htmlStatement() {
    String result = "<H1><EM>" + getName() + " 고객님의 대여 기록</EM></H1><P>\n";
    Enumeration<Rental> rentals = _rentals.elements();

    while (rentals.hasMoreElements()) {
        Rental each = rentals.nextElement();
        result += each.getMovie().getTitle() 
            + ": " + String.valueOf(each.getCharge()) + "<BR>\n";
    }

    result += "<P>누적 대여료: <EM>" 
        + String.valueOf(getTotalCharge()) + "</EM><P>\n";
    result += "적립 포인트: <EM>" 
        + String.valueOf(getTotalFrequentRenterPoints()) + "</EM><P>";

    return result;
}
```

* 템플릿 메서드 형성을 실시하기 전에 두 메서드가 어떤 공통 상위클래스의 하위클래스가 되게 정리해야 한다.
* 이렇게 하려면 그림 11-3과 같이 메서드 객체를 이용하여 내역을 출력하는 별도의 전략 패턴 계층을 작성해야 한다.

```java
class statement{}
class TextStatement extends Statement{}
class HtmlStatement extends Statement{}
```

* 이제 다음과 같이 메서드 이동을 적용해서 두 개의 statement 메서드를 하위클래스로 옮기자.

```java
public class Customer {
	public String statement() {
		return new TextStatement().value(this);
	}
	
	public String htmlStatement() {
		return new HtmlStatement().value(this);
	}
}
```

```java
public class TextStatement {
	public String value(Customer aCustomer) {
		Enumeration<Rental> rentals = _rentals.elements();
		String result = getName() + " 고객님의 대여 기록\n";
		
		while (rentals.hasMoreElements()) {
			Rental each = rentals.nextElement();
			result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(each.getCharge()) + "\n";
		}
		
		result += "누적 대여료: " + String.valueOf(getTotalCharge()) + "\n";
		result += "적립 포인트: " + String.valueOf(getTotalFrequentRenterPoints());
		
		return result;
	}
}

public class HtmlStatement {
	public String value(Customer aCustomer) {
		String result = "<H1><EM>" + getName() + " 고객님의 대여 기록</EM></H1><P>\n";
		Enumeration<Rental> rentals = _rentals.elements();
		
		while (rentals.hasMoreElements()) {
			Rental each = rentals.nextElement();
			result += each.getMovie().getTitle() + ": " + String.valueOf(each.getCharge()) + "<BR>\n";
		}
		
		result += "<P>누적 대여료: <EM>" + String.valueOf(getTotalCharge()) + "</EM><P>\n";
		result += "적립 포인트: <EM>" + String.valueOf(getTotalFrequentRenterPoints()) + "</EM><P>";
		
		return result;
	}
}
```

* 두 statement 메서드를 하위클래스로 옮기면서 그 메서드들의 이름을 더욱 전략에 알맞게 변경했다.
*  두 메서드명을 같게 한 이유는 둘의 차이가 메서드가 아니라 클래스에 있기 때문이다.
* 예제 코드를 사용해 이것을 직접 해볼 수 있게 Customer 클래스에 getRentals 메서드를 추가하고 getTotalCharge와 getTotalFrequentRenterPoints의 개방도를 높였다.
* 비슷한 두 메서드를 하위클래스에 넣으면 템플릿 메서드 형성을 적용할 수 있다.
* 이 기법의 핵심은 메서드 추출로 두 메서드의 다른 부분을 추출해서 다른 코드를 비슷한 코드와 분리하는 것이다.
* 추출할 때마다 내용은 다르고 시그너처는 같은 메서드를 작성하자.
* 첫 예제는 헤더 출력이다. 두 메서드 모두 Customer 클래스를 이용해서 정보를 얻지만, 결과 문자열은 다른 형식으로 출력된다.
* 이 문자열을 형식화하는 부분을 시그너처가 같은 별도의 메서드로 빼낼 수 있다.

```java
public class TextStatement {
	public String value(Customer aCustomer) {
		Enumeration<Rental> rentals = aCustomer.getRentals();
		String result = headerString(aCustomer);
		
		while (rentals.hasMoreElements()) {
			Rental each = rentals.nextElement();
			result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(each.getCharge()) + "\n";
		}
		
		result += "누적 대여료: " + String.valueOf(aCustomer.getTotalCharge()) + "\n";
		result += "적립 포인트: " + String.valueOf(aCustomer.getTotalFrequentRenterPoints());
		
		return result;
	}

	private String headerString(Customer aCustomer){
		return aCustomer.getName() + " 고객님의 대여 기록\n";
	}
}
```

* 그리고 각 공통 부분도 다 진행한다

```java
public class TextStatement {
	public String value(Customer aCustomer) {
		Enumeration<Rental> rentals = aCustomer.getRentals();
		String result = headerString(aCustomer);
		
		while (rentals.hasMoreElements()) {
			Rental each = rentals.nextElement();
			result += eachRentalString(each);
		}
		
		result = footerString(aCustomer, result);
		
		return result;
	}

	private String footerString(Customer aCustomer, String result) {
		result += "누적 대여료: " + String.valueOf(aCustomer.getTotalCharge()) + "\n";
		result += "적립 포인트: " + String.valueOf(aCustomer.getTotalFrequentRenterPoints());
		return result;
	}

	private String eachRentalString(Rental each) {
		return "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(each.getCharge()) + "\n";
	}

	private String headerString(Customer aCustomer){
		return aCustomer.getName() + " 고객님의 대여 기록\n";
	}
}
```

* HtmlStatment도 같은 형태로 된다. (TextStatmenet과 같아서 생략함)
* 그럼 value 메서드가 비슷하므로 메서드 상향을 적용한다.
* 메서드 올릴 때 하위 클래스의 메서드를 abstract로 선언해야 한다.

```java
public abstract class Statement {
	
	public String value(Customer aCustomer) {
		Enumeration<Rental> rentals = aCustomer.getRentals();
		String result = headerString(aCustomer);
		
		while (rentals.hasMoreElements()) {
			Rental each = rentals.nextElement();
			result += eachRentalString(each);
		}
		
		result = footerString(aCustomer, result);
		
		return result;
	}

	abstract String headerString(Customer aCustomer);
	abstract String eachRentalString(Rental each);
	abstract String footerString(Customer aCustomer, String result);
	
}

public class TextStatement extends Statement{

	String footerString(Customer aCustomer, String result) {
		result += "누적 대여료: " + String.valueOf(aCustomer.getTotalCharge()) + "\n";
		result += "적립 포인트: " + String.valueOf(aCustomer.getTotalFrequentRenterPoints());
		return result;
	}

	String eachRentalString(Rental each) {
		return "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(each.getCharge()) + "\n";
	}

	String headerString(Customer aCustomer){
		return aCustomer.getName() + " 고객님의 대여 기록\n";
	}
}
```

* HtmlStatment 클래스도 똑같이 한다.



### 11. 상속을 위임으로 전환

* 하위클래스가 상위클래스 인터페이스의 일부만 사용할 때나 데이터를 상속받지 않게 해야 할 땐 상위클래스에 필드를 작성하고, 모든 메서드가 그 상위클래스에 위임하게 수정한 후 하위클래스를 없애자.



#### 11.1 동기

* 상속은 훌륭한 기능이지만 간혹 적합하지 않을 때도 있다.
* 주로 한 클래스에서부터 상속을 시작했다가, 나중에 가서 대부분의 상위클래스 작업이 하위클래스엔 별로 적절하지 않음을 깨닫는다.
* 이것은 클래스의 기능이 인터페이스에 제대로 반영되지 않았거나, 하위클래스로 적절하지 않은 많은 데이터를 상속하게 작성했거나, 상위클래서의 protected 메서드가 하위클래스에 사용되지 않기 때문일 수 있다.
* 그 상황을 방치한 채 관례에 따라 '하위클래스이긴 해도 상위클래스 기능의 일부만 사용한다'고 말할 수도 있다.
* 하지만 그러면 코드는 의도와 달리 복잡해지고 혼동하기 쉬워져서 이를 해결해야 하는 문제가 추가된다.
* 상속 대신 위임을 이용하면 위임받은 클래스의 일부만 사용하려는 의도가 더욱 확실해진다.
* 인터페이스의 어느 부분을 사용하고 어느 부분을 무시할지를 개발자가 제어할 수 있다.
* 단지 위임하는 메서드를 추가로 작성하면 되는데, 이 작업은 지루하지만 아주 간단해서 오류가 생길 염려는 없다.



#### 11.2 방법

* 하위클래스 안에 상위클래스의 인스턴스를 참조하는 필드를 작성하자. 그 필드를 this로 초기화하자.
* 하위클래스 안의 각 메서드를 수정해서 대리 필드를 사용하게 하자. 각 메서드를 수정할 때마다 테스트하자.
  * super의 메서드를 호출하는 하위클래스의 모든 메서드는 수정할 수 없거나, 수정할 수 있더라도 무한 루프에 빠질 가능성이 있다. 이런 메서드는 수정하려면 상속 구조를 깨야 한다
* 하위클래스 선언을 삭제하고 대리 객체 대입 부분을 새 객체 대입으로 바꾸자.
* 클라이언트가 사용하는 상위클래스 메서드마다 간단한 위임 메서드를 추가하자.



#### 11.3 예제

* 부적절한 상속의 전형적인 예는 스택을 백터의 하위클래스로 만드는 것이다.

```java
public class MyStack extends Vector{
	public void push(Object element) {
		insertElementAt(element, 0);
	}
	
	public Object pop() {
		Object result = firstElement();
		removeElementAt(0);
		return result;
	}
}
```

* 앞의 클래스를 사용하는 부분을 보다가, 나는 클라이언트가 스택으로 네 개의 메서드 push, pop, size, isEmpty만 호출함을 발견했다. size와 isEmpty 메서드는 Vector 클래스에서 상속된다.
* 위임하게 만들려면 우선 작업을 위임받을 Vector 클래스에 필드를 작성하자. 이 필드를 this에 연결하면 이 리팩토링을 실시하는 동안 위임과 상속을 혼용할 수 있다.

```java
public class MyStack{
	private Vector vector;
	
	public void push(Object element) {
		vector.insertElementAt(element, 0);
	}
	
	public Object pop() {
		Object result = vector.firstElement();
		vector.removeElementAt(0);
		return result;
	}
    
    public int size() {
        vetor.size();
    }
    
    // isEmpty() 함수 생략
}
```



### 12. 위임을 상속으로 전환

* 위임을 이용 중인데 인터페이스 전반에 간단한 위임으로 도배하게 될 땐 위임 클래스를 대리 객체의 하위클래스로 만들자.



#### 12.1 동기

* 이 기법은 상속을 위임으로 전환 기법의 반대이다.
* 대리 객체의 모든 메서드를 사용하게 되고 그런 간단한 위임 메서드를 지나치게 자주 작성하게 될때는 계층구조로 꽤 손쉽게 바꿀 수 있다.
* 두 가지 주의사항을 염두에 두자.
* 위임하려난 클래스의 모든 메서드를 사용하는 게 아닐 경우엔 위임을 상속으로 전환을 적용해서는 안된다.
* 왜냐하면 하위클래스를 반드시 상위클래스의 인터페이스를 따라야 하기 때문이다.
* 위임메서드가 거치적거리면 과잉 중개 메서드 제거를 적용해서 클라이언트가 대리를 직접 호출하게 하는 것이다.
* 하위클래스 추출을 적용해서 공통 인터페이스를 분리한 후 새 클래스에서 상속받으면 된다.
* 비슷한 방법으로 인터페이스 추출을 적용해도 된다.
* 또 한가지 주의할 상황은 대리 객체를 둘 이상의 객체가 사용하고 변경 가능할 때다.
* 이럴 땐 데이터를 더 이상 공유할 일이 없어서 위임을 상속으로 바꿀 수 없다.
* 데이터 공유는 상속으로 되돌릴 수 없는 작업이다.
* 반면, 객체가 변경 불가일 땐 바로 복사할 수도 있고 다른 부분에선 모르기 때문에 데이터 공유가 문제되지 않는다.



#### 12.2 방법

* 위임 클래스를 대리 객체의 하위클래스로 만들자
* 대리 필드에 대리 객체를 할당하자.
* 단순 위임 메서드를 전부 삭제하자.
* 다른 우임 부분을 전부 대리 객체 자체를 호출하는 코드로 바꾸자.
* 대리 필드를 삭제하자.



#### 12.3 예제

```java
public class Person {
	String name;

	public String getName() {
		return name;
	}

	public void setName(String arg) {
		this.name = arg;
	}
	
	public String getLastName() {
		return name.substring(name.lastIndexOf(' ')+1);
	}
}

public class Employee {
	Person person = new Person();
	
	public String getName() {
		return person.getName();
	}
	
	public void setName(String arg) {
		person.setName(arg);
	}
	
	public String toString() {
		return "사원: " + person.getLastName();
	}
}
```

* 우선 다음과 같이 하위클래스를 선언하자.

```java
class Employee extends Person
```

* 이 시점에서 컴파일하면 충돌하는 메서드가 있을 경우 경고가 뜬다.
* 이런 충돌은 같은 이름의 메서드가 반환 타입이 다르거나 다른 예외를 통지할 때 발생한다.
* 이런 문제는 메서드명 변경을 적용해 수정해야 한다.
* 다음으로 대리 필드에 대리 객체 자체를 할당하자.
* getName과 setName 같은 단순 위임 메서드를 전부 삭제해야 한다.
* 하나라도 남겨두면 무한루프 걸린다.
* 완료했으면 위임메서드를 사용하는 메서드를 수정하자. 

```java
public class Employee extends Person {
	public String toString() {
		return "사원: " + getLastName();
	}
}
```

* 위임메서드를 사용하는 모든 메서드를 삭제했으면 person 필드를 삭제하자.