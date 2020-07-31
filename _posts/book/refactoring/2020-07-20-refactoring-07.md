---
title:  "데이터 체계화"
excerpt: "데이터 체계화"
classes: wide
categories:
  - Refactoring
tags:
  - [Refactoring]
last_modified_at: 2020-07-20
---



# 데이터 체계화

* 읽기/쓰기 메서드를 통해 접근해야 할때는 필드 자체 캡슐화를 실시한다.
* 데이터 값을 객체로 전환 기법을 실시하면 불명확한 데이터를 뚜렷한 객체로 바꿀 수 있다.
* 이런 객체가 여러 부분에 사용될 인스턴스라면 값을 참조로 전환을 실시해서 참조 객체로 만들면 된다.
* 배열이 데이터 구조 역할을 할 때는 배열을 객체로 전환을 실시하면 그 데이터 구조가 더 선명해진다.
* 상수, 배열크기 같은 숫자들은 기호 상수로 전환 기법을 실시한다.
* 단방향 연결, 양방향 연결 전환 기법.
* 관측 데이터 복제 기법.
* public 데이터를 정리할 대는 필드 캡슐화
* 데이터가 컬렉션이라면 컬렉션 캡슐화
* 하나의 레코드가 통째로 공개되어 있으면 레코드를 데이터 클래스로 전환을 실시하자.
* 분류부호란, 일종의 인스턴스에 관한 특별한 내용을 나타내는 특수한 값이다.
* 분류부호가 정보만 제공하고 클래스의 기능을 변경하진 않을 때는 분류 분호를 클래스로 전환기법을 실시.
* 클래스의 기능이 분류 부호에 영향을 미칠 땐 가능하다면 분류 부호를 하위클래스로 전환 기법을 실시한다.
* 이 기법을 적용하기 불가능할 땐 더욱 복잡하지만 유연한 분류 부호를 상태/전략 패턴으로 전환 기법을 실시한다.



### 1. 필드 자체 캡슐화

* 필드에 직접 접근하던 중 그 필드로의 결합에 문제가 생길 땐 그 필드용 읽기/쓰기 메서드를 작성해서 두 메서드를 통해서만 필드에 접근하게 만들자.

```java
private int low;
private int high;

boolean includes(int arg) {
    return arg >= low && arg <= high;
}
```

```java
private int low;
private int high;


public int getLow() {
    return low;
}

public int getHigh() {
    return high;
}

boolean includes(int arg) {
    return arg >= getLow() && arg <= getHigh();
}
```



#### 1.1 동기

* 기본적으로 변수 간접 접근 방식을 사용하면 하위 클래스가 메서드에 해당 정보를 가져오는 방식을 재정의할 수 있다.
* 데이터 관리가 더 유연해진다.
* 직접 접근 방식은 코드를 알아보기 쉬움.
* 팀원끼리 정해서 하는 편. 혼자 남았을 경우 직접 접근 방식을 우선적으로 고려.
* 그러다가 이상이 생기면 간접 접근으로 바꾼다.
* 상위클래스 안의 필드에 접근하되 이 변수 접근을 하위클래스에서 계산된 값으로 재정의해야 할 때다. 
* 먼저 필드를 자체 캡슐화한 후 필요할 때 읽기 메서드와 쓰기 메서드를 재정의하면 된다.



#### 1.2 방법

* 필드 읽기, 쓰기 메서드를 작성하자.
* 그 필드 참조 부분을 전부 찾아서 읽기 메서드와 쓰기 메서드로 고치자.
  * 필드 읽기 코드를 읽기 메서드 호출로 수정하고, 대입문을 쓰기 메서드 호출로 수정하자.
  * 컴파일러를 이용하면 필드명을 임시로 변경해서 쉽게 검사할 수 있다.
* 필드를 private로 만들자.
* 참조 코드를 빠짐없이 수정했는지 다시 확인하자.



#### 1.3 예제

* 위의 예제와 비슷
* 자체 캡슐화를 실시할 때는 생성자 안에 쓰기 메서드를 사용할 때 주의해야 한다.
* 대체로 객체가 생성된 후엔 속성을 변경하려고 쓰기 메서드를 사용하므로, 쓰기 메서드에 초기화 시점과 다른 기능이 추가됐을 수 있다고 전제할 때가 많다.
* 이럴 땐 생성자나 별도의 초기화 메서드에서 직접 접근하게 하는 것이 좋다.

```java
public class IntRange {
	private int low;
	private int high;
	
	IntRange(int low, int high) {
		initialize(low, high);
	}
	
	private void initialize(int low, int high) {
		this.low = low;
		this.high = high;
	}
	
	public int getLow() {
		return low;
	}

	public int getHigh() {
		return high;
	}

	boolean includes(int arg) {
		return arg >= getLow() && arg <= getHigh();
	}
}
```

* 이렇게 해두면 다음과 같은 하위 클래스가 생길 때 편리해진다.

```java
public class CappedRange extends IntRange{
	
	private int cap;

	CappedRange(int low, int high, int cap) {
		super(low, high);
		this.cap = cap;
	}
	
	int getCap() {
		return cap;
	}
	
	int getHigt() {
		return Math.min(super.getHigh(), getCap());
	}
}
```



### 2. 데이터 값을 객체로 전환

* 데이터 항목에 데이터나 기능을 더 추가해야 할 때는 데이터 항목을 객체로 만들자.



#### 2.1 동기

* 주로 개발 초기 단계에서는 단순 정보를 간단한 데이터 항목으로 표현하는 사안에 대해 결정한다.
* 개발이 진행되고 점점 복잡해진다.
* 예를 들어 전화번호의 경우 처음에는 문자열로 표현하다가 형식화, 지역번호 추출 등 특수 기능이 필요해진다.
* 한두 항목은 객체 안에 메서드를 넣어도 되지만 금세 중복되거나 복잡해진다.
* 이때 데이터 값을 객체로 전환한다.



#### 2.2 방법

* 데이터 값을 넣을 클래스를 작성하자. 그 클래스에 원본 클래스 안의 값과 같은 타입의 final 필드를 추가하자. 그 필드를 인자로 받는 생성자와 읽기 메서드를 추가하자.
* 원본 클래스에 든 필드의 타입을 새 클래스로 바꾸자.
* 원본 클래스 안의 읽기 메서드를 새 클래스의 읽기 메서드를 호출하게 수정하자.
* 그 필드가 원본 클래스 생성자 안에 사용된다면 새 클래스의 생성자를 이용해서 필드를 대입하자.
* 새 클래스의 새 인스턴스를 생성하게끔 읽기 메서드를 수정하자.
* 새 객체에 값을 참조로 전환을 적용해야 할 수도 있다.



#### 2.3 예제

* Order 클래스는 주문 고객을 문자열로 저장하며 customer를 객체로 전환해야 한다.
* 이렇게 하면 주소나, 신용등급 같은 데이터 저장 장소와 정보를 이용한 기능이 생긴다.

```java
public class Order {
	private String customer;
	
	public Order(String customer) {
		this.customer = customer;
	}

	public String getCustomer() {
		return customer;
	}

	public void setCustomer(String arg) {
		this.customer = arg;
	}
}
```

* 앞의 클래스를 사용하는 일부 코드는 다음과 같다.

```java
private static int numberOfOrderFor(Collection<Order> orders, String customer) {
    int result = 0;
    Iterator<Order> iter = orders.iterator();

    while(iter.hasNext()) {
        Order each = iter.next();

        if(each.getCustomer().equals(customer)) {
            result++;
        }
    }

    return result;
}
```

* 새로 Customer 클래스를 작성한다.
* 문자열 속성의 final 필드에 String 속성을 부여한 이유는 Order 클래스가 현재 String을 사용하기 때문이다.

```java
public class Customer {
	private final String name;
	
	public Customer(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}
}
```

* 이제 customer 필드의 타입을 수정하고 그 필드를 참조하는 메서드를 수정해서 Customer 클래스에 있는 적절한 참조를 사용하게 만든다.
* 메서드, 변수 명들을 더 명확하게 한다.

```java
public class Order {
	private Customer customer;
	
	public Order(String customerName) {
		this.customer = new Customer(customerName);
	}

	public String getCustomerName() {
		return customer.getName();
	}

	public void setCustomer(String customerName) {
		this.customer = new Customer(customerName);
	}
	
	private static int numberOfOrderFor(Collection<Order> orders, String customer) {
		int result = 0;
		Iterator<Order> iter = orders.iterator();
		
		while(iter.hasNext()) {
			Order each = iter.next();
			
			if(each.getCustomerName().equals(customer)) {
				result++;
			}
		}
		
		return result;
	}
}
```

