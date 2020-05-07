---
title:  "Recursion의 개념"
excerpt: "Recursion의 개념과 기본 예제들 (1-3)"
classes: wide
categories:
  - algorithm
tags:
  - [algorithm, recursion]
last_modified_at: 2020-05-07
---



# Recursion의 개념과 기본 예제들 (1-3)

* 재귀함수는 자기 자신을 호출하는 함수
* 무한 루프에 빠질 수 있다. 하지만 올바르게 작성한다면 괜찮다.
* 적어도 하나의 재귀에 빠지지 않는 경우가 존재해야 한다. 이를 base case라 한다.
* 반복하다보면 결국 base case로 수렴해야하는 조건이 있어야 한다. 이를 recursion case라 한다.
* 가장 뻔한 예제로 드는 팩토리얼 함수

```java
public static void main(String[] argv) {
	System.out.println(factorial(10));
}

private static int factorial(int i) {
	if(i <= 1) { //base case
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

