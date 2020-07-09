---
title:  "맛보기 예제"
excerpt: "맛보기 예제"
classes: wide
categories:
  - Refactoring
tags:
  - [Refactoring]
last_modified_at: 2020-07-07
---



# 맛보기 예제



### 1. 맛보기 프로그램

* 아래를 예제로 앞으로 설명한다.



![]({{site.url}}/assets/images/ref01.PNG)

![]({{site.url}}/assets/images/ref02.PNG)

* Movie Class

```java
public class Movie {
	public static final int CHILDRENS = 2;
	public static final int REGULAR = 0;
	public static final int NEW_RELEASE = 1;
	private String _title;
	private int _priceCode;
	
	public Movie(String title, int priceCode) {
		_title = title;
		_priceCode = priceCode;
	}
	
	public int getPriceCode() {
		return _priceCode;
	}

	public void setPriceCode(int _priceCode) {
		this._priceCode = _priceCode;
	}
	
	public String getTitle() {
		return _title;
	}
}
```

* Rental Class

```java
public class Rental {
	private Movie _movie;
	private int _daysRented;
	
	public Rental(Movie movie, int dayRented) {
		_movie = movie;
		_daysRented = dayRented;
	}

	public Movie getMovie() {
		return _movie;
	}

	public int getDaysRented() {
		return _daysRented;
	}
}
```

* Customer Class

```java
import java.util.Enumeration;
import java.util.Vector;

public class Customer {
	private String _name;
	private Vector<Rental> _rentals = new Vector<Rental>();
	
	public Customer(String name) {
		_name = name;
	}
	
	public void addRental(Rental arg) {
		_rentals.add(arg);
	}
	
	public String getName() {
		return _name;
	}
	
	public String statement() {
		double totalAmout = 0;
		int frequentRenterPoints = 0;
		Enumeration<Rental> rentals = _rentals.elements();
		String result = getName() + " 고객님의 대여 기록\n";
		
		while (rentals.hasMoreElements()) {
			double thisAmout = 0;
			Rental each = rentals.nextElement();
			
			//비디오 종류별 대여료 계산
			switch(each.getMovie().getPriceCode()) {
				case Movie.REGULAR:
					thisAmout += 2;
					
					if (each.getDaysRented() > 2) {
						thisAmout += (each.getDaysRented() - 2) * 1.5;
					}
					
					break;
				case Movie.NEW_RELEASE:
					thisAmout += each.getDaysRented() * 3;
					break;
				case Movie.CHILDRENS:
					thisAmout += 1.5;
					
					if (each.getDaysRented() > 3) {
						thisAmout += (each.getDaysRented() - 3) * 1.5;
					}
					
					break;
			}
			
			// 적립 포인트를 1 포인트 증가
			frequentRenterPoints++;
			// 최신물을 이틀 이상 대여하면 보너스 포인트 지급
			if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE && each.getDaysRented() >1)) {
				frequentRenterPoints++;
			}
			
			// 이번에 대여하는 비디오 정보와 대여료를 출력
			result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmout) + "\n";
			// 현재까지 누적된 총 대여료
			totalAmout += thisAmout;
		}
		
		result += "누적 대여료: " + String.valueOf(totalAmout) + "\n";
		result += "적립 포인트: " + String.valueOf(frequentRenterPoints);
		
		return result;
	}
}
```
* 위의 코드는 문제 없이 돌아간다.
* 그러나 차후에 웹에서 보일 수 있게 HTML 형식 출력을 원한다거나 대여료 규칙 수정, 비디오 분류를 바꾸고 싶을 때 등 요구 조건이 증가하면 statement 메서드를 복사해서 붙여야하고, 메소드가 증가하며 여러 메소드를 동시에 고쳐야 하는 문제들이 발생한다.

* 당장은 프로그램이 문제가 없을지 몰라도 나중에 사용자가 요구한 기능을 수장하기 힘들어서 애먹는다.
* 바로 이 상황이 리펙토링해야할 시점이다.



### 리펙토링 첫 단계

* 첫 번째는 테스트를 작성하는 것이다.



### statement 메서드 분해와 기능 재분배

* 메서드 추출 기법