* 여기서 추가적인 리팩토링을 실시하나면 기존 Customer 객체를 매개변수로 받는 쓰기 메서드와 생성자를 추가하는 작업
* Customer에 신용등급, 주소 같을 추가하려면 지금은 불가능하다.
* 왜냐하면 Customer는 값 객체로 취급되기 때문이다.
* 각 Order 인스턴스마다 고유한 Customer 객체가 들어있다.
* Customer에 이런 속성을 추가하려면 Customer에 값을 참조로 전환을 적용해서 한 고객의 모든 주문이 하나의 Customer 객체를 사용하게 해야 한다. 이는 '값을 참조로 전환'라는 기법이다.



### 3. 값을 참조로 전환

* 클래스에 같은 인스턴스가 많이 들어 있어서 이것들을 하나의 객체로 바꿔야 할 땐 그 객체를 참조 객체로 전환하자.



#### 3.1 동기

* 많은 시스템에선 객체를 참조 객체와 값 객체로 분류할 수 있다.
* 참조 객체는 고객이나 계좌 같은 것이다.
* 각 객체는 현실에서의 한 객체에 대응하므로, 둘이 같은지 검사할 때는 객체 ID를 사용한다.
* 값 객체는 날짜나 돈 같은 것이다.
* 값 객체는 전적으로 데이터 값을 통해서만 정의된다.
* 참조 객체와 값 객체 중 어느 것을 사용할지 결정하기가 애매할 때도 있다.
* 처음에는 변경불가 데이터가 들어 있는 값을 사용했다가 나중에 변경 가능 데이터를 추가한다.
* 그 변경으로 객체를 참조하는 모든 곳이 영향 받는지 확인할 때도 있다.



#### 3.2 방법

* 생성자를 팩토리 메서드로 전환을 실시하자.
* 참조 객체로의 접근을 담당할 객체를 정하자.
  * 이 기능은 정적 딕셔너리가 레지스트릐 객체가 담당할 수도 있다.
  * 참조 객체로의 접근을 둘 이상의 객체가 담당할 수도 있다.
* 객체를 미리 생성할지 사용하기 직전에 생성할지를 정하자.
  * 객체를 미리 생성했다가 메모리에서 가져오면 사용 전에 미리 로딩되어 있는지 확인해야 한다.
* 참조 객체를 반환하게 팩토리 메서드를 수정하자.
  * 객체를 미리 생성할 경우, 존재하지 않는 객체 요청에 대한 에러 처리를 어떻게 할지 정해야 한다.
  * 팩토리 메서드가 원본 객체를 반환함을 한눈에 알 수 있게 팩토리 메서드에 메서드명 변경을 적용해야 할 수도 있다.



#### 3.3 예제

* "데이터 값을 객체로 전환" 절에서 사용한 예제를 그대로 사용
* 이때 Customer 객체는 값 객체이다.
* 각 Order 인스턴스에는 고유의 Customer 객체가 있다.
* 모든 Customer 객체는 여러 주문을 한 사람이 주문했다면 개념상 동일한 고객을 나타내는 객체이다.
* 이럴 때 Customer 객체를 하나만 사용하도록 수정해야 한다.
* 우선 생성자를 팩토리 메서드로 전환을 적용한다.
* 이렇게 하면 생성 절차를 제어할 수 있는데, 이것은 나중에 필요해진다.
* Customer 객체에 다음과 같이 팩토리 메서드를 정의하자.
* 그리고 생성자 호출을 팩토리 메서드 호출로 수정하자.
* 그 생성자 메서드를 private로 만들자.

```java
public class Customer {
	private final String name;
	
	private Customer(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}
	
	public static Customer create(String name) {
		return new Customer(name);
	}
}

public class Order {
    public Order(String customerName) {
		this.customer = Customer.create(customerName);
	}
}
```

* 이제 Customer 인스턴스에 접근할 방법을 정해야 한다.
* 별도의 객체를 사용하는 것이 좋다.
* 그럴 경우엔 Order에 라인 항목 같은 것을 넣으면 좋다.
* Order는 라인 항목에 접근하는 역할을 한다.
* 그러나 이런 상황에서는 그 정도로 분명한 객체가 존재하지 않는다.
* 이럴 때는 보통 접근거점으로 사용할 레지스트리 객체를 생성한다.

```java
private static Dictionary instances = new Hashtable<>();
```

* 그 다음, Customer 인스턴스들을 요청이 있을 때 즉석에서 생성할지 아니면 미리 생성해 둘지 결정해야 한다.
* 여기서는 미리 생성해 두는 방식으로 하겠다.
* 이 애플리케이션의 시작 코드 코드 안에 사용되는 Customer 인스턴스들을 로딩하는 코드를 넣는다.
* Customer 인스턴스들은 데이터베이스나 파일에 저장되어 있을 수 있다.
* 간결하게 하고자 여기선 직접 코드에 넣는다.
* 나중에 이 방식을 변경하려면 언제든 알고리즘 전환을 실시한다.

* 팩토리 메서드를 수정해서 미리 생성해둔 Customer 인스턴스를 반환하게 하자.
* Create 메서드는 반드시 미리 생성되어 있던 Customer 인스턴스를 반환하므로 메서드명 변경을 실시해서 이것을 확실하게 나타낸다.

```java
public class Customer {
	private final String name;
	private static Dictionary<String, Customer> instances = new Hashtable<>();
	
	private Customer(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}
	
	public static Customer getNamed(String name) {
		return instances.get(name);
	}
	
	static void loadCustomer() {
		new Customer("우리 렌터카").store();
		new Customer("커피 자판기 운영업 협동조합").store();
		new Customer("삼천리 가스 공정").store();
	}

	private void store() {
		instances.put(this.getName(), this);
	}
}
```



### 4. 참조를 값으로 전환

* 참조 객체가 작고 수정할 수 없고 관리하기 힘들 땐 그 참조 객체를 객체로 만들자.



#### 4.1 동기

* 값을 참조로 전환 기법과 마찬가지로 참조 객체와 값 객체 중 무엇을 사용할지 판단하기가 항상 쉽지가 않다.
* 참조 객체를 사용한 작업이 복잡해지는 순간이 참조를 값으로 바꿔야 할 시점이다.
* 참조 객체는 어떤 식으로든 제어되어야 한다.
* 개발자는 컨트롤러에 항상 적절한 객체를 요청해야 한다.
* 값 객체를 분산 시스템이나 병렬 시스템에 주로 사용된다.
* 값 객체는 변경할 수 없어야 한다는 주요 특성이 있다.
* 하나에 대한 질의를 호출하면 항상 결과가 같아야 한다.
* 그렇기만 하면 같은 것을 나타내는 객체가 많아도 문제가 없다.
* 값을 변경할 수 없다면, 어떤 객체를 수정했을 때 같은 것을 나타내는 다른 객체들도 전부 바뀌는지 확인해야 하는데 너무 힘들다.
* 여기서 변경 불가란 예를 들어 currency(통화)와 value(가치)가 들어 있는 Money 클래스가 있다고 하면 그 클래스는 대체 불가인 값 객체이다.
* 그렇다고 해서 salary(월급)이 변경되는 것은 아니다. 
* 월급을 변경하려면 기존 Money 객체에 들어 있는 양을 수정할게 아니라 기존의 Money 객체를 새 Money 객체로 교체해야 한다.
* 관계는 바꿀 수 있지만 Money 객체 자체는 바꿀 수 없다.



#### 4.2 방법

* 전환할 객체가 변경불가인지 변경 가능인지 검사하자.
  * 전환할 객체가 변경불가가 아니면, 변경불가가 될 때까지 쓰기 메서드 제거를 실시하자.
  * 전환할 객체가 변경불가이면 이 리팩토링은 관두자.
* equlas, hash 메서드를 작성하자.
* 팩토리 메서드를 삭제하고 생성자를 public으로 만들어야 좋을지 생각해보자.



#### 4.3 예제

```java
public class Currency {
	private String code;
	
	public Currency (String code) {
		this.code = code;
	}
	
	public String getCode() {
		return code;
	}
}
```

* 이 클래스의 기능은 단순히 code를 저장하고 반환하는 것이다.
* 이 클래스는 참조 객체이므로 사용할 인스턴스를 가져오려면 다음과 같이 주어진 code에 Currency의 동일 인스턴스를 반환하는 메서드를 사용해야 한다.

```java
Currency usd = Currency.get("USD");
```

* Currency 클래스에는 여러 인스턴스가 들어 있다.
* 생성자만 사용하는 것은 불가능하다.
* 그래서 private인 것이다.

```java
new Curreny("USD").equals(new Curreny("USD"));
```

