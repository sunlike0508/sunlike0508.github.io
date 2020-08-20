---
title:  "메서드 정리"
excerpt: "메서드 정리"
classes: wide
categories:
  - Refactoring
tags:
  - [Refactoring]
last_modified_at: 2020-07-15
---



# 메서드 정리

* 리펙토링의 주된 작업은 코드를 포장하는 메서드를 적절히 정리하는 것이다.
* 핵심적인 리펙토링 기법은 코드 뭉치를 별도의 메서드로 빼내는 메서드 추출이다.
* 그와 반대로 메서드 내용 직접 삽입은 호출되는 메서드의 내용을 호출하는 메서드에 직접 넣는 기법이다.
* 여러 묶음의 코드를 개별 메서드로 빼내고 보니 그렇게 만들어진 일부 메서드가 제 역할을 못하거나 그 메서드들을 쪼갠 방식을 바꿔야 할 때는 메서드 내용 직접 삽입 기법을 적용해야 한다.
* 메서드 추출에서 가장 힘든 작업은 지역변수를 처리하는 것인데, 그건 주로 임시 변수 때문이다.
* 메서드 정리 작업을 할 때 나는 임시변수를 메서드 호출로 전환을 실시해서 없어도 되는 임시변수를 전부 제거한다.
* 임시변수가 여러 부분에 사용될 때는 임시변수 분리를 먼저 실시하면 임시변수를 메서드 호출로 전환 기법이 더 간편해진다.
* 임시변수가 너무 얽혀 있어서 메서드 호출로 전환할 수 없을 때도 있다.
* 이럴 땐 메서드를 메서드 객체로 전환을 실시한다. 그러면 새 클래스를 만들어서 심각하게 얽혀 있는 메서드도 분리할 수 있다.
* 매개변수는 값만 대입하지 않으면 임시변수보다 문제가 덜하다. 하지만 매개변수로 대입하는 값이 있을 땐 매개변수로의 값 대입 제거를 실시해야 한다.
* 메서드를 잘개 쪼개면 동작 원리를 이해하기가 훨씬 쉽다. 게다가 간혹 알고리즘을 더 명료하게 개선할 수 있음을 알게 되기도 한다.
* 그럴 땐 알고리즘 전환을 실시해서 알고리즘을 더 확실하게 만들면 된다.



### 1. 메서드 추출

* 어떤 코드를 그룹으로 묶어도 되겠다고 판단될 땐 그 코드를 빼내어 목적을 잘 나타내는 직관적 이름의 메서드로 만들자.
* 리펙토링전

```java
void printOwing(double amount) {
    printBanner();

    System.out.println("name :" + _name);
    System.out.println("name :" + amount);
}
```

* 리펙토링 후

```java
void printOwing(double amount) {
    printBanner();
    printDetails(amount);
}

void printDetails(double amount) {
    System.out.println("name :" + _name);
    System.out.println("name :" + amount);
}
```



#### 1.1 동기

* 메서드 추출기법은 제일 많이 사용된다.
* 메서드가 너무 길거나 코드에 주석을 달아야만 의도를 이해할 수 있을 때, 코드를 빼내어 별도의 메서드를 만든다.
* 직관적인 이름의 간결한 메서드가 좋은 이유는 다음과 같다.
  1. 메서드가 적절히 잘게 쪼개져 있으면 다른 메서드에서 쉽게 사용할 수 있다.
  2. 상위 계층의 메서드에서 주석 같은 더 많은 정보를 읽어들일 수 있다.
  3. 재정의하기도 훨씬 수월하다.

#### 1.2 방법

* 목적에 부합하는 이름의 새 메서드를 생성하자. 이때 메서드명은 원리가 아니라 기능을 나타내는 이름이어야 한다.
  * 메서드로 빼낼 코드가 한 줄의 명령이나 함수 호출 같이 아주 간단한 것이라면 새 메서드명을 통해 코드의 기능(목적)을 더 잘 드러 낼 수 있을 때만 추출을 한다.
  * 더 이해하기 쉬운 이름으로 추출하지 않을 바에는 추출하지 말자.