* 아래는 switch 부분 메서드로 추출하고 추출한 메서드의 변수 명을 명확하게 수정

```java
public String statement() {
    double totalAmout = 0;
    int frequentRenterPoints = 0;
    Enumeration<Rental> rentals = _rentals.elements();
    String result = getName() + " 고객님의 대여 기록\n";

    while (rentals.hasMoreElements()) {
        double thisAmout = 0;
        Rental each = rentals.nextElement();

        thisAmout = amoutFor(each);

        // 적립 포인트를 1 포인트 증가
        frequentRenterPoints++;
        // 최신물을 이틀 이상 대여하면 보너스 포인트 지급
        if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE && each.getDaysRented() >1)) {
            frequentRenterPoints++;
        }

        // 이번에 대여하는 비디오 정보와 대여료를 출력
        result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmout) + "\n";
        // 현재까지 누적된 총 대여료
        totalAmout += thisAmout;
    }

    result += "누적 대여료: " + String.valueOf(totalAmout) + "\n";
    result += "적립 포인트: " + String.valueOf(frequentRenterPoints);

    return result;
}

private double amoutFor(Rental aRental) {
    double result = 0;
    //비디오 종류별 대여료 계산
    switch(aRental.getMovie().getPriceCode()) {
        case Movie.REGULAR:
            result += 2;

            if (aRental.getDaysRented() > 2) {
                result += (aRental.getDaysRented() - 2) * 1.5;
            }

            break;
        case Movie.NEW_RELEASE:
            result += aRental.getDaysRented() * 3;
            break;
        case Movie.CHILDRENS:
            result += 1.5;

            if (aRental.getDaysRented() > 3) {
                result += (aRental.getDaysRented() - 3) * 1.5;
            }

            break;
    }

    return result;
}
```



#### 대여료 계산 메서드 옮기기

* amoutFor 메서드를 Rental로 옮기고 메서드 명을 getCharge로 변경. 해당 클래스에서 값을 가져오기 때문에 파라미터 불필요.

```java
public class Rental {
	double getCharge() {
		double result = 0;
		//비디오 종류별 대여료 계산
		switch(getMovie().getPriceCode()) {
			case Movie.REGULAR:
				result += 2;
				
				if (getDaysRented() > 2) {
					result += (getDaysRented() - 2) * 1.5;
				}
				
				break;
			case Movie.NEW_RELEASE:
				result += getDaysRented() * 3;
				break;
			case Movie.CHILDRENS:
				result += 1.5;
				
				if (getDaysRented() > 3) {
					result += (getDaysRented() - 3) * 1.5;
				}
				
				break;
		}
		
		return result;
	}
}
```

* 변경된 Customer 클래스

```java
public class Customer {	
	public String statement() {
		double totalAmout = 0;
		int frequentRenterPoints = 0;
		Enumeration<Rental> rentals = _rentals.elements();
		String result = getName() + " 고객님의 대여 기록\n";
		
		while (rentals.hasMoreElements()) {
			double thisAmout = 0;
			Rental each = rentals.nextElement();
			
			thisAmout = each.getCharge();
			
			// 적립 포인트를 1 포인트 증가
			frequentRenterPoints++;
			// 최신물을 이틀 이상 대여하면 보너스 포인트 지급
			if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE && each.getDaysRented() >1)) {
				frequentRenterPoints++;
			}
			
			// 이번에 대여하는 비디오 정보와 대여료를 출력
			result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(thisAmout) + "\n";
			// 현재까지 누적된 총 대여료
			totalAmout += thisAmout;
		}
		
		result += "누적 대여료: " + String.valueOf(totalAmout) + "\n";
		result += "적립 포인트: " + String.valueOf(frequentRenterPoints);
		
		return result;
	}
}
```

* charge 메서드를 옮긴 후 클래스 관계

![]({{site.url}}/assets/images/ref03.PNG)



* thisAmout 변수의 불필요한 중복 처리. 메서드 호출로 전환 기법을 사용해서 thisAmout 변수를 삭제후 statement 메소드.

