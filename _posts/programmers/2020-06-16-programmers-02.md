---
title:  "Programmers 두 번째 문제"
excerpt: "카카오프렌즈 컬러링북"
classes: wide
categories:
  - Programmers
tags:
  - Programmers
last_modified_at: 2020-06-16
---

#### [카카오프렌즈 컬러링북](https://programmers.co.kr/learn/courses/30/lessons/1829)

```java
private static final int VISIT_COLOR = -1;
private static final int EMPTY_COLOR = 0;

public int[] solution(int m, int n, int[][] picture) {
    int maxSizeOfOneArea = 0;
    int numberOfArea = 0;

    int[][] book = new int[m][n];

    for(int i = 0; i < picture.length; i++) {
        System.arraycopy(picture[i], 0, book[i], 0, picture[i].length);
    }

    for(int i = 0; i < book.length; i++) {

        for(int j = 0; j < book[i].length; j++) {

            int count = countMaxSizeOfArea(book, i, j, book[i][j]);

            if(count > 0) {
                numberOfArea ++;
            }

            maxSizeOfOneArea = Math.max(maxSizeOfOneArea, count);
        }
    }

    int[] answer = new int[2];
    answer[0] = numberOfArea;
    answer[1] = maxSizeOfOneArea;

    return answer;
}

private int countMaxSizeOfArea(int[][] book, int x, int y, int color) {

    if(x < 0 || y < 0 || x >= book.length || y >= book[x].length) {
        return 0;
    } else if(book[x][y] != color || book[x][y] == VISIT_COLOR || book[x][y] == EMPTY_COLOR) {
        return 0;
    } else {
        book[x][y] = VISIT_COLOR;

        return 1 + countMaxSizeOfArea(book,x,y+1,color) 
            + countMaxSizeOfArea(book,x-1,y,color) 
            + countMaxSizeOfArea(book,x+1,y,color) 
            + countMaxSizeOfArea(book,x,y-1,color); 
    }
}
```

* 그 동안 공부해왔던 Recursion이 발휘되었다.
* 일단 다들 똑같이 배열을 순회하는 것으로 시작했다.
* 그리고 Recursive 함수(countMaxSizeOfArea)로 해당 위치에 있는 영역을 count하였다.
* 이때 countMaxSizeOfArea함수 인자로는 book (picture 배열 복사. 왜 복사 했는지는 마지막 설명), 그리고 해당 위치(x, y), 그리고 해당 위치의 영역 색깔(숫자)를 보낸다.
* 그리고 상하좌우 돌면서 해당 위치가 영역 색깔과 같을 경우들을 다 더해서 return한다. 그리고 방문한 배열은 -1로 바꾼다. (영역은 0이상이라고 했으니까 -1해도 상관 없음)
* 방문한 배열이 방문했던 배열이거나 영역이 아닌 배열(색깔이 없거나 다른 색깔인 경우)이면 0을 return.
* 그런데 문제는 문제를 봤을 때, m과 n은 굳이 필요가 없다. 그리고 pictures를 굳이 복사해야 했나? 라는 생각이 든다. 
* 이게 웃긴게 당연히 다 필요가 없다. 더욱이 pictures을 굳이 복사안해도 풀 수 있는데 이게 사이트에서는 이렇게 안하면 통과를 안해준다. 암튼. 재밌는 문제였다.
