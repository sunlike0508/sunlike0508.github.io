---
title:  "객체 간의 기능 이동"
excerpt: "객체 간의 기능 이동"
classes: wide
categories:
  - Refactoring
tags:
  - [Refactoring]
last_modified_at: 2020-07-17
---



# 객체 간의 기능 이동

* 기능을  넣을 적절한 위치를 찾는 문제는 메서드 이동과 필드 이동을 실시해서 기능을 옮기면 해결된다.

* 두 기법 모두 사용할 때는 필드 이동 먼저 한다.
* 클래스가 방대해지는 이유는 기능이 너무 많기 때문이다.
* 이럴 때는 클래스 추출을 실시해야 한다.
* 클래스 기능이 너무 적으면 클래스 내용 직접 삽입을 한다.
* 다른 클래스가 사용 중일때는 주로 대리 객체 은폐를 실시한다.
* 대리 클래스를 은폐하면 그 클래스를 소유한 클래스의 인터페이스가 계속 변경될 때도 간혹 있는데, 이럴 때는 과잉 중개 메서드 제거를 실시한다.
* 외래 클래스에 메서드 추가기법과 국소적 상속확장 클래스 사용 기법은 클래스의 원본 코드에 접근할 수 없는 상황에서 이 수정 불가능한 클래스로 기능을 이동해야 할 때만 실시한다.
* 메서드가 한 두개 뿐일 때는 외래 클래스에 메서드 추가기법을 실시한다.
* 메서드가 세 개 이상일 때는 국소적 상속확장 클래스 사용 기법을 실시한다.



### 1. 메서드 이동

* 메서드가 자신이 속한 클래스보다 다른 클래스의 기능을 더 많이 사용할 땐 그 메서드가 제일 많이 이용하는 클래스 안에서 비슷한 내용의 새 메서드를 작성하자.
* 기존 메서드는 간단한 대리 메서드로 전환하든지 아예 삭제하자.



#### 1.1 동기

* 클래스에 기능이 너무 많거나 클래스가 다른 클래스와 과하게 연동되어 의존성이 지나칠 때는 메서드를 옮기는 것이 좋다.



#### 1.2 방법

* 원본 클래스에 정의되어 있는 원본 메서드에 사용된 모든 기능을 검사하여 전부 옮겨야 할지 판단하자.
  * 옮길 메서드에만 사용되는 기능도 그 메서드와 함께 옮겨야 한다.
  * 그 기능이 다른 메서드에도 사용된다면 그 메서드도 함께 옮기는 것을 고려하자.
* 원본 클래스의 하위클래스와 상위클래스에서 그 메서드에 대한 다른 선언이 있는지 검사하자.
  * 다른 선언이 있다면 대상 클래스에도 재정의를 넣을 수 있을 때만 옮길 수 있을지는 모른다.
* 그 메서드를 대상 클래스 안에 선언하자.
  * 대상 클래스 안에 있을 때 더욱 어울리는 다른 이름으로 그 메서드를 정의해도 된다.
* 원본 메서드의 코드를 대상 메서드에 복사한 후, 대상 클래스 안에서 잘 돌아가게끔 수정하자.
  * 대상 메서드가 원본 메서드를 사용한다면 대상 메서드 안에서 원본 객체를 참조할 방법을 정해야 한다.
  * 대상 클래스에 원본객체를 참조하는 기능이 없다면 원본 객체 참조를 대상 메서드에 매개 변수로 전달하자.
  * 대상 메서드에 예외처리 코드가 들어 있다면 예외를 논리적으로 어느 클래스가 처리할지 정하자.
  * 원본 클래스에서 처리하기로 결정했다면 예외처리 코드는 내버려두자.
* 대상 클래스를 컴파일하자.
* 원본 객체에서 대상 객체를 참조할 방법을 정하자.
  * 대상 클래스를 참조하는 속성이나 메서드가 없고 대상 클래스를 참조하는 메서드를 쉽게 작성하기 힘들다면 원본 클래스 안에 대상 클래스를 저장할 수 있는 새 속성을 선언해야 한다. 
  * 이렇게 수정한 코드를 그대로 둬도 되지만, 리펙토링을 완료할 때가지 임시로 나뒀다가 나중에 삭제해도 된다.
* 원본 메서드를 위임 메서드로 전환하자.
* 원본 메서드를 삭제하든지, 아니면 위임 메서드로 사용하게 내버려두자.
  * 참조가 많을 땐 원본 메서드를 위임 메서드로 내버려두는 방법이 더 편하다.