* 기존 메서드에서 빼낸 코드를 새로 생성한 메서드로 복사하자.
* 빼낸 코드에서 기존 메서드의 모든 지역변수 참조를 찾자. 그것들을 새로 생성한 메서드 안에 임시변수로 선언하자.
* 추출 코드에 의해 변경되는 지역변수가 있는지 파악하자. 만약 하나의 지역변수만 변경된다면 추출코드를 메서드 호출처럼 취급할 수 있는지 알아내고 그 결과를 관련된 변수에 대입할 수 있는지 알아내자. 이렇게 하기가 까다롭거나 둘 이상의 지역변수가 변경될 때는 메서드를 추출하기 위해 먼저 임시변수 분리 등의 기법을 적용해야 할 수도 있다. 임시변수를 제거하려면 임시변수를 메서드 호출로 전환 기법을 적용하면 된다.
* 빼낸 코드에서 읽어드린 지역변수를 대상 메서드에 매개변수로 전달한다.
* 모든 지역변수 처리를 완료했으면 컴파일을 실시하자.
* 원본 메서드 안에 있는 빼낸 코드 부분을 새로 생성한 메서드 호출로 수정하자.
  * 대상 메서드로 임시변수를 옮겼으면 그 임시변수가 원본 코드 외부에 선언되어 있었는지 검사해서 그렇다면 대상 코드에서는 그 선언 부분을 삭제하자.



#### 1.3 예제

##### 1.3.1 지역변수 사용 안함

* 리팩토링 전 (앞으로 이 코드를 계속해서 사용)

```java
void printOwing() {
    Enumeration e = _orders.elements();
    double outstanding = 0.0;

    System.out.println("**************");
    System.out.println("** 고객 외상 **");
    System.out.println("**************");

    while(e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        outstanding += each.getAmount();
    }

    System.out.println("name :" + _name);
    System.out.println("name :" + outstanding);
}
```

* 리팩토링 후

```java
void printOwing() {
    Enumeration e = _orders.elements();
    double outstanding = 0.0;

    printBanner();

    while(e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        outstanding += each.getAmount();
    }

    System.out.println("name :" + _name);
    System.out.println("name :" + outstanding);
}

void printBanner() {
    System.out.println("*************");
    System.out.println("** 고객 외상 **");
    System.out.println("*************");
}
```



##### 1.3.2 지역변수 사용

* 원본 메서드로 전달된 매개변수와 원본 메서드안에 선언된 임시변수가 문제다.
* 지역변수는 해당 메서드 안에서만 효력이 있으므로 메서드 추출을 적용하면 지역변수와 관련된 작업을 추가로 처리해야 한다.
* 간혹 지역변수 때문에 리팩토링이 아예 불가능할 때도 있다.
* 리팩토링 후

```java
void printOwing() {
    Enumeration e = _orders.elements();
    double outstanding = 0.0;

    printBanner();

    while(e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        outstanding += each.getAmount();
    }

    printDetails(outstanding);
}

void printBanner() {
    System.out.println("************");
    System.out.println("** 고객 외상 **");
    System.out.println("************");
}

void printDetails(double outstanding) {
    System.out.println("name :" + _name);
    System.out.println("name :" + outstanding);
}
```



##### 1.3.3 지역변수를 다시 대입

* 매개변수로의 값 대입이 있을 경우엔 즉시 매개변수로의 값 대입 제거를 실시해야 한다.
* 임시변수로의 값 대입은 두 가지 경우가 있다.
  1. 임시변수가 추출한 코드 안에서만 사용되는 경우, 임시변수를 추출한 코드로 옮기면 된다.
  2. 임시변수가 추출한 코드 밖에서 사용되는 경우, 만약 그 임시변수가 코드 추출된 후 사용되지 않는 다면 추출한 코드에서 그 임수변수의 변경된 값을 반환하게 수정해야 한다.
* 아래는 임시변수가 추출한 코드 밖에서 사용되는 경우로 리팩토링

```java
void printOwing() {
    printBanner();

    double outstanding = getOutstanding(e);

    printDetails(outstanding);
}

private double getOutstanding() {
    Enumeration e = _orders.elements();
    double result = 0.0;

    while(e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        result += each.getAmount();
    }

    return result;
}

void printBanner() {
    System.out.println("************");
    System.out.println("** 고객 외상 **");
    System.out.println("************");
}

void printDetails(double outstanding) {
    System.out.println("name :" + _name);
    System.out.println("name :" + outstanding);
}
```

* 변환 값의 이름을 변경하자. outstanding -> result
* 임시변수에 더 복잡한 작업이 일어 날땐 이전 값을 매개변수로 전달해야 한다.

* 리팩토링 전