```java
public String statement() {
    double totalAmout = 0;
    int frequentRenterPoints = 0;
    Enumeration<Rental> rentals = _rentals.elements();
    String result = getName() + " 고객님의 대여 기록\n";

    while (rentals.hasMoreElements()) {
        Rental each = rentals.nextElement();
        // 적립 포인트를 1 포인트 증가
        frequentRenterPoints++;
        // 최신물을 이틀 이상 대여하면 보너스 포인트 지급
        if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE && each.getDaysRented() >1)) {
            frequentRenterPoints++;
        }

        // 이번에 대여하는 비디오 정보와 대여료를 출력
        result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(each.getCharge()) + "\n";
        // 현재까지 누적된 총 대여료
        totalAmout += each.getCharge();
    }

    result += "누적 대여료: " + String.valueOf(totalAmout) + "\n";
    result += "적립 포인트: " + String.valueOf(frequentRenterPoints);

    return result;
}
```



#### 적립 포인트 계산을 메서드로 빼기

* 대여 비디오 종류에 따라 적립 포인트 계산법이 달라진다. 따라서 처리 코드도 Rental 클래스에 넣는 것이 합리적.

* 적립 포인트 계산 코드 부분에 메서드 추출 기법을 적용

```java
public class Rental {
	public int getFrequentRenterPoints() {
		// 보통 영화는 포인트 1 지급, 최신물을 이틀 이상 대여하면 보너스 포인트 지급
		if ((getMovie().getPriceCode() == Movie.NEW_RELEASE && getDaysRented() >1)) {
			return 2;
		} else {
			return 1;
		}
	}
}
```

```java
public String statement() {
    double totalAmout = 0;
    int frequentRenterPoints = 0;
    Enumeration<Rental> rentals = _rentals.elements();
    String result = getName() + " 고객님의 대여 기록\n";

    while (rentals.hasMoreElements()) {
        Rental each = rentals.nextElement();

        // 경우에 따른 적립 포인트 지급 함수를 호출
        frequentRenterPoints += each.getFrequentRenterPoints();

        // 이번에 대여하는 비디오 정보와 대여료를 출력
        result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(each.getCharge()) + "\n";
        // 현재까지 누적된 총 대여료
        totalAmout += each.getCharge();
    }

    result += "누적 대여료: " + String.valueOf(totalAmout) + "\n";
    result += "적립 포인트: " + String.valueOf(frequentRenterPoints);

    return result;
}
```



* 적립 포인트 계산 코드를 메서드로 빼서 옮기기 전 클래스 관계

![]({{site.url}}/assets/images/ref04.PNG)

* 적립 포인트 계산 코드를 메서드로 빼서 옮기기 전 상호작용 다이어그램

![]({{site.url}}/assets/images/ref05.PNG)

* 적립 포인트 계산 코드를 메서드로 빼서 옮긴 후 클래스 관계

![]({{site.url}}/assets/images/ref06.PNG)

* 적립 포인트 계산 코드를 메서드로 빼서 옮긴 후 상호작용 다이어그램

![]({{site.url}}/assets/images/ref07.PNG)



#### 임시변수 없애기

* 임시 변수 frequentRenterPoints, totalAmout 변수를 각각 getTotalFrequentRenterPoints, getTotalCharge 메서드로 교체

```java
public class Customer {
	public String statement() {
		Enumeration<Rental> rentals = _rentals.elements();
		String result = getName() + " 고객님의 대여 기록\n";
		
		while (rentals.hasMoreElements()) {
			Rental each = rentals.nextElement();
			
			// 이번에 대여하는 비디오 정보와 대여료를 출력
			result += "\t" + each.getMovie().getTitle() + "\t" + String.valueOf(each.getCharge()) + "\n";
		}
		
		result += "누적 대여료: " + String.valueOf(getTotalCharge()) + "\n";
		result += "적립 포인트: " + String.valueOf(getTotalFrequentRenterPoints());
		
		return result;
	}

	private int getTotalFrequentRenterPoints() {
		int result = 0;
		Enumeration<Rental> rentals = _rentals.elements();
		
		while (rentals.hasMoreElements()) {
			Rental each = rentals.nextElement();
			result += each.getFrequentRenterPoints();
		}
		
		return result;
	}

	private double getTotalCharge() {
		double result = 0;
		Enumeration<Rental> rentals = _rentals.elements();
		
		while (rentals.hasMoreElements()) {
			Rental each = rentals.nextElement();
			result += each.getCharge();
		}
		
		return result;
	}
}
```