* 원본 메서드를 삭제할 때는 기존의 참조를 전부 대상 메서드 참조로 수정하자.
  * 각 참조를 하나씩 수정해서 테스트하는 것보다  모든 참조를 한꺼번에 수정하는 것이 편하다.



#### 1.3 예제

* 몇 가지 새 계좌 유형을 추가할 예정
* 각 계좌 유형마다 당좌대월 금액을 계산하는 공식이 다르다고 가정.
* 그래서 overdraftCharge 메서드를 AccountType 클래스로 옮기려 한다.

```java
public class Account {
	
	private AccountType _type;
	private int _daysOverdrawn;

	double overdraftCharge() {
		if(_type.isPremium()) {
			double result = 10;
			
			if(_daysOverdrawn > 7) {
				result += (_daysOverdrawn - 7) * 0.85;
			}
			
			return result;
		} else {
			return _daysOverdrawn * 1.75;
		}
	}
	
	double bankCharge() {
		double result = 4.5;
		
		if(_daysOverdrawn > 0) {
			result += overdraftCharge();
		}
		
		return result;
	}
}
```

* _daysOverdrawn 필드는 Account 클래스에 그대로 둬야 한다.

* overdraftCharge 메서드를 AccountType 클래스로 복사한 후 적절히 수정해야 한다.

```java
public class AccountType {
	double overdraftCharge(int daysOverdrawn) {
		if(isPremium()) {
			double result = 10;
			
			if(daysOverdrawn > 7) {
				result += (daysOverdrawn - 7) * 0.85;
			}
			
			return result;
		} else {
			return daysOverdrawn * 1.75;
		}
	}
}
```

* 이때 적절한 수정이란, AccoutType 클래스 기능의 사용에서 _type을 삭제하고 여전히 필요한 Account 클래스의 기능에 대해 뭔가를 수행하는 것이다.
* 원본 클래스의 기능을 사용하려면 다음 네 가지 작업 중 하나를 실시하면 된다.
  * 그 기능을 대상 클래스로도 옮기자.
  * 대상 클래스에서 원본 클래스로의 참조를 생성하거나 사용하자.
  * 원본 객체를 대상 객체에 매개변수로 전달하자.
  * 그 그능이 변수라면 그 변수를 매개변수로 전달하자.
* 위의 코드는 마지막 방법을 선택
* 나머지 호출 메서드를 전부 찾아서 AccountType 클래스에 들어 있는 메서드를 참조하게 수정한다.

```java
public class Account {
	
	private AccountType _type;
	private int _daysOverdrawn;

	double overdraftCharge() {
		return _type.overdraftCharge(_daysOverdrawn);
	}
	
	double bankCharge() {
		double result = 4.5;
		
		if(_daysOverdrawn > 0) {
			result += _type.overdraftCharge(_daysOverdrawn);
		}
		
		return result;
	}
}
```

* 위의 예제에서는 원본 메서드는 필드를 하나만 참조하고 있으니, 그 필드를 변수로 전달해도 된다.
* 만약 그 메서드가 Account 클래스의 다른 메서드를 호출한다면 그렇게 변수로 전달할 수 없으니, 그럴 땐 다음과 같이 원본 객체를 전달해야 한다.

```java
public class AccountType {
	double overdraftCharge(Account account) {
		if(isPremium()) {
			double result = 10;
			
			if(account.getDaysOverdrawn > 7) {
				result += (account.getDaysOverdrawn - 7) * 0.85;
			}
			
			return result;
		} else {
			return account.getDaysOverdrawn * 1.75;
		}
	}
}
```



### 2. 필드 이동

* 어떤 필드가 자신이 속한 클래스보다 다른 클래스에서 더 많이 사용될 때는 대상 클래스 안에 새 필드를 선언하고 그 필드 참조 부분을 전부 새 필드 참조로 수정하자.



#### 2.1 동기

* 어떤 필드가 자신이 속한 클래스보다 다른 클래스에 있는 메서드를 더 많이 참조해서 정보를 이용한다면 그 필드를 옮기는 것을 생각해보자.
* 클래스 추출을 실시하는 중에도 필드를 옮기는 기법이 사용된다.
* 이럴 때는 필드가 우선이고 메서드가 다음이다.



#### 2.2 방법

* 필드가 public이면 필드 캡슐화 기법을 실시하자.
  * 필드에 자주 접근하는 메서드를 옮기게 될 가능성이 높거나 그 필드에 많은 메서드가 접근할 때는 필드 자체 캡슐화를 실시하자.
