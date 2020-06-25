---
title:  "Programmers 다섯 번째 문제"
excerpt: "다트 게임"
classes: wide
categories:
  - Programmers
tags:
  - Programmers
last_modified_at: 2020-06-25
---

#### [다트 게임](https://programmers.co.kr/learn/courses/30/lessons/17682)

```java
public int solution(String dartResult) {

    Pattern dartToken = Pattern.compile("\\d{1,2}\\D[\\*#]?");
    Matcher dartMatcher = dartToken.matcher(dartResult);
    List<Integer> pointList = new ArrayList<Integer>();

    while (dartMatcher.find()) {
        String pointGroup = dartMatcher.group();

        int point = Integer.parseInt(pointGroup.replaceAll("[^0-9]", ""));
        String bouns = pointGroup.replaceAll("[^(S|D|T)]", "");
        String option = pointGroup.replaceAll("[0-9][S|D|T]", "");

        if(!"S".equals(bouns)) {
            point = (int) Math.pow(point, "D".equals(bouns) ? 2 : 3);
        }

        if("#".equals(option)) {
            point *= -1;
        }

        if("*".equals(option)) {
            point *= 2;

            if(pointList.size() > 0) {
                pointList.set(pointList.size()-1, 
                              pointList.get(pointList.size()-1) * 2);
            }
        }

        pointList.add(point);
    }

    int answer = 0;

    for(int point : pointList) {
        answer += point;
    }

    return answer;
}
```

* 정규식이 없으면 정말 if문의 향연일듯 하다. 실제로도 다른 사람들의 풀이를 봤는데 정말 if문 밖에 안보였다.
* 아직도 정규식에 대해서 잘 모르지만 그래도 이정도까진 정규식으로 풀 수 있어서 실력이 올라가는 기분이 든다.