* 총 대여료 계산을 메서드로 빼기 전 클래스 호출 관계

![]({{site.url}}/assets/images/ref08.PNG)

* 총 대여료 계산을 메서드로 빼기 전 상호작용 다이어그램

![]({{site.url}}/assets/images/ref09.PNG)

* 총 대여료 계산을 메서드로 뺀 후 클래스 호출 관계

![]({{site.url}}/assets/images/ref10.PNG)

* 총 대여료 계산을 메서드로 뺀 후 상호 작용 다이어 그램

![]({{site.url}}/assets/images/ref11.PNG)



##### 여기까지 리펙토링 생각해보기

* 코드 양이 늘어났다.
* 성능이 안 좋아졌다. while 문 루프가 3번으로 늘어났다.
* 그러나 while 문은 최적화 단계에서 생각해도 된다.



#### htmlStatement 메서드 추가

```java
public String htmlStatement() {
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
```



* 이 상황에서 비디오 분류를 바꾸려고 한다.
* 각 비디오마다 대여료와 적립 비율을 결정해야한다.
* 이런 식의 수정은 지금 무리다. 그 전에 대여료 메서드와 적립 포인트 메서드부터 마무리 짓고 진행해야 한다.



### 가격 책정 부분의 조건문을 재정의 교체

* 타 객체의 속성을 switch 문의 인자로 하는 것은 나쁜 방법
* 자신의 데이터를 사용해야 한다.
* getCharge, getFrequentRenterPoints 메소드를 Movie로 옮긴다.

```java
public class Movie {
	double getCharge(int daysRented) {
		double result = 0;

		switch(getPriceCode()) {
			case Movie.REGULAR:
				result += 2;
				
				if (daysRented > 2) {
					result += (daysRented - 2) * 1.5;
				}
				
				break;
			case Movie.NEW_RELEASE:
				result += daysRented * 3;
				break;
			case Movie.CHILDRENS:
				result += 1.5;
				
				if (daysRented > 3) {
					result += (daysRented - 3) * 1.5;
				}
				
				break;
		}
		
		return result;
	}
    
    int getFrequentRenterPoints(int daysRented) {
		// 보통 영화는 포인트 1 지급, 최신물을 이틀 이상 대여하면 보너스 포인트 지급
		if (getPriceCode() == Movie.NEW_RELEASE && daysRented > 1) {
			return 2;
		} else {
			return 1;
		}
	}
}
```

```java
public class Rental {
	double getCharge() {
		return _movie.getCharge(_daysRented);
	}
    
    int getFrequentRenterPoints() {
		return _movie.getFrequentRenterPoints(_daysRented);
	}
}
```



* 메서드를 Movie 클래스로 옮기기 전 클래스 호출 관계

![]({{site.url}}/assets/images/ref12.PNG)

* 메서드를 Movie 클래스로 옮긴 후 클래스 호출 관계

![]({{site.url}}/assets/images/ref13.PNG)



#### 마지막 단계, 상속 구조 만들기

* Movie 클래스는 비디오 종류에 따라 같은 메소드 호출에도 각기 다른 값을 반환한다.
* 그런데 이건 하위클래스가 처리할 일이다.
* 따라서 Movie 클래스를 상속받는 3개의 하위클래스를 작성하고, 비디오 종류별 대여료 계산을 각 하위 클래스에 넣어야 한다.
* 이렇게 하면 switch 문을 재정의로 바꿀 수 있다.
* RegularPrice, ChildrensPrice, NewReleasePrice 클래스의 상위 클래스로 Price 클래스를 만들면 언제든 대여료를 변경할 수 있다.
* 하위 클래스를 작성해서 Movie 클래스를 상속 구조로 만듦

![]({{site.url}}/assets/images/ref14.PNG)

* Movie 클래스에 상태 패턴 적용

![]({{site.url}}/assets/images/ref15.PNG)

* 최종 코드