```java
void printOwing(double previousAmount) {
    double outstanding = previousAmount * 1.2;
    printBanner();

    Enumeration e = _orders.elements();
    double result = 0.0;

    while(e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        result += each.getAmount();
    }

    printDetails(outstanding);
}
```

* 리팩토링 후

```java
void printOwing(double previousAmount) {
    double outstanding = previousAmount * 1.2;
    printBanner();

    outstanding = getOutstanding(outstanding);

    printDetails(outstanding);
}

private double getOutstanding(double initialValue) {
    Enumeration e = _orders.elements();
    double result = 0.0;

    while(e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        result += each.getAmount();
    }

    return result;
}
```

* outstanding 변수의 초기화 방식을 다음과 같이 수정한다.

```java
void printOwing(double previousAmount) {
    printBanner();

    outstanding = getOutstanding(previousAmount * 1.2);

    printDetails(outstanding);
}
```

* 변수 두 개 이상 반환해야 할 경우 최선의 방법은 다른 부분의 코드를 빼내는 것이다.
* 각기 다른 값을 하나씩 반환하는 여러 개의 메서드를 만든다.
* 자신이 사용하는 언어에 출력 매개변수 기능이 있다면 출력 매개변수를 사용하면 된다.
* 가능하면 하나의 값을 반환하는 것이 좋다.
* 임시변수가 너무 많으면 메서드 호출 전환 기법을 실시해서 임시변수를 줄인다.
* 이것도 힘들면 메서드를 메서드 객체로 전환 기법을 사용한다.



### 2. 메서드 내용 직접 삽입

* 메서드 기능이 너무 단순해서 메서드명만 봐도 너무 뻔할 땐 그 메서드의 기능을 호출되는 메서드에 넣어버리고 그 메서드는 삭제하자.

```java
int getRating() {
    return (moreThanFiveLateDeliveries()) ? 2 : 1;
}

private boolean moreThanFiveLateDeliveries() {
    return _numberOfLateDeliveries > 5;
}
```

```java
int getRating() {
    return (_numberOfLateDeliveries > 5) ? 2 : 1;
}
```



#### 2.1 동기

* 리팩토링 핵심은 의도한 기능을 한눈에 파악할 수 있는 직관적인 메서드명을 사용하는 것과 메서드를 간결하게 만드는 것이다.
* 그러나 메서드명에 모든 기능이 반영될 정도로 메서드 기능이 지나치게 단순할 때 없어애 한다.
* 인다이렉션을 사용하면 해결되지만 불필요한 인다이렉션은 오히려 장애물이다.
* 메소드 내용 직접 삽입 기법은 잘못 쪼개진 메서드에도 적용할 수 있다.
* 잘못 쪼개진 메서드의 내용을 전부 하나의 메서드에 직접 삽입 후, 다시 작은 메서드로 추출하면 된다.
* 메서드를 메서드 객체로 전환 기법 사용하기 전에 이 기법을 먼저 적용하면 좋다.
* 과다한 인다이렉션과 동시에 모든 메서드가 다른 메서드에 단순히 위임을 하고 있어서 코드가 지나치게 복잡할 땐 주로 메서드 내용 직접 삽입을 실시한다.
* 이럴 땐 일부 인다이렉션을 제외한 나머지는 불필요하다.
* 메서드 내용 직접 삽입을 통해 필요한 인다이렉션을 정리하고 나머지는 제거할 수 있다.



#### 2.2 방법

* 메서드가 재정의되어 있지 않은지 확인하자
  * 그 메서드가 하위클래스에 재정의되어 있다면 메서드 내용 직접 삽입을 실시하지 말자.
* 그 메서드를 호출하는 부분을 모두 찾자.
* 각 호출 부분을 메서드 내용으로 교체하자.
* 메서드 정의를 삭제하자.



### 3. 임시변수 내용 직접 삽입

* 간단한 수식을 대입받는 임시변수로 인해 다른 리팩토링 기법 적용이 힘들 땐 그 임시변수를 참조하는 부분을 전부 수식으로 치환하자.

```java
boolean isBasePriceThan1000() {
    double basePrice = anOrder.basePrice();
    return basePrice > 1000;
}
```

```java
boolean isBasePriceThan1000() {
    return anOrder.basePrice() > 1000;
}
```



#### 3.1 동기

