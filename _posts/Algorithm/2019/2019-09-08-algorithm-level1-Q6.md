---
layout: post
title:  "[알고리즘] 프로그래머스 > 가운데 글자 가져오기"
categories: [Algorithm, 프로그래머스, 프로그래머스]
tags: [프로그래머스, Level1]
---

[프로그래머스 문제](https://programmers.co.kr/learn/courses/30/lessons/12903) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/programmers/level1/PGLevel1Q6.java)

**문제 설명**

  단어 s의 가운데 글자를 반환하는 함수, solution을 만들어 보세요. 단어의 길이가 짝수라면 가운데 두글자를 반환하면 됩니다.

**제한사항**

 - s는 길이가 1 이상, 100이하인 스트링입니다.

**입출력 예**
- ![스크린샷 2020-01-04 오후 1.15.04](/img/스크린샷 2020-01-04 오후 1.15.04.png)

**소스 코드**

```java
class PGLevel1Q6 {
    public String solution(String s) {
        String answer = "";
        if (s.length() <= 1) {
            return s;
        }
        int midIndex = s.length() / 2;
        // 짝수인 경우
        if (s.length() % 2 == 0) {
            answer = s.substring(midIndex - 1, midIndex + 1);
        } else { // 홀수인 경우
            answer = s.substring(midIndex, midIndex + 1);
        }
        return answer;
    }
}
```