* 대상 클래스 안에 읽기/쓰기 메서드와 함께 필드를 작성하자.
* 원본 객체에서 대상 객체를 참조할 방법을 정하자.
  * 기존 필드나 메서드에 대상 클래스를 참조하는 기능이 있을 수도 있다.
  * 그런 기능의 메서드가 존재하지도 않고 간편히 작성할 수 없다면 원본 클래스에 대상 클래스를 저장할 수 있는 새 필드를 만들어야 한다.
* 원본 클래스에서 필드를 삭제하자.
* 원본 필드를 참조하는 모든 부분을 대상 클래스에 있는 적절한 메서드를 참조하게 수정하자.
  * 변수 접근 참조 부분은 대상 객체의 읽기 메서드 호출로 수정하고, 대입 참조 부분은 쓰기 메서드 호출로 수정하자.
  * 필드가 private가 아니면 원본 클래스의 모든 하위 클래스를 뒤져서 필드 참조 부분을 찾아내자.



#### 2.3 예제

##### 2.3.1 필드 캡슐화

```java
public class Account {
	
	private AccountType _type;
	private double _interestRate;
	
	double interestForAmount_days (double amount, int days) {
		return _interestRate * amount * days / 365;
	}
}
```

* rate 필드를 AccountType 클래스로 옮기려 한다.
* rate 필드를 참조하는 메서드가 몇 개 있는데, interestForAmount_days도 그 중 하나다.
* 그 다음, AccountType 클래스에 필드와 읽기/쓰기 메서들 작성하자.

```java
public class AccountType {
	private double _interestRate;
	
	void setInterestRate(double arg) {
		_interestRate = arg;
	}
	
	double getInterestRate() {
		return _interestRate;
	}
}
```

* 이제 Account 클래스 안의 메서드들을 AccountType 클래스를 사용하게끔 참조를 변경하고, Account 클래스에서 interestRate 필드를 삭제하자.
* 참조 변경이 실제로 이뤄지려면 interestRate 필드를 삭제해야 하기 때문이다.
* 이렇게 하면 컴파일러가 참조 변경에 실패한 메서드를 쉽게 찾아낼 수 있다.

```java
public class Account {
	
	private AccountType _type;
	
	double interestForAmount_days (double amount, int days) {
		return _type.getInterestRate() * amount * days / 365;
	}
	
}
```



##### 2.3.2 필드 자체 캡슐화

* 많은 메서드가 interestRate 필드를 사용한다면 다음과 같이 필드 자체 캡슐화를 실시해야 한다.

```java
public class Account {
	
	private AccountType _type;
	
	double interestForAmount_days (double amount, int days) {
		return getInterestRate() * amount * days / 365;
	}
	
	private void setInterestRate(double arg) {
		_type.setInterestRate(arg);
	}

	private double getInterestRate() {
		return _type.getInterestRate();
	}
}
```



### 3. 클래스 추출

* 두 클래스가 처리해야 할 기능이 하나의 클래스 들어 있을 땐 새 클래스를 만들고 기존 클래스의 관련 필드와 메서드를 새 클래스로 옮기자.



#### 3.1 동기

* 클래스는 확실하게 추상화되어야 하며 두 세가지 명확한 기능을 담당해야 한다.
* 시간이 지날 수록 클래스는 방대해진다.
* 클래스의 어느 부분을 분리할지 궁리해야 한다.



#### 3.2. 방법

* 클래스의 기능 분리 방법을 정하자.
* 분리한 기능을 넣을 새 클래스를 작성하자.
  * 원본 클래스의 기능이 이름과 어울리지 않게 바뀌었으면 원본 클래스의 이름을 변경하자.
* 원본 클래스에서 새 클래스로의 링크를 만들자.
  * 양방향 링크를 만들어야 할 수도 있는데, 필요할 때까진 양방향 링크를 만들지 말자.
* 옮길 필드마다 필드 이동을 적용하자.
* 필드를 하나씩 옮길 때마다 컴파일과 테스트를 실시하자.
* 메서드 이동을 실시해서 원본 클래스의 메서드를 새 클래스로 옮기자. 하급 메서드(피호출 메서드)부터 시작해서 상급 메서드에 적용하자.
* 메서드 이동을 실시할 때마다 테스트를 실시하자.
* 각 클래스를 다시 검사해서 인터페이스를 줄이자
  * 양방향 링크가 있다면 그것을 단방향으로 바꿀 수 있을지 알아보자.
* 여러 곳에서 클래스에 접근할 수 있게 할지 결정하자. 여러곳에서 접근할 수 있게 할 경우, 새 클래스를 참조 객체나 변경불가 값 객체로서 공개할지 여부를 결정하자.



#### 3.3 예제

