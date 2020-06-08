---
title:  "Programmers 첫 번째 문제"
excerpt: "스킬트리"
classes: wide
categories:
  - Programmers
tags:
  - Programmers
last_modified_at: 2020-06-07
---

#### [스킬트리](https://programmers.co.kr/learn/courses/30/lessons/49993)

```java
public int solution(String skill, String[] skill_trees) {
    int answer = 0;

    for (String skill_tree : skill_trees) {

        skill_tree = skill_tree.replaceAll("[^" + skill + "]", "");

        if(skill_tree.equals(skill.substring(0, skill_tree.length()))) {
            answer++;
        }
    }

    return answer;
}
```

* 그 동안 코드워즈를 풀면서 익혀왔던 스킬(?)들이 발휘 되는 순간이었다.
* 다른 스킬들은 불필요하므로 정규식으로 삭제를 한다. 그리고 항상 첫 스킬부터 맞추어야 하기 때문에 skil_tree이 길이만큼 skill의 첫번째부터 substring하여 같은 문자이면 가능한 스킬 트리이다.
* 굿!