* 여기서 참조 객체는 변경불가이므로 이제 다음과 같이 equals 메서드를 재정의 한다.

```java
public class Currency {
	private String code;
	
	public Currency (String code) {
		this.code = code;
	}
	
	public String getCode() {
		return code;
	}
	
	public boolean equals(Object arg) {
		if(!(arg instanceof Currency)) {
			return false;
		}
		
		Currency other = (Currency) arg;
		
		return code.equals(other.code);
	}
}
```

* equals 메서드를 정의할 때 다음과 같이 hashCode 메서드도 정의해야 한다. 
* 간단히 하려면 equals 메서드에 사용되는 모든 필드의 해시 코드를 가져다 XOR 비트 연산을 수행해야 한다.

```java
public int hashCode() {
	return code.hashCode();
}
```

* 이제 Currency 인스턴스를 원하는 수만큼 생성할 수 있다.
* 다음과 같이 Currency 클래스와 팩토리 메서드에 있는 모든 컨트롤러 기능을 삭제하고 생성자만 사용해도 된다.
* 그 생성자를 이제 public으로 만들 수 있다.



### 5. 배열을 객체로 전환

* 배열을 구성하는 특정 원소가 별의별 의미를 지닐 땐 그 배열을 각 원소마다 필드가 하나씩 든 객체로 전환하자.



#### 5.1 동기

* 배열은 비슷한 객체들의 컬렉션을 일정 순서로 담는 용도로만 사용해야 한다.



#### 5.2 방법

* 배열 안의 정보를 표현할 새 클래스를 작성하자. 그 클래스 안에 배열을 저장할 public 필드를 하나 작성하자.
* 배열 참조 부분을 전부 새 클래스 참조로 수정하자.
* 배열의 각 원소마다 참조 코드에 사용할 읽기 메서드를 하니씩 넣자.
* 배열 원소의 용도를 따서 읽기 메서드 이름을 정하자.
* 참조 부분을 읽기 메서드 호출로 전부 수정하자.
* 배열 참조 부분을 전부 메서드로 교체했으면 배열을 private로 만들자.
* 클래스 안에 배열의 각 원소에 대응되는 하나의 필드를 생성한 후, 그 필드는 사용하게끔 읽기/쓰기 메서드를 수정하자.
* 모든 원소를 필드로 교체했으면 배열을 삭제하자.



#### 5.3 예제

* 스포츠팀 이름, 승수, 패수를 저장할 배열이 있다고 가정

```java
String[] row = new String[3];
row[0] = "리버풀";
row[1] = "15";

String name = row[0];
int wins = Integer.parseInt(row[1]);
```

* 이 배열을 객체로 바꾸려면 새 클래스로 하나 작성하자.

```java
public class Performance {	
	public String[] data = new String[3];
}
```

```java
Performance row = new Performance();
		
row.data[0] = "리버풀";
row.data[1] = "15";

String name = row.data[0];
int wins = Integer.parseInt(row.data[1]);
```

* 읽기/쓰기 메서드를 만든다.

```java
public class Performance {
	
	public String[] data = new String[3];
	
	public String getName() {
		return data[0];
	}
	
	public void setName(String arg) {
		data[0] = arg;
	}
	
	public int getWins() {
		return Integer.parseInt(data[1]);
	}
	
	public void setWins(String arg) {
		data[1] = arg;
	}
}
```

* row 사용하는 부분마다 읽기/쓰기 메서드를 사용하게 고치자.

```java
Performance row = new Performance();

row.setName("리버풀");
row.setWins("15");

String name = row.getName();
int wins = row.getWins();
```

* 수정이 완료되면 private으로 바꾸고 배열 원소에 맞게 필드 변수를 추가한다.

```java
public class Performance {
	
	private String name;
	private int wins;
	private int loses;
	
	public String getName() {
		return name;
	}
	
	public void setName(String arg) {
		name = arg;
	}
	
	public int getWins() {
		return wins;
	}
	
	public void setWins(int arg) {
		wins = arg;
	}
}
```

```java
Performance row = new Performance();
		
row.setName("리버풀");
row.setWins(15);

String name = row.getName();
int wins = row.getWins();
```



### 6. 관측 데이터 복제

* 도메인 데이터는 GUI 컨트롤 안에서만 사용 가능한데, 도메인 메서드가 그 데이터에 접근해야 할 땐 그 데이터를 도메인 객체로 복사하고, 양측의 데이터를 동기화하는 관측 인터페이스 observer를 작성하자.

![]({{site.url}}/assets/images/ref20.PNG)

#### 6.1 동기

* 계층구조가 체계적인 시스템은 비즈니스 로직 처리 코드와 사용자 인터페이스 처리 코드가 분리되어 있다.
* 이유는 다음과 같다.
  * 비슷한 비즈니스 로직을 여러 인터페이스가 처리해야 하는 경우라서
  * 비즈니스 로직까지 처리하려면 사용자 인터페이스가 너무 복잡하니까
  * GUI와 분리된 도메인 객체가 더욱 유지보수하기 쉬우니까
  * 두 부분을 서로 다른 개발자가 다루게 할 수 있어서
* 기능은 간단히 분리할 수 있어도 데이터는 분리하기 어려울 때가 많다.
* 도메인 모델에 있는 데이터와 같은 의미를 지닌 GUI컨트롤에 넣어야 하기 때문이다.
* MVC 패턴부터 시작해서 그 이후로, 사용자 인터페이스 프레임워크는 이러한 데이터를 제공하고 모든 데이터의 동기화를 유지하는 다층 시스템을 사용했다.
* 비즈니스 로직이 사용자 인터페이스 안에 들어 있는 2계층 방식으로 개발된 코드가 있다면 인터페이스에서 기능을 분리해야 한다.
* 대부분 메서드를 쪼개서 옮기는 작업이다.
* 하지만 데이터는 그저 옮기기만 해선 안 되고 복제하고 동기화 기능까지 작성해야 한다.



#### 6.2 방법

* 표현 클래스를 도메인 클래스의 관측 인터페이스로 만들자.
  * 도메인 클래스가 없으면 하나 작성하자.
  * 표현 클래스에 도메인 클래스로의 연결 코드가 없으면 표현 클래스의 필드에 도메인 클래스를 대입하자.
* GUI 클래스 안의 도메인 데이터를 대상으로 필드 자체 캡슐화를 실시하자.
* 이벤트 핸들러 메서드 안에 쓰기 메서드 호출 코드를 추가하자. 이 쓰기 메서드는 직접 접근 방식으로 컴포넌트를 현재 값으로 수정한다
  * 현재 값에 따라 컴포넌트 값을 수정하는 메서드를 이벤트 핸들러 안에 넣자. 물론 이 단계는 컴포넌트 값을 현재 값으로 설정하는 것뿐이니 필요가 없지만, 쓰기 메서드를 사용하면 어떤 기능이든 실행할 수 있다.
  * 이렇게 수정할 때 읽기 메서드를 사용하지 말고 컴포넌트에 직접 접근하자. 나중에 읽기 메서드는 도메인에서 값을 가져오는 데 쓰이는데, 그 값은 쓰기 메서드 실행 전까진 변하지 않는다.
  * 테스트 코드로 이벤트 처리 절차를 확인하자.
* 도메인 클래스 안에 데이터와 읽기/쓰기 메서드를 정의하자.
  * 도메인에 있는 쓰기 메서드가 반드시 관측 인터페이스 패턴의 통지 절차를 시작하게 만들자.
  * 도메인 클래스 안의 데이터 타입을 표현 클래스 안의 데이터 타입과 같게 하자. 주로 문자열 타입이다. 이 데이터 타입을 추가 리팩토링 기법을 실시할 때 변환하자.
* 쓰기 메서드가 도메인 필드에 쓰도록 참조를 수정하자.
* 관측 인터페이스의 update 메서드를 도메인 필드에서 GUI 컨트롤로 데이터를 복사하게 수정하자.



#### 6.3 예제

* 일단 내가 당장 필요한 부분이 아니라 스킵



### 7. 클래스의 단방향 연결을 양방향으로 전환

* 두 클래스가 서로의 기능을 사용해야 하는데 한 방향으로만 연결되어 있을 땐 역 포인터를 추가하고 두 클래스를 모두 업데이트할 수 있게 접근 한정자를 수정하자.



#### 7.1 동기