```java
public class Person {
	private String _name;
	private String _officeAreaCode;
	private String _officeNumber;
	
	public String getName() {
		return _name;
	}
	public String getOfficeAreaCode() {
		return _officeAreaCode;
	}
	public void setOfficeAreaCode(String arg) {
		this._officeAreaCode = arg;
	}
    public String getOfficeNumber() {
		return _officeNumber;
	}
	public void setOfficeNumber(String arg) {
		this._officeNumber = arg;
	}
}
```

* 전화번호 기능을 하나의 클래스로 만들 수 있다.

* 그리고 필드 이동을 하자. (책에서는 필드 하나씩 하는데 그냥 최종본만 아래 보여줌)

```java
public class Person {
	private String _name;
	private TelephoneNumber _officeTelephone = new TelephoneNumber();

	public String getTelephoneNumber() {
		return ("(" + getOfficeAreaCode() + ") " + getOfficeNumber());
	}
	
	public String getName() {
		return _name;
	}
	
	public String getOfficeNumber() {
		return _officeTelephone.getOfficeNumber();
	}
	
	public void setOfficeNumber(String arg) {
		_officeTelephone.setOfficeNumber(arg);
	}
	
	String getOfficeAreaCode() {
		return _officeTelephone.getAreaCode();
	}
	
	void setOfficeAreaCode(String arg) {
		_officeTelephone.setAreaCode(arg);
	}
}
```

```java
public class TelephoneNumber {
	private String _areaCode;
	private String _officeNumber;
	
	String getAreaCode() {
		return _areaCode;
	}
	
	void setAreaCode(String arg) {
		_areaCode = arg;
	}
	
	String getOfficeNumber() {
		return _officeNumber;
	}
	
	void setOfficeNumber(String arg) {
		_officeNumber = arg;
	}
}
```

* 새 클래스를 클라이언트에 어느 정도 공개할지 결정하자.
* 인터페이스에 위임 메서드를 작성해서 새 클래스를 완전히 감추든지, 아니면 새 클래스를 공개할 수 도 있다.
* 새 클래스를 공개하는 방식은 왜곡의 위험을 고려해야 한다.
* 만약 TelephoneNumber 클래스를 공개했는데 클라이언트가 이 객체 안의 지역번호를 변경한다면?
* 변경한 주체가 직접호출한 클라이언트가 아니라 클라이언트의 클라이언트일 수도 있다.
* 다음 방법 중 하나를 택해야 한다.
  * 모든 객체가 TelephoneNumber 클래스의 어느 부분이든 변경할 수 있음을 받아들인다.
  * 이렇게 하면 TelephoneNumber 클래스가 참조 객체가 되므로 값을 참조로 전환을 실시해야 한다.
  * 이때 Person 클래스가 TelephoneNumber 클래스의 접근 지점이 된다.
  * 어느 주체든 Person 클래스를 거치지 않고는 TelephoneNumber 클래스의 값을 변경하지 못하게 한다.
  * TelephoneNumber 클래스를 변경불가로 만들어야 한다.
  * TelephoneNumber 클래스를 외부로 전달하기 전에 복사한 후 변경불가로 만든다.
  * 하지만 이 방법은 코드를 보는 이들이 값을 변경할 수 있다는 착각을 불러일으킨다.
  * 게나다 전화번호가 여기저기로 무수히 전달될 경우 클라이언트 간에 왜곡 문제가 발생할 수도 있다.
* 클래스 추출은 두 결과 클래스에 따로 락을 걸 수 있어서 병렬 실행 프로그램의 생동감을 향상시키는 용도로 흔히 사용되는 기법이다.
* 두 객체에 락을 걸 필요가 없다면 이 기법을 실시하지 않아도 된다.
* 그러나 이 방법도 위험성은 있다.
* 두 객체에 동시에 락을 걸어야 할 경우는 트랜잭션과 다른 종류의 공유 락 영역으로 넘어가게 된다.
* 이것은 복잡한 영역이며 대부분 얻는 것에 비해 많은 작업을 요한다.
* 트랜잭션은 사용하기엔 좋지만, 트랜잭션 관리자를 작성하는 일은 웬만한 프로그래머가 하기에 부적합한 일이다.



### 4. 클래스 내용 직접 삽입

* 클래스 기능이 너무 적을 땐 그 클래스의 모든 기능을 다른 클래스로 합쳐 놓고 원래의 클래는 삭제하자.



#### 4.1 동기

* 클래스 내용 직접 삽입은 클래스 추출과 반대다.
* 이런 상황은 주로 리펙토링으로 인해 남은 기능이 없어져 나타난다.



