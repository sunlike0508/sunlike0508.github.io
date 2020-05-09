---
title:  "Recursion의 개념"
excerpt: "Recursion의 개념과 기본 예제 & 응용"
classes: wide
categories:
  - algorithm
tags:
  - [algorithm, recursion]
last_modified_at: 2020-05-08
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



### 응용 예제 : 미로찾기

* 현재 위치에서 출구까지 가는 경로가 있으려면
  1. 현재 위치가 출구이거나 혹은
  2. 이웃한 셀들 중 하나에서 현재 위치를 지나지 않고 출구까지 가는 경로가 있거나

* **pseudo code**

```java
boolean findPath(x,y){
    if(x,y) is the exit // 1
        return true;
    else
        mark (x,y) as a visited cell; // 2
        for each neighbouring cell (x',y') of (x,y) do // 3
            if(x',y') is on the pathWay and not visited// 4
                if findPath(x',y') // 5
                    return true;
    	return false;
}
```

* 1 - x,y가 출구이면 true
* 2 - x,y가 이미 접근했던 위치라면 표시
* 3 - x,y에 인접한 cell(경로)들을 순환하는데 (최대 4번)
* 4 - x',y'가 유효한 cell(경로)이라면 (벽이거나 이미 방문했던 것은 재검사 필요 없음)
* 5 - 4에서 유효하다면 새로운 x',y'에 대한 검증 findPath 호출

* 이를 더 깔끔하게 변경

```java
boolean findPath(x,y){
    if (x,y) is either on the wall or a visited cell
        return false;
    else if(x,y) is the exit
        return true;
    else
        mark (x,y) as a visited cell;
        for each neighbouring cell (x',y') of (x,y) do
                if findPath(x',y')
                    return true;
    	return false;
}
```

* 대신 순환 함수가 더 호출됨. 대신 코드가 간결 해짐

* 최종 작성된 코드

```java
public class Maze {
	private static int N = 8;
	private static int [][] maze = {
		{0,0,0,0,0,0,0,1},
		{0,1,1,0,1,1,0,1},
		{0,0,0,1,0,0,0,1},
		{0,1,0,0,1,1,0,0},
		{0,1,1,1,0,0,1,1},
		{0,1,0,0,0,1,0,1},
		{0,0,0,1,0,0,0,1},
		{0,1,1,1,0,1,0,0},
	};
	
	private static final int PATH_WAY_COLOUR = 0;
	private static final int WALL_COLOUR = 1;
	private static final int BLOCKED_COLOUR = 2;
	private static final int PATH_COLOUR = 3;
	
	public static void main(String[] argv) {
		printMaze();
		findMazePath(0,0);
		System.out.println();
		printMaze();
	}
	
	public static boolean findMazePath(int x, int y) {
		if(x < 0 || y < 0 || x >= N || y >= N) { // 행렬의 크기 8을 넘었을 경우
			return false;
		} else if(maze[x][y] != PATH_WAY_COLOUR) { //이미 방분했거나 벽인 경우
			return false;
		} else if(x == N-1 && y == N-1) { // maze[7][7]. 즉 출구이면 true
			maze[x][y] = PATH_COLOUR;
			return true;
		} else {
			maze[x][y] = PATH_COLOUR; // 위에 조건을 통과했다는 것은 유효한 셀에 도착
			if(findMazePath(x-1, y) || findMazePath(x, y+1)
				|| findMazePath(x+1, y) || findMazePath(x, y-1)) {
				return true;
			} // 그 다음 인접한 방향(위치, 셀)에 대한 유효성 검사
			maze[x][y] = BLOCKED_COLOUR;
			return false;
		}
	}

	private static void printMaze() {
		for(int i = 0 ; i < N ; i++) {
			for(int j = 0 ; j < N; j++) {
				System.out.print(maze[i][j]);
			}
			System.out.println();
		}
		
	}
}
```