* 애초에 두 클래스를 설정할 때 한 클래스가 다른 클래스를 참조하게 해놓은 경우가 있을 수 있다.
* 나중에 참조되는 클래스를 사용하는 부분에서 그 클래스를 참조하는 객체들을 가져와야 할 수도 있다.
* 즉, 포인터를 역방향으로 참조해야 한다.
* 포인터는 단방향 연결이라서 이런 식의 역방향 참조는 불가능하다.
* 다른 경로를 찾아서 해결할 수 있을 때도 있다.
* 그렇게 하면 계산에 비용이 들긴 해도 합리적이며, 이 기능을 사용하는 메서드를 참조되는 클래스에 넣을 수 있다.
* 하지만 이게 쉽지 않을 때도 있으므로 양방향 참조를 설정해야 한다.



#### 7.2 방법

* 역 포인터 참조용 필드를 추가하자.
* 연결 제어 기능을 어느 클래스에 넣을지 정하자.
* 연결 제어 기능이 없는 클래스 안에 헬퍼 메서드를 작성하고, 그 메서드에 제한된 용도가 분명히 드러나는 이름을 붙이자.
* 기존 변경 메서드가 연결 제어 클래스에 들어 있으면 역 포인터를 업데이트하게 변경 메서드를 수정하자.
* 기존 변경 메서드가 연결 제어 클래스에 들어 있으면 제어 클래스 안에 제어 메서드를 작성하고 기존 변경 메서드가 그 메서드를 호출하게 하자.



#### 7.3 예제

* Order 클래스가 Customer 클래스를 참조한다고 하자.

```java
public class Order {
	Customer customer;

	public Customer getCustomer() {
		return customer;
	}

	public void setCustomer(Customer arg) {
		this.customer = arg;
	}
}
```

* Customer 클래스에는 Order 클래스를 참조하는 코드가 들어 있지 않다.
* 우선 Customer 클래스에 필드를 하나 추가하자. 
* 고객 한 명이 어러 건의 주문을 할 수 있으므로 이 필드는 컬렉션이다.
* 고객은 컬렉션 안에서 두 번 이상 같은 주문을 해서는 안 되므로 다음과 같이 set 컬렉션이 적합하다.

```java
public class Customer {
	private Set<Order> orders = new HashSet<Order>();
}
```

* 이 연결을 제어할 클래스를 정해야 한다.

* 연결을 조작하는 로직을 전부 한 곳에 두기 위해 연결 제어 로직은 하나의 클래스에 넣는 것이 좋다.

* 결정 과정은 다음과 같다.

  * 두 객체가 모두 참조 객체이고 연결이 일대다이면 참조가 하나 들어 있는 객체를 제어 객체로 정한다.

    (즉, Customer에는 Order 참조가 여러 개이므로 Order 클래스를 연결 제어 객체로 택한다.)

  * 한 객체가 다른 객체에 포함될 때는 포함되는 객체를 제어 객체로 정한다.

  * 두 객체가 모두 참조 객체이고 연결이 다대다이면, Order나 Customer중 어느 클래스를 연결 제어 객체로 정해도 상관없다.

* Order 클래스에 연결 제어 기능을 구현할 것이므로, 헬퍼 메서드는 주문 컬레견으로 직접 접근 할 수 있게 하는 Customer 클래스에 넣어야 한다.

* Order 변경자는 이것을 사용해서 양측의 포인터 세트를 동기화하게 된다.

```java
public class Customer {
	private Set<Order> orders = new HashSet<Order>();
	
	Set friendOrders() {
		/** 연결을 변경할 때 Order에 의해서만 사용되어야 함 */
		return orders;
	}
}
```

* 이제 역 포인터를 업데이트하게 변경 메서드를 다음과 같이 수정하자.

```java
public void setCustomer(Customer arg) {
    if(customer != null) {
        customer.friendOrders().remove(this);
    }

    this.customer = arg;

    if(customer != null) {
        customer.friendOrders().add(this);
    }
}
```

* 제어하는 변경 메서드의 정확한 코드는 연결의 다양한 조합에 따라 달라진다.
* customer 변수의 값이 null 되는 게 불가능하다면, null 검사를 취소해도 되지만 null 인자는 꼭 검서해야 한다.
* Customer 클래스를 거쳐서 연결을 변경하려면 다음과 같이 Customer 클래스 안에 제어 메서드 호출 코드를 넣으면 된다.

```java
public class Customer {
	private Set<Order> orders = new HashSet<Order>();
	
	Set friendOrders() {
		/** 연결을 변경할 때 Order에 의해서만 사용되어야 함 */
		return orders;
	}
	
	void addOrder(Order arg) {
		arg.setCustomer(this);
	}
}
```

* 하나의 주문을 여러 고객이 할 수 있다면 연결이 다대다가 되므로 제어 메서드는 다음과 같다.

```java
class Order {
    // 연결 제어 메서드
	void addCustomer(Customer arg) {
		arg.friendOrders().add(this);
		customer.add(arg); // 이게 뭔제 모르겠네
	}
	
	void removeCustomer(Customer arg) {
		arg.friendOrders().remove(this);
		customer.remove(arg);
	}
}
```

```java
public class Customer {
	private Set<Order> orders = new HashSet<Order>();
	
	Set friendOrders() {
		/** 연결을 변경할 때 Order에 의해서만 사용되어야 함 */
		return orders;
	}
	
	void addOrder(Order arg) {
		arg.setCustomer(this);
	}

	void removeOrder(Order arg) {
		arg.removeCustomer(this);
	}
}
```



### 8. 클래스의 양방향 연결을 단방향으로 전환

* 두 클래스가 양방향으로 연결되어 있는데 한 클래스가 다른 클래스의 기능을 더 이상 사용하지 않게 됐을 땐 불필요한 방향의 연결을 끊자.



#### 8.1 동기

* 양방향 연결을 유지하고 객체가 적절히 생성되고 제거되는지 확인하는 복잡함이 더해진다.
* 양방향 연결이 익숙하지 않은 대부분의 프로그래머는 에러를 발생시킨다.
* 양방향 연결이 많으면 좀비 객체가 발생하기도 한다.
* 좀비 객체란 참조가 삭제되지 않아 제거되어야 함에도 남아서 떠도는 객체를 뜻한다.
* 양방향 연결로 인해 두 클래스는 서로 종속된다.
* 한 클래스를 수정하면 수정하면 다른 클래스도 변경된다.
* 종속성이 많으면 시스템의 결합력이 강해져서 사소한 수정에도 예기치 못한 각종 문제가 발생한다.



#### 8.2 방법

* 삭제하려는 포인터가 저장된 필드를 읽는 모든 부분을 검사해서 삭제해도 되는지 파악하자.
  * 필드를 직접 읽는 메서드와 그 메서드를 호출하는 다른 메서드도 살펴보자.
  * 포인터를 사용하지 않고 다른 객체를 알아내는 것이 가능한지 보고, 가능하다면 속성 읽기 메서드에 알고리즘 전환을 실시해서 참조 코드가 포인터 없이 속성 읽기 메서드를 사용할 수 있게 하자.
* 참조 코드가 속성 읽기 메서드를 사용해야 한다면 속성 읽기 메서드에 필드 자체 캡슐화를 적용하고 알고리즘 전환을 적용한 후 테스트를 실시하자.
* 참조코드에 읽기 메서드 호출을 넣을 필요가 없다면, 각 필드 사용 부분을 찾아서 필드 안의 객체를 다른 방법으로 가져오게끔 수정하자.
* 필드 안의 속성 읽기 메서드를 모두 삭제했으면 필드 업데이트 코드 전부와 필드를 삭제하자.
  * 필드에 값을 할당하는 여러 군데 있다면 필드 자체 캡슐화를 실시해서 그 모든 부분이 한 개의 속성쓰기 메서드를 사용하게 하자. 테스트를 실시한 후 속성 쓰기 메서드를 빈 메서드로 만들고 다시 테스트한다. 오류가 없으면 필드, 속성 읽기 메서드, 모든 속성 쓰기 메서드 호출 코드를 삭제하자.



#### 8.3 예제

```java
public class Order {
    Customer customer;
    
	double getDiscountedPrice() {
		return getGrossPrice() * (1 - customer.getDiscount());
	}
}
```

```java
public class Order {
	double getDiscountedPrice(Customer customer) {
		return getGrossPrice() * (1 - customer.getDiscount());
	}
}
```

* 책에서 자세히 나와 있는데 내가 볼 땐 "클래스 단방향에서 양방향으로" 예제랑 반대 같음.



### 9. 마법 숫자를 기호 상수로 전환

* 특수 의미를 지닌 리터럴 숫자가 있을 땐 의미를 살린 이름의 상수를 작성한 후 리터럴 숫자를 그 상수로 교체하자.

```java
double potentialEnergy(double mass, double height) {
	return mass * 9.81 * height;
}
```