#### 4.2 방법

* 원본 클래스의 public 프로토콜 메서드를 합칠 클래스에 선언하고, 이 메서드를 전부 원본 클래스에 위임하자.
  * 원본 클래스의 메서드 대신 별도의 인터페이스가 알맞다고 판단되면 클래스 내용 직접 삽입을 실시하기 전에 인터페이스 추출 기법을 실시하자.
* 원본 클래스의 모든 참조를 합칠 클래스 참조로 수정하자.
  * 원본 클래스를 private로 선언하고 패키지 밖의 참조를 삭제하자.
  * 컴파일러가 껍데기만 남은 원본 클래스 참조를 찾아낼 수 있게 원본 클래스명을 변경하자.
* 메서드 이동과 필드 이동을 실시해서 원본 클래스의 모든 기능을 합칠 클래스로 차례로 옮기자.
* 원본 클래스를 삭제하자.



#### 4.3 예제

* 클래스 추출과 반대로 보면 된다.



### 5. 대리 객체 은폐

* 클라이언트가 객체의 대리 클래스를 호출할 땐 대리 클래스를 감추는 메서드를 서버에 작성하자.



#### 5.1 동기

* 객체에서 핵심 개념 중 하나는 캡슐화다.
* 캡슐화란 객체가 시스템의 다른 부분에 대한 정보의 일부만 알 수 있게 은폐하는 것을 뜻한다.
* 객체를 캡슐화하면 무언가를 변경할 때 그 변화를 전달해야 할 객체가 줄어들어 변경하기 쉬워진다.
* 클라이언트가 서버 객체의 필드 중 하나에 정의된 메서드를 호출할 때 그 클라이언트는 이 대리 객체를 알아야 한다.
* 대리 객체가 변경될 때 클라이언트도 변경해야 할 가능성이 있기 때문이다.
* 이런 의존성을 없애려면, 다음 그림처럼 대리 객체를 감추는 간단한 위임 메서드를 서버에 두면 된다.
* 변경은 서버에만 이뤄지고 클라이언트에는 영향을 주지 않는다.

* 간단한 위임을 다이어그램으로 나타내면 다음과 같다.

![]({{site.url}}/assets/images/ref19.PNG)

* 서버의 일부 클라이언트나 모든 클라이언트에 대리 객체 은폐를 실시하는 것이 좋을 때도 있다.
* 모든 클라이언트를 대상으로 대리 객체를 감출 경우에는 서버의 인터페이스에서 대리 객체에 대한 모든 부분을 삭제해도 된다.



#### 5.2 방법

* 대리 객체에 들어 있는 각 메서드를 대상으로 서버에 간단한 위임 메서드를 작성하자.
* 클라이언트를 수정해서 서버를 호출하게 만들자
  * 클라이언트 클래스가 서버 클래스와 같은 패키지에 들어 있지 않다면 대리 메서드의 접근을 같은 패키지에 든 클래스만 접근할 수 있게 수정하는 것을 고려하자.
* 각 메서드를 수정할 때마다 컴파일과 테스트를 한다.
* 대리 객체를 읽고 써야 할 클라이언트가 하나도 남지 않게 되면, 서버에서 대리 객체가 사용하는 읽기/쓰기 메서드를 삭제하자.



#### 5.3 예제

```java
class Person {
	Department department;

	public Department getDepartment() {
		return department;
	}

	public void setDepartment(Department arg) {
		this.department = arg;
	}
}

class Department {
	private String chargeCode;
	private Person manager;
	
	public String getChargeCode() {
		return chargeCode;
	}
	public Person getManager() {
		return manager;
	}
	public void setChargeCode(String chargeCode) {
		this.chargeCode = chargeCode;
	}
	public void setManager(Person manager) {
		this.manager = manager;
	}
}
```

* 클라이언트 클래스는 어떤 사람의 팀장이 누군지 알아내려면 먼저 부서를 알아내야 한다.

```java
manager = john.getDepartment().getManager();
```

* 위의 코드를 보면 Department 클래스의 원리와,  Department 클래스의 기능이 팀장 정보를 지속적으로 알아내는 것을 알 수 있다.

* 이런 의존성을 줄이려면 Department 클래스를 클라이언트가 알 수 없게 감춰야 한다.
* 그러려면 Person 클래스에 다음과 같이 간단한 위임 메서드를 작성하면 된다.

```java
public Person getManager() {
	return department.getManager();
}
```

* 이제 다음과 같이 새 메서드를 사용하게 끔 Person 클래스의 모든 참조 부분을 수정해야 한다.