* 임시변수 내용 직접 삽입은 임시변수를 메서드 호출로 전환기법을 실시하는 도중에 병행하게 되는 경우가 태반이다.
* 임시변수 내용 직접 삽입은 임시변수를 메서드 호출로 전환을 실시해야 하는 동기라고 할 수 있다.
* 임시변수가 메서드 추출 등 리팩토링에 방해가 된다면 임시변수 내용 직접 삽입을 적용해야 한다.



#### 3.2 방법

* 대입문의 우변에 문제가 없는지 확인하자.
* 문제가 없다면 임시변수를 final로 선언하고 컴파일하자.
  * 이것으로 그 임시변수에는 값을 한 번만 대입할 수 있다.
* 그 임시변수를 참조하는 모든 부분을 찾아서 대입문 우변의 수식으로 바꾸자.
* 하나씩 수정을 마칠 때마다 컴파일과 테스트를 실시하자.
* 임시변수 선언과 대입문을 삭제하자.



### 4. 임시변수를 메서드 호출로 전환

* 수식의 결과를 저장하는 임시변수가 있을 땐 그 수식을 빼내어 메서드로 만든 후, 임시변수 참조부분을 전부 수식으로 교체하자.
* 새로 만든 메서드는 다른 메서드에서도 호출 가능하다.

```java
double calculateBasePrice() {
    double basePrice = _quantitiy * _itemPrice;

    if(basePrice > 1000) {
        return basePrice * 0.95;
    } else {
        return basePrice * 0.98;
    }
}
```

```java
double calculateBasePrice() {
    if(getBasePrice() > 1000) {
        return getBasePrice() * 0.95;
    } else {
        return getBasePrice() * 0.98;
    }
}

private double getBasePrice() {
    return _quantitiy * _itemPrice;
}
```



#### 4.1 동기

* 임시변수는 일시적이고 적용이 국소적 범위로 제한된다는 단점이 있다.
* 임시변수를 메서드 호출로 수정하면 클래스 안 모든 메서드가 그 정보에 접근할 수 있다.
* 임시변수를 메서드 호출로 전환은 대부분의 경우 메서드 추출을 적용하기 전에 반드시 적용해야 한다.
* 지역변수가 많을수록 메서드 추출이 힘들어지므로 최대한 많은 변수를 메서드 호출로 고쳐야 한다.
* 가장 간단한 상황은 임시변수에 값이 한 번만 대입되고 대입문을 이루는 수식에 문제가 없을 때다.
* 간편한 리팩토링을 위해 임시변수 분리나 상태 변경 메서드와 값 반환 메서드를 분리를 먼저 적용해야 할 때도 있다.



#### 4.2 방법

* 값이 한 번만 대입되는 임시변수를 찾자.
  * 값이 여러 번 대입되는 임시변수는 임시변수 분리 기법을 고려하자.
* 그 임시변수를 final로 선언하자.
* 컴파일을 실시하자.
  * 이것으로 임시변수엔 값을 한 번만 대입할 수 있다.
* 대입문 우변을 빼내어 메서드로 만들자.
  * 처음에는 메서드를 private로 선언하자. 그러다 나중에 더 여러 곳에서 사용하게 되면 접근 제한을 간단히 완화하면 된다.
  * 추출 메서드에 문제는 없는지(즉, 객체를 변경하진 않는지) 확인하자. 만약 객체 변경 등의 문제가 있다면 상태 변경 메서드와 값 반환 메서드를 분리 기법을 실시하자.
* 임시변수를 대상으로 임시변수 내용 직접 삽입 기법을 실시하자.



#### 4.3 예제

* 리팩토링 전

```java
double getPrice() {
    int basePrice = _quantity * _itemPrice;
    double discountFactor;

    if(basePrice > 1000) {
        discountFactor = 0.95;
    } else {
        discountFactor = 0.98;
    }

    return basePrice * discountFactor;
}
```

* final로 선언하여 임시변수가 값을 한번만 대입 받는지 시험해보자.

```java
double getPrice() {
    final int basePrice = _quantity * _itemPrice;
    final double discountFactor;

    if(basePrice > 1000) {
        discountFactor = 0.95;
    } else {
        discountFactor = 0.98;
    }

    return basePrice * discountFactor;
}
```

* 컴파일에 문제가 생기면 경고가 출력된다.
* 문제가 있으면 임시변수를 메서드 호출로 전환하는 기법을 실시하지 말아야 하므로 컴파일부터 해야 한다.
* 임시변수를 한 번에 하나씩 메서드 호출로 바꾸자.