```java
static final double GRAVITIATONAL_CONSTANT = 9.81;

double potentialEnergy(double mass, double height) {
    return mass * GRAVITIATONAL_CONSTANT * height;
}
```



### 10. 필드 캡슐화

* public 필드는 private으로 만들고 읽기 /쓰기 메서드를 만들자



### 11. 컬렉션 캡슐화

* 메서드가 컬렉션을 반환할 땐, 그 메서드가 읽기 전용 뷰를 반환하게 수정하고 추가 메서드와 삭제 메서드를 작성하자.



#### 11.1 동기

* 컬렉션은 다른 종류의 데이터와는 약간 다른 읽기/쓰기 방식을 사용해야 한다.
* 읽기 메서드는 컬렉션 객체 자체를 반환해서는 안된다.
* 왜냐하면 컬렉션 참조 부분이 컬렉션의 내용을 조작해도 그 컬렉션이 든 클래스는 무슨 일이 일어나는지 모르기 때문이다.
* 쓰기 메서드는 절대 있으면 안되므로, 원소를 추가하는 메서드와 삭제하는 메서드를 대신 사용해야 한다.
* 이렇게 하면 컬렉션이 든 객체가 든 객체가 컬렉션의 원소 추가와 삭제를 통제할 수 있다.
* 이렇게 하면, 컬렉션 참조 부분에 대한 종속성이 줄어든다.



#### 11.2 방법

* 컬렉션 원소를 추가, 삭제 메서드를 작성한다.
* 필드를 빈 컬렉션으로 초기화하자.
* 쓰기 메서드 호출 부분을 찾아서 add 메서드오 remove 메서드 호출로 바꾸든지, 그 위치에 직접 컬렉션에 원소를 추가하고 삭제하는 코드를 작성하자.
  * 쓰기 메서드가 사용되는 경우는 두 가지다. 첫째는 컬렉션이 비어 있을 때고, 둘째는 비어 있지 않은 컬렉션을 교체할 때다.
  * 메서드명 변경을 실시
* set이라는 이름을 initialized나 replace로 변경
* 읽기 메서드를 호출하여 컬렉션을 변경하는 부분을 전부 찾아서 add 메서드, remove 메서드 호출로 바꾸자.
* 컬렉션을 변경하고자 읽기 메서드를 호출하는 부분을 전부 추가/삭제 메서드 호출로 고쳤으면, 컬렉션의 읽기 전용 뷰를 반환하게 읽기 메서드를 수정하자.
* 읽기 메서드 호출부분을 찾고 컬렉션이 컬렉션이 든 객체로 옮겨야 할 코드를 찾아서 메서드 추출과 메서드 이동을 실시해서 컬렉션이 든 객체로 옮기자.



#### 11.2 예제

* 한 사람이 여러 과정을 수강한다.
* 수강 과정을 나타내는 course 클래스는 다음과 같다.

```java
public class Course {
	public Course (String name, boolean isAdvanced) {
		
	}
	
	public boolean isAdvance() {
		
	}
}
```

```java
class Person {
	Set<Course> courses;
	
	public Set getCourses() {
		return courses;
	}
	
	public void setCouses(Set arg) {
		courses = arg;
	}
}
```

* 앞의 인터페이스를 이용하여 과정을 추가하는 코드와 고급과정을 알아내는 코드는 다음과 같다.

```java
Person kent = new Person();
Set s = new HashSet();
s.add(new Course("스몰토크 프로그래밍", false));
s.add(new Course("싱글몰트 위스키 음미하기", true));
kent.setCouses(s);
// 여기에 assert 있어야 함
Course refact = new Course("리팩토링", true);
kent.getCourses().add(refact);
kent.getCourses().add(new Course("지독한 빈정거림", false));
// 여기도 assert
kent.getCourses().remove(refact);
// 여기도 assert
```

```java
public class Course {
	String name;
	boolean isAdvanced;
	
	public Course (String name, boolean isAdvanced) {
		this.name = name;
		this.isAdvanced = isAdvanced;
	}
	
	public boolean isAdvance() {
		return isAdvanced;
	}
	
	int countAdvance(Person person) {
		
		Iterator iter = person.getCourses().iterator();
		int count = 0;
		
		while(iter.hasNext()) {
			Course each = (Course) iter.next();
			
			if(each.isAdvance()) {
				count++;
			}
		}
		
		return count;
	}
}
```

* 이때 다음과 같이 적절한 컬렉션 변경 메서드를 작성한다.
* courses 초기화하면 더 좋다.

```java
class Person {	
    Set<Course> courses = new HashSet<Course>();
    
	public void addCourse(Course arg) {
		courses.add(arg);
	}
	
	public void remove(Course arg) {
		courses.remove(arg);
	}
}
```

* 다음 쓰기 메서드 호출 부분을 관찰하자.
* 참조 부분이 여러 곳이고 쓰기 메서드가 많이 사용된다면 쓰기 메서드 내용을 추가/삭제 기능의 코드도 바꿔야 한다.
* 이 과정은 쓰기 메서드의 사용 방식에 따라 복잡한 정도가 둘로 나뉜다.
* 간단한 경우는 참조 부분이 쓰기 메서드를 호출해서 값을 초기화할 때다.
* 즉, 쓰기 메서드가 적용되기 전에 과정이 없는 경우다.
* 이럴 땐 쓰기 메서드 내용을 다음과 같이 과정 추가 메서드 호출로 바꾸면 된다.
* 동시에 메서드 이름을 바꾼다. (기능적 의미를 더 분명히 전달)

```java
class Person {
	Set<Course> courses = new HashSet<Course>();
	
	public void initializeCourses(Set arg) {
		assertTrue(courses.isEmpty());
		
		Iterator iter = arg.iterator();
		
		while(iter.hasNext()) {
			addCourse((Course) iter.next());
		}
	}
}
```

* 초기화하려고 원소를 추가할 때 추가 기능이 확실히 필요 없다면 다음과 같이 루프를 삭제하고 addAll 메서드 사용한다.

```java
public void initializeCourses(Set arg) {
    assertTrue(courses.isEmpty());		
    courses.addAll(arg);
}
```

* 이전 세트가 비어 있더라고 그 세트에 직접 대입할 순 없다.
* 참조 부분이 전달 후 그 세트를 수정한다면 캡슐화에 위배된다.
* 그래서 복사해 둬야 한다.
* 참조 부분이 단순히 세트를 생성하고 쓰기 메서드를 사용한다면 다음과 같이 그 참조 부분의 add 메서드와 remove 메서드를 직접 사용하게 수정하고 쓰기 메서드를 삭제하면 된다.

```java
Person kent = new Person();
Set s = new HashSet();
s.add(new Course("스몰토크 프로그래밍", false));
s.add(new Course("싱글몰트 위스키 음미하기", true));
kent.initializeCourses(s);
```

```java
Person kent = new Person();
kent.addCourse(new Course("스몰토크 프로그래밍", false));
kent.addCourse(new Course("싱글몰트 위스키 음미하기", true));
```

* 읽기 메서드는 다음과 같다

```java
kent.addCourse(new Course("지독한 빈정거림", false));
```

* 모든 읽기 메서드 호출 부분을 수정했으면 다음과 같이 읽기 메서드의 내용을 수정불가 뷰를 반환하게 수정해서 읽기 메서드를 통해 수정하는 부분이 없는지 확인하자.

```java
class Person {
	Set<Course> courses = new HashSet<Course>();
	
	public Set getCourses() {
		return Collections.unmodifiableSet(courses);
	}
}
```

* 이제 컬렉션을 캡슐화 한다.

* countAdvance의 메서드는 person에서 하는게 좋다. Person 클래스로 옮긴다.

```java
class Person {
	private Set<Course> courses = new HashSet<Course>();
	
	public Set getCourses() {
		return Collections.unmodifiableSet(courses);
	}
	
	public void initializeCourses(Set arg) {
		assertTrue(courses.isEmpty());
		
		Iterator iter = arg.iterator();
		
		while(iter.hasNext()) {
			addCourse((Course) iter.next());
		}
		
		courses.addAll(arg);
	}
	
	public void addCourse(Course arg) {
		courses.add(arg);
	}
	
	public void remove(Course arg) {
		courses.remove(arg);
	}
	
	int countAdvance() {
		
		Iterator iter = getCourses().iterator();
		int count = 0;
		
		while(iter.hasNext()) {
			Course each = (Course) iter.next();
			
			if(each.isAdvance()) {
				count++;
			}
		}
		
		return count;
	}
}
```



### 12. 레코드를 데이터 클래스로 전환

* 전통적인 프로그래밍 환경에서 레코드 구조를 이용한 인터페이스를 제공해야 할 땐 레코드 구조를 저장할 덤 데이터 객체를 작성하자.