```java
manager = john.getDepartment().getManager();
```

* Department 클래스의 모든 메서드와 Person 클래스의 모든 클라이언트를 수정했으면 Person 클래스에 들엉 있는 getDepartment 읽기 메서드를 삭제하자.

```java
class Person {
	Department department;

	public void setDepartment(Department arg) {
		this.department = arg;
	}
	
	public Person getManager() {
		return department.getManager();
	}
}

class Department {
	private String chargeCode;
	private Person manager;
	
	public String getChargeCode() {
		return chargeCode;
	}
	public Person getManager() {
		return manager;
	}
	public void setChargeCode(String chargeCode) {
		this.chargeCode = chargeCode;
	}
	public void setManager(Person manager) {
		this.manager = manager;
	}
}
```



### 6. 과잉 중개 메서드 제거

* 클래스에 자잘한 위임이 너무 많을 땐 대리 객체를 클라이언트가 직접 호출하게 하자.



#### 6.1 동기

* 대리 객체  사용을 캡슐화하면 단점도 있다.
* 클라이언트가 대리 개체의 새 기능을 사용해야 할 때마다 서버에 간단한 위임 메서드를 추가해야 한다는 점이다.
* 서버 클래스는 그저 중개자에 불과하므로, 이때는 클라이언트가 대리 객체를 직접 호출하게 해야 한다.
* 은폐의 적절한 정도를 알기란 어렵다.
* 시스템이 변경되면 은폐 정도 기준도 변한다.
* 필요할 때마다 리펙토링으로 수정하면 된다.



#### 6.2 방법

* 대리 객체에 대한 접근 메서드를 작성하자
* 대리 메서드를 클라이언트가 사용할 때마다 서버에서 메서드를 제거하고 클라이언트에서의 호출을 대리 객체에서의 메서드 호출로 교체하자.
* 메서드를 수정할 때마다 테스트를 실시하자.



#### 6.3 에제

* 대리 객체 은폐의 예제랑 반대다.
* getManager 메소드같은 기능이 많다면 이런 자잘한 위임이 Person 객체에 너무 많아진다.
* 이럴 때 과잉 중개 제거를 실시한다.



### 7. 외래 클래스에 메서드 추가

* 사용 중인 서버 클래스에 메서드를 추가해야 하는데 그 클래스를 수정할 수 없을 땐 클라이언트 클래스 안에 서버 클래스의 인스턴스를 첫 번째 인자로 받는 메서드를 작성하자.



#### 7.1 동기

* 원하는 기능을 추가할 때 원본 클래스가 수정 가능하다면 추가하면 된다.
* 그러나 수정할 수 없다면 클라이언트 클래스 안에 작성해야 한다.
* 한 번정도는 괜찮지만 여러 번 사용한다면 이 코드를 여기 저기 중복되게 작성해야 할 것이다.
* 이 리팩토링 기법을 실시할 때 새로 만들 메서드를 외래 메서드로 만들면 그 메서드가 원래는 원본 메서드인 서버 메서드에 있어야 할 메서드임을 분명히 나타낼 수 있다.
* 서버 클래스에 수많은 외래 메서드를 작성해야 하거나 하나의 외래 메서드를 여러 클래스가 사용해야 할 때는 이 기법 대신 국소적 상속확장 사용 기법을 실시해야 한다.
* 외래 메서드는 임시방편에 불과하다.
* 가능하면 원래 있어야할 위치로 옮긴다.
* 코드 저작권 문제라면 서버 클래스 제작자에게 구현해 달라 요청하자.



#### 7.2 방법

* 필요한 기능의 메서드를 클라이언트 클래스 안에 작성하자
  * 그 메서드는 클라이언트 클래스의 어느 기능에도 접근해선 안 된다.
  * 그 메서드에 값이 필요할 땐 매개변수로 전달해야 한다.
* 서버 클래스의 인스턴스를 첫 번째 매개변수로 만들자.
* 그 메서드에 '서버 클래스에 있을 외래 메서드' 같은 주석을 달자
  * 이렇게 하면 나중에 그 외래 메서드를 옮길 일이 생겼을 때 문자열 검색 기능으로 외래 메서드를 쉽게 찾을 수 있다.



#### 7.3 예제

```java
Date newStart = new Date (previousEnd.getYear(), previousEnd.getMonth(), previousEnd.getDate() + 1);
```

* 리팩토링 후

```java
Date newStart = nextDay(previousEnd);

private Date nextDay(Date arg) {
	return new Date (arg.getYear(), arg.getMonth(), arg.getDate() + 1);
}
```



