---
layout: post
title:  "[알고리즘] leetcode > Reverse String 풀이"
categories: [Algorithm, leetcode]
tags: [leetcode, easy]

---

[문제 출처](https://leetcode.com/explore/interview/card/top-interview-questions-easy/127/strings/879/) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/leetcode/ReverseString.java)

String 타입 거꾸로 출력하기

### 최종 풀이

#### 소스 코드

```java
public class ReverseString {
    public static void main(String[] args) {
//        char[] s = {'h','e','l','l','o'};
        char[] s = {'H','a','n','n','a','h'};
        reverseString(s);
    }

    public static void reverseString(char[] s) {
        // 첫번째 풀이
        // left는 증가, right는 감소하면서 서로 자리를 스왑하면 된다.
        // 단, left < right 인 경우 break 한다.
        int left = 0;
        int right = s.length - 1;
        while (left < right) {
            char temp = s[left];
            s[left] = s[right];
            s[right] = temp;

            left++;
            right--;
        }
    }
}
```



#### 풀이

캐릭터 배열 s의 최좌측은 `left`, 최우측은 `right` 로 지정하고 두 index 가 교차될 때까지 while 문을 돌리며 두 index에 해당하는 값들을 swap 한다.

#### 결과

![스크린샷 2020-09-08 오전 12.20.55](/img/스크린샷 2020-09-08 오전 12.20.55.png)

### 회고

이번 문제는 꽤나 알고리즘답게 풀었던 것 같다. 메모리도 저정도면 꽤나 적게 사용하였고, 속도는 가장 빠른 알고리즘으로 풀었다. 이 문제를 풀고 나서 기분이 좋아서 몇 번 돌려보기도 했던 것 같다. 😂