#### 12.1 동기

* 레코드 구조를 사용할 경우 외부 요소를 처리할 인터페이스 역할을 하는 클래스를 작성하는 것이 좋다.
* 다른 필드와 메서드는 나중에 옮기면 된다.



#### 12.2 방법

* 레코드를 표현할 클래스를 작성하자.

* 그 클래스에 각 데이터 항목에 대한 읽기/쓰기 메서드를 작성하고 private 필드를 선언하자.

* 이 객체에는 아직 기능이 없지만 추후 소개할 기법에서 기능 추가에 대해 설명한다.

  

### 13. 분류 부호를 클래스로 전환

* 기능에 영향을 미치는 숫자형 분류 부호가 든 클래스가 있을 땐 그 숫자를 새 클래스로 바꾸자.



#### 13.1 동기

* 숫자형 분류 부호, 즉, 열거 타입은 코드를 이해하기 쉬워지나 그것 뿐이다.
* 분류부호를 인자로 받는 모든 메서드는 숫자만을 인자로 받으며, 상징적 이름을 전달할 방법은 없다.
* 그래서 코드를 이해하기 힘들어질 수 있고 버그가 생길 수도 있다.

* 숫자형 분류 부호를 클래스로 빼내면 컴파일러는 그 클래스 안에서 종류 판단을 수행할 수 있다.
* 그 클래스 안에 팩토리 메서드를 작성하면 유효한 인스턴스만 생성되는지와 그런 인스턴스가 적절한 객체로 전달되는지를 정적으로 검사할 수 있다.
* 그렇다고 무작정하는 것이 아니라 그 전에 분류 부호를 다른 것으로 전환하는 방안을 고려한다.
* 분류 부호를 클래스로 만드는 건 분류 부호가 순수한 데이터일 때만 실시해야 한다.
* 다시 말해, 분류 부호가 switch 문안에 사용되어 다른 기능을 수행하거나 메서드를 호출할 땐 클래스로 전환하면 안 된다.
* switch문에는 임의의 클래스를 사용할 수 없으며 오직 정수 타입만 사용 가능하므로 클래스로의 전환은 실패를 맞게 된다.
* 더 중요한 건, 모든 switch문은 조건문을 재정의로 전환 기법을 적용해 전부 없애야 하는 대상이다.
* 분류 부호가 값에 따라 다른 기능을 호출하지 않더라도 분류 부호 클래스로 만들면 더 좋은 기능이 있을 수도 있으므로, 메서드 이동 기법을 한두 차례 실시할 것을 고려하자.



#### 13.2 방법

* 분류 부호의 종류를 판단할 새 클래스를 작성하자.
  * 그 클래스 안에 분류 부호와 대응하는 code 필드를 선언하고 그 값을 읽을 읽기 메서드를 작성하자. 
  * 그 클래스에서 생성이 허용되는 인스턴스를 저정할 static 변수와, 원래의 클래스에 따라 다른 인자를 받아서 적절한 인스턴스를 반환하는 static 메서드를 저장한 static 변수도 선언해야 한다.
* 새 클래스를 사용하게 원본 클래스의 내용을 수정하자.
  * 기존 코드를 사용하는 인터페이스는 그대로 놔두되, static 필드는 새 클래스를 통해 코드를 생성하게 수정하자.
  * 다른 코드를 사용하는 메서드도 새 클래스에서 부호 숫자를 읽게 수정하자.
* 원본 클래스 코드를 사용하는 원본 클래스 안의 각 메서드마다, 그에 대응하는 새 클래스를 사용하는 새 메서드를 작성하자.
  * 코드를 인자로 받는 메서드는 새 클래스의 인스턴스를 인자로 받는 새 메서드로 바꿔야 한다.
  * 코드를 반환하는 메서드는 새 클래스 인스턴스를 반환하는 새 메서드로 바꿔야 한다.
  * 기존 코드를 사용하는 프로그램을 더욱 이해하기 쉽게 만들려면 새 읽기/쓰기 메서드를 작성하기 전에 기존의 읽기/쓰기 메서드에 메서드명 변경을 적용해야 한다.
* 원본 클래스 호출 부분을 한 번에 하나씩 새 인터페이스 호출로 수정하자.
* 전부 수정했으면 컴파일과 테스트를 실시하자.
  * 컴파일과 테스트 결과가 일관되게 나오기까지 여러 메서드를 수정해야 할 가능성도 있다.
* 기존 코드를 사용하는 기존 인터페이스를 삭제하고, 기존 코드의 static 변수 선언도 삭제하자.



#### 13.3 예제

* Person 클래스에는 혈액형 그룹이 있다.

```java
class Person {
	public static final int O = 0;
	public static final int A = 1;
	public static final int B = 2;
	public static final int AB = 3;
	
	private int bloodGroup;
	
	public Person (int bloodGroup) {
		this.bloodGroup = bloodGroup;
	}
	
	public void setBllodGroup(int arg) {
		this.bloodGroup = arg;
	}
	
	public int getBloodGroup() {
		return bloodGroup;
	}
}
```

* 먼저 다음과 같이 혈액형 그룹을 판단할 BloodGroup 클래스를 작성하고, 분류 부호 숫자가든 인스턴스를 작성하자.

```java
public class BloodGroup {
	public static final BloodGroup O = new BloodGroup(0);
	public static final BloodGroup A = new BloodGroup(1);
	public static final BloodGroup B = new BloodGroup(2);
	public static final BloodGroup AB = new BloodGroup(3);
	
	private final int code;
	
	private BloodGroup(int code) {
		this.code = code;
	}
	
	public int getCode() {
		return this.code;
	}
	
	public static BloodGroup code(int arg) {
		return values(arg);
	}
}
```



* Person 클래스의 코드를 수정해서 새 클래스를 사용하게 하자.

```java
class Person {
	public static final int O = BloodGroup.O.getCode();
	public static final int A = BloodGroup.A.getCode();
	public static final int B = BloodGroup.B.getCode();
	public static final int AB = BloodGroup.AB.getCode();
	
	private BloodGroup bloodGroup;
	
	public Person (int bloodGroup) {
		this.bloodGroup = BloodGroup.code(bloodGroup);
	}
	
	public void setBloodGroup(int arg) {
		this.bloodGroup = BloodGroup.code(arg);
	}
	
	public int getBloodGroup() {
		return bloodGroup.getCode();
	}
}
```

* 이것으로써 BloodGroup 클래스 안에서 런타임 분류 부호 검사가 이뤄진다.
* 이 수정에서 진정한 효과를 얻으려면 Person 클래스 사용 부분이 int 타입 대신 bloodGroup을 사용하게 해야 한다.
* 우선 Person 클래스의 bloodGroup을 읽는 읽기 메서드명 변경을 적용해서 변경된 코드를 이해하기 쉽게 만들자.

* 새 클래스를 사용하는 읽기 메서드를 작성하자.
* 생성자 메서드와 새 클래스를 사용하는 쓰기 메서드도 작성하자.

```java
class Person {
	public static final int O = BloodGroup.O.getCode();
	public static final int A = BloodGroup.A.getCode();
	public static final int B = BloodGroup.B.getCode();
	public static final int AB = BloodGroup.AB.getCode();
	
	private BloodGroup bloodGroup;
	
	public Person (int bloodGroup) {
		this.bloodGroup = BloodGroup.code(bloodGroup);
	}
	
	public void setBloodGroup(int arg) {
		this.bloodGroup = BloodGroup.code(arg);
	}
	// 수정함. getBloodGroupCode 함수가 생기므로
	public BloodGroup getBloodGroup() {
		return bloodGroup;
	}
	// 아래 새로 추가
	public Person (BloodGroup bloodGroup) {
		this.bloodGroup = bloodGroup;
	}
	
	public int getBloodGroupCode() {
		return bloodGroup.getCode();
	}	
	
	public void setBloodGroup(BloodGroup arg) {
		this.bloodGroup = arg;
	}
}
```

* 이제 Person 클래스 사용 부분을 수정한다.
* static 변수 참조 부분은 전부 다음과 같이 수정해야 한다.

```java
Person thePerson = new Person(Person.A); 	//  수정 전
Person thePerson = new Person(BloodGroup.A); // 수정 후
```

* 읽기 메서드 호출 부분은 다음과 같이 수정한다.

```java
thePerson.getBloodGroupCode();		//수정 전
thePerson.getBloodGroup().getCode();//수정 후
```

* 쓰기 메서드도 똑같다.

```java
thePerson.setBloodGroup(Person.AB);
thePerson.setBloodGroup(BloodGroup.AB);
```