```java
public abstract class Price {
	abstract int getPriceCode();
	abstract double getCharge(int daysRented);
	
	int getFrequentRenterPoints(int daysRented) {
		return 1;
	}

}

class ChildrensPrice extends Price {
	@Override
	int getPriceCode() {
		return Movie.CHILDRENS;
	}

	@Override
	double getCharge(int daysRented) {
		double result = 1.5;
		
		if (daysRented > 3) {
			result += (daysRented - 3) * 1.5;
		}
		
		return result;
	}
}

class NewReleasesPrice extends Price {
	@Override
	int getPriceCode() {
		return Movie.NEW_RELEASE;
	}
	
	@Override
	double getCharge(int daysRented) {
		return daysRented * 3;
	}

	@Override
	int getFrequentRenterPoints(int daysRented) {
		return (daysRented > 1) ? 2 : 1;
	}
}

class RegularPrice extends Price {
	@Override
	int getPriceCode() {
		return Movie.REGULAR;
	}

	@Override
	double getCharge(int daysRented) {
		double result = 2;

		if (daysRented > 2) {
			result += (daysRented - 2) * 1.5;
		}
		
		return result;
	}
}
```

```java
public class Movie {
	public static final int CHILDRENS = 2;
	public static final int REGULAR = 0;
	public static final int NEW_RELEASE = 1;
	private String _title;
	private int _priceCode;
	private Price _price;
	
	public Movie(String title, int priceCode) {
		_title = title;
		setPriceCode(priceCode);
	}
	
	public int getPriceCode() {
		return _priceCode;
	}
	
	public String getTitle() {
		return _title;
	}

	public void setPriceCode(int arg) {
		switch(arg) {
			case Movie.REGULAR:
				_price = new RegularPrice();
				
				break;
			case Movie.NEW_RELEASE:
				_price = new ChildrensPrice();
				break;
			case Movie.CHILDRENS:
				_price = new NewReleasesPrice();
				break;
			default:
				throw new IllegalArgumentException("가격 코드가 잘못됐습니다.");
		}
	}
	
	public double getCharge(int _daysRented) {
		return _price.getCharge(_daysRented);
	}
	
	int getFrequentRenterPoints(int daysRented) {
		return _price.getFrequentRenterPoints(daysRented);
	}
}
```

```java
public class Rental {
	private Movie _movie;
	private int _daysRented;
	
	public Rental(Movie movie, int dayRented) {
		_movie = movie;
		_daysRented = dayRented;
	}

	public Movie getMovie() {
		return _movie;
	}

	public int getDaysRented() {
		return _daysRented;
	}

	int getFrequentRenterPoints() {
		return _movie.getFrequentRenterPoints(_daysRented);
	}
	
	double getCharge() {
		return _movie.getCharge(_daysRented);
	}
}
```

```java
public class Customer {
	private String _name;
	private Vector<Rental> _rentals = new Vector<Rental>();
	
	public Customer(String name) {
		_name = name;
	}
	
	public void addRental(Rental arg) {
		_rentals.add(arg);
	}
	
	public String getName() {
		return _name;
	}
	
	public String statement() {
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

	private int getTotalFrequentRenterPoints() {
		int result = 0;
		Enumeration<Rental> rentals = _rentals.elements();
		
		while (rentals.hasMoreElements()) {
			Rental each = rentals.nextElement();
			result += each.getFrequentRenterPoints();
		}
		
		return result;
	}

	private double getTotalCharge() {
		double result = 0;
		Enumeration<Rental> rentals = _rentals.elements();
		
		while (rentals.hasMoreElements()) {
			Rental each = rentals.nextElement();
			result += each.getCharge();
		}
		
		return result;
	}
	
	public String htmlStatement() {
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

* 이렇게 하면 대여료 계산 방식을 변경하거나
* 새 대여료를 추가하거나
* 부수적인 대여료 관련 동작을 추가할 때 아주 쉽게 수정할 수 있다.
* 위처럼 적용하는 것을 상태 패턴이라고 한다.
* 상태 패턴 적용후 클래스 호출 관계

![]({{site.url}}/assets/images/ref17.PNG)



##### ''간단한 수정 -> 테스트'' 를 리듬처럼 반복하는 것이 가장 중요