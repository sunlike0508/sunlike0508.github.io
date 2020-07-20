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

