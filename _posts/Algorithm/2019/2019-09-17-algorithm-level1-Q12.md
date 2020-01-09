---
layout: post
title:  "[알고리즘] 프로그래머스 > 문자열 내림차순으로 배치하기"
categories: Algorithm
tags: Algorithm Level1
author: TaeHyungK
---

* content
{:toc}

[프로그래머스 문제](https://programmers.co.kr/learn/courses/30/lessons/12917) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/programmers/level1/PGLevel1Q11.java) 

**문제 설명**

  - 문자열 s에 나타나는 문자를 큰것부터 작은 순으로 정렬해 새로운 문자열을 리턴하는 함수, solution을 완성해주세요.
    <br>s는 영문 대소문자로만 구성되어 있으며, 대문자는 소문자보다 작은 것으로 간주합니다.

**제한사항**

 - str은 길이 1 이상인 문자열입니다.

**입출력 예**
- ![스크린샷 2020-01-09 오후 10.40.15](/img/스크린샷 2020-01-09 오후 10.40.15.png)

**소스 코드**

```java
import java.util.*;

class PGLevel1Q12 {
    public String solution(String s) {
        String answer = "";
        String[] splitS = s.split("");
        List<String> listS = Arrays.asList(splitS);
        Collections.sort(listS);
        Collections.reverse(listS);

        for (int i = 0; i < listS.size(); i++) {
            answer += listS.get(i);
        }

        return answer;
    }
}
```