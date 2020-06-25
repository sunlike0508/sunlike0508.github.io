---
title:  "Programmers 네 번째 문제"
excerpt: "예산"
classes: wide
categories:
  - Programmers
tags:
  - Programmers
last_modified_at: 2020-06-24
---

#### [예산](https://programmers.co.kr/learn/courses/30/lessons/12982)

```java
public int solution(int[] d, int budget) {

    Arrays.sort(d);
    int count = 0;

    for(int i = 0; i < d.length; i++) {
        budget -=d[i];

        if(budget < 0) {
            break;
        }

        count++;
    }

    return count;
}
```

* 문제는 어렵지 않았다.
* 그런데 나는 접근을 너무 어렵게 생각했다. 모든 조합을 다 구해서 각 합에 대해 맞는 것들 중 가장 많은 부서가 많은 조합을 추출하려고 했다.
* 1시간을 생각하다가 이렇게 어렵게 풀게 아니라고 생각이 들어서 문제를 다시 보았다.
* 그랬더니 가장 작은 수들로 합이 가장 많은 부서들이 포함될 수 있다는 것을 알게 되었다.
* 후.. 문제를 처음부터 잘 해석해야겠다.