### 8. 국소적 상속확장 클래스 사용

* 사용 중인 서버 클래스에 여러 개의 메서드를 추가해야 하는데 클래스를 수정할 수 없을 땐 새 클래스를 작성하고 그 안에 필요한 여러 개의 메서드를 작성하자.
* 이 상속확장 클래스를 원본 클래스의 하위클래스나 래퍼 클래스로 만들자.



#### 8.1 동기

* 클래스 제작자도 신이 아니므로 개발자에게 필요한 메서드가 전부 든 클래스를 만드는 건 불가능하다.
* 원본 클래스에 수정할 수 있어서 필요한 메서드를 추가하는 것이 최선의 방법이다. 그러나 대부분 불가능하다.
* 필요한 메서드가 한 두개면 외래 클래스에 메서드 추가기법을 실시하면 된다.
* 하지만 세 개 이상이면 하위클래스화와 래퍼화를 실시한다. 이들을 국소적 상속확장 클래스라고 부른다.
* 국소적 상속확장 클래스는 별도의 클래스지만 상속확장하는 클래스의 하위 타입이다.
* 따라서 국소적 상속확장 클래스는 원본 클래스의 모든 기능도 사용 가능하면서 추가 기능도 들어 있다.
* 원본 클래스를 사용할 것이 아니라 국소적 상속확장 클래스를 인스턴스화해서 사용하자.
* 국소적 상속확장 클래스를 사용하면 메서드와 데이터가 체계적인 단위로 묶여야 한다는 원칙이 지켜진다.
* 필요한 메서드가 엉뚱한 클래스에 쌓이게 되면 결국 복잡해져서 재사용하기 힘들어진다.
* 나는 하위클래스와 래퍼 클래스 중에서 작업량이 적은 하위클래스를 선호한다.
* 하위클래스로 만드는 방식의 문제점은 객체를 생성함과 동시에 하위클래스로 만들어야 한다는 점이다.
* 하위클래스로 만들려면 그 하위클래스의 새 객체를 생성해야 한다.
* 다른 객체가 기존 클래스를 참조한다면 원본 클래스의 데이터가 든 객체가 두 개임을 알 수 있다.
* 원본 객체가 변경불가 상태이면 안심하고 복사하면 되니 문제가 없다.
* 그러나 원본 객체가 변경할 수 있는 상태이면 한 객체를 수정해도 다른 객체가 수정되지 않아 문제가 된다.
* 따라서 래퍼 클래스를 사용해야 한다.
* 그러면 국소적 상속확장 클래스를 통해 이뤄진 수정이 원본 객체에도 반영되고, 그 반대로도 반영된다.



#### 8.2 방법

* 상속확장 클래스를 작성한 후 원본 클래스의 하위클래스나 래퍼 클래스로 만들자.
* 상속확장 클래스에 변환 생성자 메서드를 작성하자.
  * 생성자는 원본 클래스를 인자로 받는다.
  * 하위클래스의 경우 적절한 상위클래스 생성자를 호출하며, 래퍼 클래스의 경우 대리 필드에 그 인자를 할당한다.
* 상속확장 클래스에 새 기능을 추가하자.
* 필요한 위치마다 원본 클래스를 상속확장 클래스로 수정하자.
* 이 클래스용으로 정의된 외래 메서드를 전부 상속확장 클래스로 옮기자.



#### 8.3 예제

* 하위 클래스화 방식

```java
class MfDateSub extends Date{
	public MfDateSub nextDay() ...
	public int dayOfYear() ...
}
```

* 래퍼 클래스는 다음과 같이 위임을 이용

```java
class MfDateWrap {
	private Date original;
}
```



##### 8.3.1 하위클래스 사용

* 먼저 다음과 같이 새 클래스를 원본 Date 클래스의 하위클래스로 작성하자.

```java
class MfDateSub extends Date {...}
```

* 다음으로 Date 클래스와 상속확장 클래스의 호출 관계를 처리하자.
* 원본 Date 클래스의 생성자는 간단한 위임을 통해 다음과 같이 반복시켜야 한다.

```java
public MfDateSub(String dateString) {
	super(dateString)
}
```

* 이제 원본 Date 클래스를 인자로 받는 변환 생성자를 추가하자.

```java
public MfDateSub(Date arg) {
	super(arg.getTime());
}
```

* 이제 그 상속확장 클래스에 새 기능을 추가하고, 메서드 이동 기법을 실시해서 외래 메서드를 전부 상속확장 클래스로 옮기면 된다.