```java
double getPrice() {
    final int basePrice = basePrice();
    final double discountFactor;

    if(basePrice > 1000) {
        discountFactor = 0.95;
    } else {
        discountFactor = 0.98;
    }

    return basePrice * discountFactor;
}

private int basePrice() {
    return _quantity * _itemPrice;
}
```

* 컴파일과 테스트 후 임시변수 내용 직접 삽입을 실시하자.

```java
double getPrice() {
    final int basePrice = basePrice();
    final double discountFactor;

    if(basePrice() > 1000) {
        discountFactor = 0.95;
    } else {
        discountFactor = 0.98;
    }

    return basePrice * discountFactor;
}

private int basePrice() {
    return _quantity * _itemPrice;
}
```

* 컴파일과 테스트 후 첫 번째 임시변수 삭제하고 두 번째 임시변수도 바꾼다.

```java
double getPrice() {
    final double discountFactor = discountFactor();

    return basePrice() * discountFactor;
}

private double discountFactor() {
    if(basePrice() > 1000) {
        return 0.95;
    } else {
        return 0.98;
    }
}

private int basePrice() {
    return _quantity * _itemPrice;
}
```

* 컴파일과 테스트 후 두번 째 변수도 더이상 참조 부분이 없으므로 삭제한다.

```java
double getPrice() {		
    return basePrice() * discountFactor();
}

private double discountFactor() {
    if(basePrice() > 1000) {
        return 0.95;
    } else {
        return 0.98;
    }
}

private int basePrice() {
    return _quantity * _itemPrice;
}
```



### 5. 직관적 임시변수 사용

* 사용된 수식이 복잡할 땐 수식의 결과나 수식의 일부분을 용도에 부합하는 직관적 이름의 임시변수에 대입하자.

```java
void test() {
    if((platform.toUpperCase().indexOf("MAC") > -1) 
       && (browser.toUpperCase().indexOf("IE") > -1)
       && wasInitialized()
       && resize > 0 ) {
        // 기능 코드
    }
}
```

```java
final boolean isMacOs = platform.toUpperCase().indexOf("MAC") > -1;
final boolean isIEBrowser = browser.toUpperCase().indexOf("IE") > -1;
final boolean wasResized = resize > 0;

void test() {
    if(isMacOs && isIEBrowser && wasInitialized() && wasResized ) {
        // 기능 코드
    }
}
```



#### 5.1 동기

* 수식이 너무 복잡하면 임시변수를 사용하여 쉽게 쪼개라.
* 직관적 임시변수 사용 기법은 각 조건 절을 가져와서 직관적 이름의 임시변수를 사용해 그 조건의 의미를 설명하려 할 때 많이 사용된다.
* 그 외에 긴 알고리즘에서 임시변수를 사용해서 계산의 각 단계를 설명할 수 있을 때도 사용한다.
* 그러나 왠만하면 메서드 추출을 사용하려 노력하는 것이 좋다.



#### 5.2 방법

* 임시변수를 final로 선언하고, 복잡한 수식에서 한 부분의 결과를 그 임시변수에 대입하자.
* 그 수식에서 한 부분의 결과를 그 임시변수의 값으로 교체하자.
  * 수식의 결과 부분이 반복될 경우엔 한 번에 하나씩 교체하면 된다.
* 수식의 다른 부분을 대상으로 위의 과정을 반복 실시하자.



#### 5.3 예제

##### 5.3.1 직관적 임시변수 사용 

```java
double price() {
    // 결제액 (price) = 총 구매액(base price) 
    // - 대량 구매 할인(quauntit discount) + 배송비(shipping)
    return quantity + itemPrice 
        - Math.max(0, quantity - 500) * itemPrice * 0.05 
        + Math.min(quantity * itemPrice * 0.1, 100.0);
}
```

* 다음과 같이 임시 변수로 바꾼다.

```java
double price() {
    // 결제액 (price) = 총 구매액(base price) 
    // - 대량 구매 할인(quauntit discount) + 배송비(shipping)
    final double basePrice = quantity + itemPrice;
    return basePrice
        - Math.max(0, quantity - 500) * itemPrice * 0.05 
        + Math.min(basePrice * 0.1, 100.0);
}
```

* 대량 구매 할인액과 배송료에 해당하는 임시변수를 사용한다.
* 이렇게 하면 주석을 제거 할 수 있다.

