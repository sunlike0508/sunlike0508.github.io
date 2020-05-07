---
title:  "Recursion의 개념"
excerpt: "Recursion의 개념과 기본 예제들 (1-3)(2-3)(3-3)"
classes: wide
categories:
  - algorithm
tags:
  - [algorithm, recursion]
last_modified_at: 2020-05-07
---



# Recursion의 개념과 기본 예제들

* 순환함수(재귀함수)는 자기 자신을 호출하는 함수
* 무한 루프에 빠질 수 있다. 하지만 올바르게 작성한다면 괜찮다.
* 순환함수는 반복문(interation)으로 변경 가능. 그 역도 성립
* 경우에 따라 반복문보다 간단하게 보일 수 있음
* 적어도 하나의 재귀에 빠지지 않는 경우가 존재해야 한다. 이를 base case라 한다.
* 반복하다보면 결국 base case로 수렴해야하는 조건이 있어야 한다. 이를 recursion case라 한다.
* 팩토리얼 함수 예제

```java
public static void main(String[] argv) {
	System.out.println(factorial(10));
}

private static int factorial(int i) {
	if(i <= 1) { //base case // 0!=1 이므로 조건을 이렇게
		return 1;
	}

	return i+ factorial(i-1); //recursion case
}
```

* 최대공약수 예제

```java
public static void main(String[] argv) {
	System.out.println(gcd(12, 18));
}

private static int gcd(int i, int j) {

	if(j == 0) { // base case
		return i;
	}

	return gcd(j, i % j); // recursion case
}
```

* 피보나치 예제

```java
public static void main(String[] argv) {
	System.out.println(fibonacci(5));
}

private static int fibonacci(int i) {

	if(i <=2) {
		return i;
	}

	return fibonacci(i-2) + fibonacci(i-1);
}
```

* 문자열을 역순으로 출력하는 예제

```java
public static void main(String[] argv) {
	reverseString("recursion");
}

private static void reverseString(String string) {

    if(string.length() == 0) {
    	return;
    }

    reverseString(string.substring(1));

    System.out.print(string.charAt(0));
}
```

* 이진수로 변환 하는 예제

```java
public static void main(String[] argv) {
	printBinary(12);
}

private static void printBinary(int i) {
    if(i < 2) {
    	System.out.print(i);
    	return;
    }

    printBinary(i / 2);

    System.out.print(i % 2);
}
```



### 순환적 알고리즘 설계

* 암시적(implicit) 매개변수를 명시적(explicit) 매개변수로 바꿔라

```java
int search(int[] data, int n, int target) {
    for(int i = 0; i < n ; i++) {
        if(data[i] == target) {
        	return i;
        }
    }

    return -1;
}
// 검색 구간의 인덱스 0은 부터 시작. 명시적이아닌 즉, 암시적 매개변수이다.
```

* 위의 코드를 순환함수로 변경

```java
int search(int[] data, int begin, int end, int target) {
    if(begin > end) {
        return -1;
    }
    else if(target == data[begin]) {
        return begin;
    } else {
        return search(data, begin+1, end, target);
    }
}
// 검색 구간의 시작점을 명시적으로 지정.
// search(data, 0, n-1, target)으로 호출
```