```java
client class ...

private static Date nextDay(Date arg) {
	return new Date (arg.getYear(), arg.getMonth(), arg.getDate() + 1);
}
```

* 그러면 다음과 같이 된다.

```java
client MfDateSub ...

    Date nextDay() {
        return new Date (getYear(), getMonth(), getDate() + 1);
    }
```



##### 8.3.2 래퍼 클래스 사용

* 다음과 같이 래퍼 클래스를 선언하자.

```java
class MfDateWrap {
	private Date original;
}
```

* 래퍼 클래스로 만들 땐, 생성자 구성 방식을 달리해야 한다.
* 원본 생성자 메서드는 다음과 같이 간단히 위임한다.

```java
public MfDateWrap(String dateString) {
    original = new Date(dateString);
}
```

* 변환 생성자는 이제 단순히 인스턴스 변수에 값을 할당하는 기능만 수행한다.

```java
public MfDateWrap(Date arg) {
    original = arg;
}
```

* 이제 원본 Date 클래스의 모든 메서드를 위임하는 지루한 작업을 해야 한다.
* 두 개의 메서드만 보여주면 다음과 같다.

```java
public int getYear() {
    return original.getYear();
}

public boolean equals(Object arg) {
    if(this == arg) {
        return true;
    }

    if(!(arg instanceof MfDateWrap)) {
        return false;
    }

    MfDateWrap other = ((MfDateWrap) arg);

    return original.equals(other.original);
}
```

* 이제 메서드 이동기법을 실시해서  날짜 전용 기능을 다음과 같이 새 클래스에 넣자.

```java
// 클라이언트 클래스
// 원래는 Date 클래스에 있어야 할 외래 메서드
private static Date nextDay(Date arg) {
	return new Date (arg.getYear(), arg.getMonth(), arg.getDate() + 1);
}
```

* 그러면 앞의 코드가 다음과 같이 바뀐다.

```java
class MfDate ...
    Date nextDay() {
        return new Date (getYear(), getMonth(), getDate() + 1);
    }
```

* 래퍼 클래스화 방식을 사용할 때는 다음과 같이 원본 클래스를 인자로 받는 메서드들을 처리하는 방법이 문제 된다.

```java
public boolean after(Date arg)
```

* 원본 클래스를 변경할 수 없으므로 다음과 같이 after 메서드 실행이 한 방향으로만 가능하다.

```java
aWrapper.after(aDate) // 제대로 돌아가게 할 수 있음
aWrapper.after(anotherWrapper) // 제대로 돌아가게 할 수 있음
aDate.after(aWrapper) // 돌아가지 않음
```

* 이런 식으로 재정의하는 이유는 래퍼 클래스 사용 사실을 원본 클래스 사용 부분이 모르게 하기 위해서다.
* 래퍼 클래스 사용 부분은 래퍼 클래스에 전혀 관여해선 안 되며 원본과 래퍼를 동등하게 다룰 수 있어야 하므로 이 방식이 좋다.
* 하지만 래퍼 클래스 사용 사실을 완벽히 감추진 못한다.
* 문제는 equals 같은 특정 시스템 메서드에 있다.
* 이론을 토대로 개발자는 MfDateWrap 클래스의 equals 메서드를 다음과 같이 재정의할 수 있다고 생각할 것이다.

```java
public boolean equals (Date arg) //문제 발생
```

* 앞의 코드처럼 재정의하는 것은 위험하다.
* 왜냐하면 이 방식은 의도한 기능을 수행하게 할 수 있을진 몰라도 기본 자바에서는 대칭적이라고 전제하는데 이 전제에 어긋나는 코딩을 하면 이상한 버그가 생긴다.
* 이것을 방지하는 유일한 방법은 Date 클래스를 수정하는 것인데 애시당초 수정할 수가 없다.
* 따라서 이런 상황에서는 어쩔 수 없이 래퍼 클래스 사용 사실을 공개할 수밖에 없다.
* 대등성 검사라는 목적을 나타내고자 메서드명을 다음과 같이 변경하자.

```java
public boolean equalsDate(Date arg)
```

* 앞의 equalsDate 메서드를 Date 클래스와 MfDateWrap 클래스에 모두 넣으면, 미지의 객체타입을 검사하지 않아도 된다.

```java
public boolean equalsDate(MfDateWrap arg)
```

* 기능을 재정의하지 않는 이상, 하위클래스화 방식을 사용할 땐 이런 문제가 없다.
* 그러나 기능을 재정의하면 메서드를 검색할 때 정말 애먹게 된다.
* 상속확장 클래스를 사용할 땐 메서드를 재정의하지 않고 그냥 메서드를 추가한다.