```java
double price() {
    final double basePrice = quantity + itemPrice;
    final double quantitiyDiscount = Math.max(0, quantity - 500) * itemPrice * 0.05;
    final double shipping = Math.min(basePrice * 0.1, 100.0);

    return basePrice - quantitiyDiscount + shipping;
}
```



##### 5.3.2 메서드 추출

* 위의 예제를 가지고 메서드 추출

```java
double price() {		
    return basePrice() - quantitiyDiscount() + shipping();
}

private double shipping() {
    return Math.min(basePrice() * 0.1, 100.0);
}

private double quantitiyDiscount() {
    return Math.max(0, quantity - 500) * itemPrice * 0.05;
}

private double basePrice() {
    return quantity + itemPrice;
}
```

* 메서드 추출이 더 좋은 이유는 객체의 다른 부분에서 사용가능하기 때문이다.
* 메서드 추출을 적용하기 힘들면 직관적 임시변수 사용 기법을 사용한다.



### 6. 임시변수 분리

* 루프 변수나 값 누적용 임시변수가 아닌 임시변수에 여러 번 값이 대입될 땐 각 대입마다 다른 임시변수를 사용하자.

```java
void test() {
    double temp = 2 * (height + width);
    System.out.println(temp);
    temp = height + width;
    System.out.println(temp);
}
```

```java
void test() {
    double temp = 2 * (height + width);
    System.out.println(temp);
    double area = height + width;
    System.out.println(area);
}
```



#### 6.1 동기

* 임시변수는 긴 코드의 계산 결과를 나중에 간편히 참조할 수 있게 저장하는 용도로 사용된다.
* 값은 한번만 대입되어야 한다. 그 이상은 메서드 안에서 여러 용도로 사용된다는 반증이다.
* 이럴 때는 각 용도 별로 다른 변수를 사용하게 분리해야 한다.



#### 6.2 방법

* 선언문과 첫 번째 대입문에 있는 임시변수 이름을 변경하자.
* 이름을 바꾼 새 임시변수를 final로 선언하자.
* 그 임시변수의 모든 참조 부분을 두 번째 대입문으로 수정하자.
* 두 번째 대입문에 있는 임시변수를 선언하자.
* 컴파일과 테스트를 실시하자.
* 각 대입문마다 차례로 선언문에서 임시변수 이름을 변경하고, 그 다음 대입문까지 참조를 수정하며 위의 과정을 반복한다.



#### 6.3 예제

```java
double getDistanceTravelled (int time) {
    double result;
    double acc = primaryForce / mass;
    int primaryTime = Math.min(time, delay);
    result = 0.5 * acc * primaryTime * primaryTime;
    int secondaryTime = time - delay;

    if(secondaryTime > 0) {
        double primaryVel = acc * delay;
        acc = (primaryForce * secondaryForce) / mass;
        result += primaryVel * secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
    }

    return result;
}
```

* acc 값이 두번 대입된다.
* 임시 변수 이름을 변경하고 새 변수를 final로 선언한다.
* 거기서부터 다음 대입문까지의 임시변수 참조 부분을 전부 수정한다.
* 다음 대입문에서 임시변수를 다음과 같이 선언한다.

```java
double getDistanceTravelled (int time) {
    double result;
    final double primaryAcc = primaryForce / mass;
    int primaryTime = Math.min(time, delay);
    result = 0.5 * primaryAcc * primaryTime * primaryTime;
    int secondaryTime = time - delay;

    if(secondaryTime > 0) {
        double primaryVel = primaryAcc * delay;
        double acc = (primaryForce * secondaryForce) / mass;
        result += primaryVel * secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
    }

    return result;
}
```

* 두 번째 임시변수도 똑같이 하자.

```java
double getDistanceTravelled (int time) {
    double result;
    final double primaryAcc = primaryForce / mass;
    int primaryTime = Math.min(time, delay);
    result = 0.5 * primaryAcc * primaryTime * primaryTime;
    int secondaryTime = time - delay;

    if(secondaryTime > 0) {
        double primaryVel = primaryAcc * delay;
        final double secondaryAcc = (primaryForce * secondaryForce) / mass;
        result += primaryVel * secondaryTime + 0.5 * secondaryAcc * secondaryTime * secondaryTime;
    }

    return result;
}
```

* 책에서 이후에는 직접 해보라고 해서 직접 해봄

