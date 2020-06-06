---
title:  "정렬의 하계"
excerpt: "정렬의 하계"
classes: wide
categories:
  - algorithm
tags:
  - [algorithm, sort]
last_modified_at: 2020-05-25
---

##### 강의 사이트

* http://www.kocw.net/home/search/kemView.do?kemId=1148815



# 정렬의 하계



### 1. 정렬 알고리즘의 유형

* 보통 모든 정렬 알로리즘들은 시간복잡도 O(N log2N)가 제일 좋다.

#### 1.1 Comparison sort

* 데이터들간의 상대적 크기관계만을 이용해서 정렬하는 알고리즘
* 버블, 삽입, 퀵, 힙 정렬 등

#### 1.2 Non-Comparsion sort

* 정렬할 데이터에 대한 사전지식을 이용 - 적용에 제한
* Bucket, Radix



### 2. 정렬 문제의 하한

* 입력된 데이터를 한번씩 다 보기위해서 최소 O(n)의 시간복잡도가 필요하다.
* 합병정렬과 힙정렬 알고리즘들의 시간복잡도는 O(Nlog2N)



### 3. Decision Tress

![]({{site.url}}/assets/images/algo41.PNG)

* 어떤 Comparsion sort도 추상화 시키면 Decision Tree 가 나온다.
* 위 그림은 삽입정렬 알고리즘을 Decision tree 시킨 것
* Leaf 노드(모든 결과 값)의 개수는 N!개. 
* 최악의 경우 시간 복잡도는 트리의 높이이다.
* 트리의 높이(height) >= log2N != O(N log2N)