* Person 클래스 사용 부분을 전부 수정했으면 읽기, 생성자, static 변수, 정수 타입을 사용하는 쓰기 메서드를 삭제한다.
* 그리고 BloodGroup 클래스에 있는 숫자형 부호 사용 메서드를 private 타입으로 만든다.

```java
class Person {
	
	private BloodGroup bloodGroup;
	
	public BloodGroup getBloodGroup() {
		return bloodGroup;
	}
	
	public Person (BloodGroup bloodGroup) {
		this.bloodGroup = bloodGroup;
	}
	
	public void setBloodGroup(BloodGroup arg) {
		this.bloodGroup = arg;
	}
}
```

```java
public class BloodGroup {
	public static final BloodGroup O = new BloodGroup(0);
	public static final BloodGroup A = new BloodGroup(1);
	public static final BloodGroup B = new BloodGroup(2);
	public static final BloodGroup AB = new BloodGroup(3);
	
	private final int code;
	
	private BloodGroup(int code) {
		this.code = code;
	}
	
	private int getCode() {
		return this.code;
	}
	
	private static BloodGroup code(int arg) {
		return values(arg);
	}
}
```



### 14. 분류 부호를 하위클래스로 전환

* 클래스 기능에 영향을 주는 변경불가 분류 부호가 있을 땐 분류 부호를 하위클래스로 만들자.



#### 14.1 동기

* 클래스 기능에 영향을 주지 않는 분류 부호가 있을 땐 분류 부호를 클래스로 전환기법을 실시하면 된다.
* 그러나 분류 부호가 클래스 기능에 영향을 준다면 재정의를 통해 조금씩 다른 기능을 처리하는 것이 최선이다.
* 분류 부호가 클래스 기능에 영향을 미치는 현상은 조건문이 있을 때 주로 나타난다.
* 어느 조건문이든 분류 부호의 값을 검사해서 그 값에 따라 다른 코드를 실행한다.
* 이런 조건문은 조건문을 재정의로 전환을 실시해서 재정의로 바꿔야 한다.
* 이 기법이 효과를 보려면 분류 부호를 다형화된 기능이 든 상속 구조로 고쳐야 한다.
* 그런 상속 구조는 하나의 클래스와 각 분류 부호에 대한 하위클래스로 구성된다.
* 제일 쉬운 방법은 분류 후호를 하위 클래스로 전환을 실시하는 것
* 분류 부호가 든 클래스 코드를 이용해 각 분류 부호별 하위클래스를 작성하자.
* 단, 기법을 적용할 수 없는 몇 가지 경우는 다음과 같다.

1. 분류 부호의 값이 객체 생성 후 변할 때
2. 다은 이유로 분류 부호를 이미 하위클래스로 만들었을 때
3. 분류 부호를 상태/전략 패턴으로 전환을 실시해야 할 때



#### 14.2 방법

* 분류 부호를 자체 캡슐화하자.
  * 분류 부호가 생성자 메서드로 전달될 땐 생성자 메서드를 팩토리 메서드로 바꿔야 한다.
* 각 분류 부호 값마다 그에 해당하는 하위클래스를 작성하자. 그 하위클래스 안에 관련 값을 반환하는 분류 부호 읽기 메서드를 재정의하자.
  * 반환 값은 return  같이 return 문으로 하드 코딩된다. 복잡해 보여도 모든 case 문을 수정할 때 까진 임시방편으로 취하는 불가피한 조치다.
* 상위 클래스의 분류 부호 필드를 삭제하자. 분류 부호 읽기 메서드와 쓰기 메서드를 abstract 타입으로 선언하자.



#### 14.3 예제

```java
public class Employee {
	private int type;
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	public Employee(int type) {
		this.type = type;
	}
}
```

* 우선 분류 부호에 필드 자체 캡슐화 기법을 적용하자.

```java
public class Employee {
	private int type;
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	public Employee(int type) {
		this.type = type;
	}
	
	int getType() {
		return type;
	}
}

```

* Employee 클래스의 생성자 메서드가 분류 부호를 매개변수로 받으니까, 그 생성자 메서드를 다음과 같이 팩토리 메서드로 바꿔야 한다.

```java
public class Employee {
	private int type;
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	int getType() {
		return type;
	}
	
	Employee create(int type) {
		return new Employee(type);
	}
	
	private Employee (int type) {
		this.type = type;
	}
}
```

* 먼저 분류 부호 ENGINEER 변수를 Engineer 하위 클래스로 만들자. 다음과 같이 하위클래스를 작성하고 ENGINEER 분류 부호에 해당하는 재정의 메서드를 작성하자.

```java
public class Engineer extends Employee{
	int getType() {
		return Employee.ENGINEER;
	}
}
```

* 적절한 객체를 생성하게 팩토리 메서드도 다음과 같이 작성한다.

```java
public class Employee {
	private int type;
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	int getType() {
		return type;
	}
	
	static Employee create(int type) {
		if(type == ENGINEER) {
			return new Engineer(type);
		}
		
		return new Employee(type);
	}
	
	private Employee (int type) {
		this.type = type;
	}
}
```

* 이런 식으로 나머지 분류 부호도 바꾼다. 
* (여기까지 작성했는데 Employee에서 생성자 오류가 나는데 일단 내맘대로 작성함)
* 그리고 Employee 클래스의 분류 부호 필드를 삭제하고 abstract 타입의 getType 메서드를 작성한다.

```java
public abstract class Employee {
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	abstract int getType();
	
	static Employee create(int type) {
		if(type == ENGINEER) {
			return new Engineer(type);
		} else if(type == SALESMAN) {
			return new Salesman(type);
		} else if(type == MANAGER) {
			return new Manager(type);
		} else {
			throw new IllegalArgumentException("분류 부호 값이 잘못됨");
		}
	}
}
```

* (책에서는 switch를 했는데 나는 그냥 if로 함)
* 이거 이후에 메서드 하향과 필드 하향을 실시한다.



### 15. 분류 부호를 상태/전략 패턴으로 전환

* 분류 부호가 클래스의 기능에 영향을 주지만 하위클래스로 전환할 수 없을 땐 그 분류 부호를 상태 객체로 만들자.



#### 15.1 동기

* 하위클래스로 전환과 비슷하지만, 분류 부호가 객체 수명주기 동안 변할 때나 다른 이유로 하위클래스로 만들 수 없을 때 사용한다.
* 이 기법은 상태 or 전략 패턴 중 하나를 사용한다.
* 조건문을 재정의로 전환으로 하나의 알고리즘을 단순화해야 할 때는 전략이 더 적절
* 상태별 데이터를 이동하고 객체를 변화하는 상태로 생각할 때는 상태 패턴이 더 적절



#### 15.2 방법

* 분류 부호를 자체 캡슐화하자.
* 분류 부호의 목적을 나타내는 이름으로 새 클래스를 작성하자. 이것이 상태 객체이다.
* 그 상태 객체에 각 분류 부호에 해당하는 하위클래스를 추가하자
  * 하위클래스 추가는 한 번에 하나씩보단 한꺼번에 추가하는 것이 간편하다.
* 상태 객체 안에 분류 코드를 반환하는 abstract 메서드 호출을 작성하자. 올바른 분류 부호를 반환하는 상태 객체 하위클래스 각각에 대한 재정의 메서드 호출을 작성하자.
* 원본 클래스 안에 새 상태 객체를 나타내는 필드를 선언하자.
* 원본 클래스에 있는 분류 부호 판단 코드를 상태 객체에 위임하게 수정하자.
* 원본 클래스의 분류 부호 쓰기 메서드를 적절한 상태 객체 하위클래스의 인스턴스를 할당하게 수정하자.



#### 15.3 예제

* 아래 코드가 있다고 가정

```java
public class Employee {
	private int type;
	private int monthlySalary;
	private int commission;
	private int bonus;
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	public Employee(int type) {
		this.type = type;
	}
	
	int payAmout() {
		switch (type) {
			case ENGINEER:
				return monthlySalary;
			case SALESMAN:
				return monthlySalary + commission;
			case MANAGER:
				return monthlySalary + bonus;
			default:
				throw new RuntimeException("사원이 잘못됨");
		}
	}
}
```

* 분류 부호가 수시로 변하므로 하위 클래스로 만들 수 없다.
* 우선 분류 부호를 자체 캡슐화 한다.

```java
public class Employee {
	private int type;
	private int monthlySalary;
	private int commission;
	private int bonus;
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	public Employee(int type) {
		setType(type);
	}
	
	int getType() {
		return type;
	}
	
	void setType(int arg) {
		type = arg;
	}

	int payAmout() {
		switch (getType()) {
			case ENGINEER:
				return monthlySalary;
			case SALESMAN:
				return monthlySalary + commission;
			case MANAGER:
				return monthlySalary + bonus;
			default:
				throw new RuntimeException("사원이 잘못됨");
		}
	}
}
```