```java
double getDistanceTravelled (int time) {
    return primaryDistance(time) + secondaryDistance(time);
}

private double secondaryDistance(int time) {
    return (secondaryTime(time) > 0) ? primaryVel() * secondaryTime(time) 
            + 0.5 * secondaryAcc() * secondaryTime(time) * secondaryTime(time) : 0;
}

private double primaryDistance(int time) {
    return 0.5 * primaryAcc() * primaryTime(time) * primaryTime(time);
}

private int secondaryTime(int time) {
    return time - delay;
}

private double secondaryAcc() {
    return (primaryForce * secondaryForce) / mass;
}

private double primaryVel() {`
    return primaryAcc() * delay;
}

private int primaryTime(int time) {
    return Math.min(time, delay);
}

private double primaryAcc() {
    return primaryForce / mass;
}
```



### 7. 매개변수로의 값 대입 제거

* 매개변수로 값을 대입하는 코드가 있을 땐 매개변수 대신 임시변수를 사용하게 수정하자.

```java
int discount (int inputVal, int quantity, int yearToDate) {
    if(inputVal > 50) {
        inputVal -= 2;
    }

    // 기능 코드

    return outputVal;
}
```

```java
int discount (int inputVal, int quantity, int yearToDate) {
    int result = inputVal;
    if(inputVal > 50) {
        result -= 2;
    }

    // 기능 코드

    return outputVal;
}
```



#### 7.1 동기

* 매개변수로의 값 대입의 의미는 어떤 메서드에 foo 객체를 매개변수로 전달하면 foo의 값을 다른 객체 참조로 변경한다는 의미다.

* 매개변수로 전달받은 객체에 어떠한 처리를 하든 상관없고 일상적인 작업이지만, foo의 값을 다른 객체 참조로 변경하는 것은 절대로 안된다.

```java
int discount (int inputVal, int quantity, int yearToDate) {
    int result = inputVal; // 괜찮다.
    inputVal = anotherVal; // 안된다.

}
```

* 전달받은 객체를 나타내는 용도로만 매개변수를 사용하면 용도의 일관성으로 코드가 읽기 쉽다.



#### 7.2 방법

* 매개변수 대신 사용할 임시변수를 선언하자.
* 매개변수로 값을 대입하는 코드 뒤에 나오는 매개변수 참조를 전부 임시변수로 수정하자.
* 매개변수로의 값 대입을 임시변수로의 값 대입으로 수정하자.



#### 7.3 예제

```java
int discount (int inputVal, int quantity, int yearToDate) {
    if(inputVal > 50) {
        inputVal -= 2;
    }
    if(quantity > 50) {
        inputVal -= 1;
    }
    if(yearToDate > 50) {
        inputVal -= 4;
    }

    return inputVal;
}
```

* 리팩토링 후
* final의 경우 다른 코드가 매개변수에 새 값을 대입하는지 쉽게 파악하기 위해서 사용한다.
* 매개변수 셋트는 final을 하지 않는다.

```java
int discount (final int inputVal, final int quantity, final int yearToDate) {
    int result = inputVal;

    if(inputVal > 50) {
        result -= 2;
    }
    if(quantity > 50) {
        result -= 1;
    }
    if(yearToDate > 50) {
        result -= 4;
    }

    return result;
}
```



### 8. 메서드를 메서드 객체로 전환

* 지역변수 때문에 메서드 추출을 적용할 수 없는 긴 메서드가 있을 땐 그 메서드 자체를 객체로 전환해서 모든 지역변수를 객체의 필드로 만들자.
* 그런 다음 그 메서드를 객체 안의 여러 메서드로 쪼개면 된다.

```java
public class Order {
	double price() {
		double primaryBasePrice;
		double secondaryBasePrice;
		double tertiaryBasePrice;
		// 긴 계산 코드;
	}
}
```

![]({{site.url}}/assets/images/ref18.jpg)



#### 8.1 동기

* 지역변수가 많으면 메서드 분해가 힘들다.
* 임시변수를 메서드 호출로 전환을 적용하면 이런 어려움이 어느정도 해소되지만, 분해가 필요한 메서드를 분해할 수 없을 때도 있다.
* 이럴땐 메서드 객체로 수정해야한다.
* 메서드를 메서드 객체로 전환 기법을 적용하면 모든 지역변수가 메서드 객체의 속성이 된다.
* 그러면 그 객체에 메서드 추출을 적용해서 원래의 메서드를 쪼개어 여러 개의 추가 메서드를 만들면 된다.



#### 8.2 방법

* 전환할 메서드의 이름과 같은 이름으로 새 클래스를 생성하자.
* 그 클래스에 원본 메서드가 들어 있던 객체를 나타내는 final 필드를 추가하고 원본 메서드 안의 각 임시변수와 매개변수에 해당하는 속성을 추가하자.
* 새 클래스에 원본 객체와 각 매개변수를 받는 생성자 메서드를 작성하자.
* 새 클래스에 compute라는 이름의 메서드를 작성하자.
* 원본 메서드 내용을 compute 메서드 안에 복사해 넣자. 원본 객체에 있는 메서드를 호출할 땐 원본 객체를 나타내는 필드를 사용하자.
* 원본 메서드를 새 객체 생성과 compute 메서드 호출을 담당하는 메서드로 바꾸자.



#### 8.3 예제

```java
public class Account {
    //delta() 메소드가 있다고 가정
	int gamma (int inputVal, int quantity, int yearToDate) {
		int importantValue1 = (inputVal * quantity) + delta();
		int importantValue2 = (inputVal * yearToDate) + 100;
		
		if(yearToDate - importantValue1 > 100) {
			importantValue2 -= 20;
		}
		
		int importantValue3 = importantValue2 * 7;
		
		return importantValue3 - 2 * importantValue1;
	}
}
```

* gamma 메서드를 메서드 객체로 전환하고자 우선 새 클래스를 선언한다.
* 원본 객체를 나타내는 final 필드를 작성하고 원본 메서드 안의 각 매개변수와 임시변수를 나타내는 필드를 작성한다.
* 그리고 생성자 메서드를 추가한다.

```java
class Gamma {
	private final Account account;
	private int inputVal;
	private int quantity;
	private int yearToDate;
	private int importantValue1;
	private int importantValue2;
	private int importantValue3;
    
