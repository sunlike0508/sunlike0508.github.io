---
title:  "Programmers 세 번째 문제"
excerpt: "비밀지도"
classes: wide
categories:
  - Programmers
tags:
  - Programmers
last_modified_at: 2020-06-24
---

#### [비밀지도](https://programmers.co.kr/learn/courses/30/lessons/17681)

```java
public String[] solution(int n, int[] arr1, int[] arr2) {

    String[] overlap = new String[n];

    for(int i = 0; i < n ; i++) {
        overlap[i] = 
            String.format("%" + n + "s",Integer.toBinaryString(arr1[i] | arr2[i]));
    }

    return spaceWall(overlap);
}

private String[] spaceWall(String[] overlap) {

    for(int i = 0; i < overlap.length; i++) {
        overlap[i] = overlap[i].replaceAll("1", "#");
        overlap[i] = overlap[i].replaceAll("0", " ");
    }

    return overlap;
}
```

* 문제는 어렵지 않았다.
* 그런데 String.format을 이용했으면 코드가 더 쉬워졌을 텐데 아쉽다.
* 위의 코드는 내가 푼 코드와 format을 이용해서 수정한 코드이다.