* 이제 상태 클래스를 선언하자.
* 상태 클래스를 abstract 타입으로 선언하고 그 안에 분류 부호를 반환하는 abstract 메서드를 다음과 같이 작성한다.

```java
public abstract class EmployeeType {
	abstract int getTypeCode();
}
```

* 분류 부호별 하위 클래스 작성

```java
public class Engineer extends EmployeeType{
	
	@Override
	int getTypeCode() {
		return Employee.ENGINEER;
	}
}
```

* 분류 부호 읽기/쓰기 메서드를 수정해서 하위클래스를 Employee 클래스에 실제로 연결한다.

```java
public class Employee {
	private EmployeeType type;
	private int monthlySalary;
	private int commission;
	private int bonus;
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	public Employee(int type) {
		setType(type);
	}
	
	int getType() {
		return type.getTypeCode();
	}
	
	void setType(int arg) {
		switch(arg) {
			case ENGINEER:
				type = new Engineer();
			case SALESMAN:
				type = new Salesman();
			case MANAGER:
				type = new Manager();
			default:
				throw new RuntimeException("사원 부호가 잘못됨");
		}
	}

	int payAmout() {
		switch (getType()) {
			case ENGINEER:
				return monthlySalary;
			case SALESMAN:
				return monthlySalary + commission;
			case MANAGER:
				return monthlySalary + bonus;
			default:
				throw new RuntimeException("사원이 잘못됨");
		}
	}
}
```

* 생성자를 팩토리 메서드로 전환 기법을 사용할 수 있다.
* 조건문을 재정의로 전환기법을 재빨리 사용할 수도 있다.
* 이제 우선 분류 부호 정의를 EmployeeType으로 복사하고, EmployeeType에 대한 팩토리 메서드를 작성한 후, Employee 클래스의 쓰기 메서드를 수정하자.
* 그리고 Employee 클래스에서 분류 부호 정의를 삭제하고 EmployeeType 클래스 참조를 넣는다.

```java
public class Employee {
	private EmployeeType type;
	private int monthlySalary;
	private int commission;
	private int bonus;
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	public Employee(int type) {
		setType(type);
	}
	
	int getType() {
		return type.getTypeCode();
	}
	
	void setType(int arg) {
		type = EmployeeType.newType(arg);
	}

	int payAmout() {
		switch (getType()) {
			case ENGINEER:
				return monthlySalary;
			case SALESMAN:
				return monthlySalary + commission;
			case MANAGER:
				return monthlySalary + bonus;
			default:
				throw new RuntimeException("사원이 잘못됨");
		}
	}
}
```

```java
public abstract class EmployeeType {
	static final int ENGINEER = 0;
	static final int SALESMAN = 1;
	static final int MANAGER = 2;
	
	abstract int getTypeCode();

	public static EmployeeType newType(int code) {
		switch(code) {
			case ENGINEER:
				return new Engineer();
			case SALESMAN:
				return new Salesman();
			case MANAGER:
				return new Manager();
			default:
				throw new RuntimeException("사원 부호가 잘못됨");
		}
	}
}
```

* 이것으로 payAmount 메서드에 조건문을 재정의로 전환 기법을 적용할 수 있게 됐다.



### 16. 하위클래스를 필드로 전환

* 여러 하위클래스가 상수 데이터를 반환하는 메서드만 다를 땐 각 하위클래스의 메서드를 상위클래스 필드로 전환하고 하위클래스는 전부 삭제하자.



#### 16.1 동기

* 기능을 추가하거나 기능을 조금씩 달리할 하위클래스를 작성하자.
* 다형적인 기능의 한 형태는 상수 메서드다.
* 상수 메서드는 하드코딩된 값을 반환하는 메서드다.
* 상수 메서드는 읽기 메서드에 각기 다른 값을 반환하는 하위클래스에 넣으면 유용하다.
* 상위클래스 안에 읽기 메서드를 정의하고 그 읽기 메서드를 하위클래스에서 다양한 값으로 구현하자.
* 상수 메서드가 유용하기 해도, 하위클래스를 상수 메서드로만 구성한다고 해서 그만큼 효용성이 커지는 것은 아니다.
* 상위클래스 안에 필드를 넣고 그런 하위클래스는 완전히 삭제하면 된다.
* 그렇게 하면 하위클래스의 불필요한 복잡함도 사라진다.



#### 16.2 방법

* 하위클래스에 생성자를 팩토리 메서드로 전환을 적용하자.
* 하위클래스 참조 부분을 전부 상위클래스 참조로 수정하자.
* 상위클래스에 각 상수 메서드에 대한 final 타입의 필드를 선언하자.
* protected 타입의 상위클래스 생성자를 선언해서 필드를 초기화하자.
* 하위클래스 생성자를 추가하거나 수정해서 새 상위클래스 생성자를 호출하자.
* 상위클래스 안에 필드를 반환하는 각 상수 메서드를 구현하고 하위클래스의 메서드는 삭제하자.
* 하위클래스 메서드를 전부 삭제했으면 메서드 내용 직접 삽입 기법을 실시해서 생성자 메서드 내용을 상위클래스의 팩토리 메서드에 직접 넣자.
* 하위 클래스를 삭제하자.
* 생성자 메서드 내용을 다시 팩토리 메서드에 직접 넣고, 작업이 완료되면 각 하위클래스를 삭제하자.



#### 16.3 예제

```java
abstract class Person {
	abstract boolean isMale();
	abstract char getCode();
}
```

```java
public class Male extends Person{
	boolean isMale() {
		return true;
	}
	
	@Override
	char getCode() {
		return 'M';
	}
}
```

```java
public class Female extends Person{
	boolean isMale() {
		return false;
	}
	
	@Override
	char getCode() {
		return 'F';
	}
}
```

* Male 하위클래스와 Female 하위클래스는 하드코딩된 상수 메서드 반환만 다르다.
* 이렇게 기능이 충실하지 못한 하위클래스는 삭제하자.
* 우선 생성자를 팩토리 메서드로 전환을 실시해야 한다.
* 이 예제는 각 하위클래스마다 팩토리 메서드가 필요하다.

```java
abstract class Person {
	abstract boolean isMale();
	abstract char getCode();
	
	static Person createMale() {
		return new Male();
	}
	
	static Person createFemale() {
		return new Female();
	}
}
```

* 다음과 같은 형태의 호출 코드를 찾아서 바꾸자.

```java
Person kent = new Male(); // 바꾸기 전
Person Kent = Person.createFemale(); // 바꾼 후
```

* 호출 부분을 수정했으면 하위클래스 참조 코드를 전부 삭제해야 한다.
* 텍스트 검색 기능으로 하위클래스 참조 부분을 찾은 후 하위클래스들을 private 타입으로 바꿔서 적어도 패키지 외부에서 이 하위클래스들을 참조하는 부분이 없는지 확인하면 된다.
* 상위클래스에서 다음과 같이 각 상수 메서드별 필드를 선언하자.
* 상위클래스에 protected 타입의 생성자 메서드를 추가한다.
* 새로 작성한 생성자를 호출하는 생성자 메서드를 추가한다.
* 상위클래스에 읽기 메서드를 넣어 필드를 사용하게 만들고 하위클래스 메서드는 삭제하자.

```java
class Person {
	
	private final boolean isMale;
	private final char code;
	
	protected Person(boolean isMale, char code) {
		this.isMale = isMale;
		this.code = code;
	}
	
	boolean isMale() {
		return isMale;
	}
	
	char getCode() {
		return code;
	}
	
	static Person createMale() {
		return new Male();
	}
	
	static Person createFemale() {
		return new Female();
	}
}
```

```java
public class Male extends Person{
	Male() {
		super(true, 'M');
	}
}

public class Female extends Person{
	Female() {
		super(false, 'F');
	}
}
```

* 하위 클래스 내용을 모두 삭제했으면 메서드 내용 직접 삽입을 실시해서 하위클래스 생성자 내용을 상위 클래스 안에 넣는다.

```java
class Person {
	
	private final boolean isMale;
	private final char code;
	
	protected Person(boolean isMale, char code) {
		this.isMale = isMale;
		this.code = code;
	}
	
	boolean isMale() {
		return isMale;
	}
	
	char getCode() {
		return code;
	}
	
	static Person createMale() {
		return new Person(true, 'M');
	}
	
	static Person createFemale() {
		return new Person(false, 'F');
	}
}
```

* Male, Female 클래스는 삭제한다.