    Gamma (Account source, int inputValArg, int quantityArg, int yearToDateArg) {
		account = source;
		inputVal = inputValArg;
		quantity = quantityArg;
		yearToDate = yearToDateArg;
	}
}
```

* 그리고 Account 객체의 메서드 호출 부분을 account 필드를 사용하게 수정한다.
* 그리고 원본 메서드가 이 메서드 객체로 위임하게 수정한다.
* 아래는 최종 리펙토링

```java
class Account {
	int gamma (int inputVal, int quantity, int yearToDate) {
		return new Gamma(this, inputVal, quantity, yearToDate).compute();
	}
}

class Gamma {
	private final Account account;
	private int inputVal;
	private int quantity;
	private int yearToDate;
	private int importantValue1;
	private int importantValue2;
	private int importantValue3;
	
	Gamma (Account source, int inputValArg, int quantityArg, int yearToDateArg) {
		account = source;
		inputVal = inputValArg;
		quantity = quantityArg;
		yearToDate = yearToDateArg;
	}
	
	int compute() {
		importantValue1 = (inputVal * quantity) + account.delta();
		importantValue2 = (inputVal * yearToDate) + 100;
		importantThing();
		int importantValue3 = importantValue2 * 7;
		
		return importantValue3 - 2 * importantValue1;
	}
    
    void importantThing() {
        if(yearToDate - importantValue1 > 100) {
			importantValue2 -= 20;
		}
    }
}
```



### 9. 알고리즘 전환

* 알고리즘을 더 분명한 것으로 교체해야 할 땐 해당 메서드의 내용을 새 알고리즘으로 바꾸자.

```java
String foundPerson(String[] people) {
    for(int i = 0; i < people.length; i++) {
        if(people[i].equals("Don")) {
            return "Don";
        }

        if(people[i].equals("John")) {
            return "John";
        }

        if(people[i].equals("Kent")) {
            return "Kent";
        }
    }

    return "";
}
```

```java
String foundPerson(String[] people) {
    List<String> candidates = Arrays.asList(new String[] {"Don", "John", "Kent" });

    for(int i = 0; i < people.length; i++) {
        if(candidates.contains(people[i])) {
            return people[i];
        }
    }

    return "";
}
```



#### 9.1 동기

* 어떤 기능을 수행하기 위한 비교적 간단한 방법이 있다면 교체한다.



#### 9.2 방법

* 간결한 알고리즘을 준비한다.
* 모든 테스트 결과가 같으면 성공
* 결과가 다르면 이-전과 비교한다.