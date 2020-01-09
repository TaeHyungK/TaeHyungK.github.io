---
layout: post
title:  "[알고리즘] 프로그래머스 > 두 정수 사이의 합"
categories: Algorithm
tags: 프로그래머스 Level1
author: TaeHyungK
---

* content
{:toc}

[프로그래머스 문제](https://programmers.co.kr/learn/courses/30/lessons/12912) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/programmers/level1/PGLevel1Q9.java) 

**문제 설명**

  - 두 정수 a, b가 주어졌을 때 a와 b 사이에 속한 모든 정수의 합을 리턴하는 함수, solution을 완성하세요.
    <br>예를 들어 a = 3, b = 5인 경우, 3 + 4 + 5 = 12이므로 12를 리턴합니다.

**제한사항**

 - a와 b가 같은 경우는 둘 중 아무 수나 리턴하세요.
 - a와 b는 -10,000,000 이상 10,000,000 이하인 정수입니다.
 - a와 b의 대소관계는 정해져있지 않습니다.



**입출력 예**
- ![스크린샷 2020-01-09 오후 9.55.17](/img/스크린샷 2020-01-09 오후 9.55.17.png)

**소스 코드**

```java
class PGLevel1Q9 {
    public long solution(int a, int b) {
        long answer = 0;
        if (a > b) {
            int temp = a;
            a = b;
            b = temp;
        }
        for (int i = a; i <= b; i++) {
            answer += i;
        }
        return answer;
    }
}
```