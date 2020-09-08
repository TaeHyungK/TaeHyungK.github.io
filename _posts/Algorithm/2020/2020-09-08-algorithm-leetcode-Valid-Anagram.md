---
layout: post
title:  "[알고리즘] leetcode > Valid Anagram 풀이"
categories: [Algorithm, leetcode]
tags: [leetcode, easy]

---

[문제 출처](https://leetcode.com/explore/interview/card/top-interview-questions-easy/127/strings/882/) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/leetcode/ValidAnagram.java)

애너그램(Anagram) 찾기

애너그램 문제는 알고리즘에서 꽤 자주 나오므로 용어를 한번 정리해두려고 한다.

> **애너그램**이란?
> 
> 하나의 문자열의 문자들을 섞었을 때 다른 단어가 나오는 것을 의미한다.
> 예를 들면, "국왕" 과 "왕국" 은 애너그램이다.   

### 최종 풀이

#### 소스 코드
```java
public class ValidAnagram {
    public static void main(String[] args) {
//        String s = "anagram";
//        String t = "nagaram";

        String s = "cat";
        String t = "car";

        System.out.println("result: " + isAnagram(s, t));
    }

    public static boolean isAnagram(String s, String t) {
        boolean result = true;
        if (s.length() != t.length()) return false;

        char[] sArray = s.toCharArray();
        char[] tArray = t.toCharArray();
        Arrays.sort(sArray);
        Arrays.sort(tArray);

        for (int i = 0; i < sArray.length; i++) {
            char sChar = sArray[i];
            char tChar = tArray[i];

            if (sChar != tChar) {
                result = false;
                break;
            }
        }
        return result;
    }
}
```

#### 풀이

먼저, 애너그램은 글자 수의 길이가 같아야하기 때문에 글자수를 먼저 체크해준다.
```java
if (s.length() != t.length()) return false;
```
 
그 후 각 문자열을 `char[]` 로 만들어 준 후 두 값을 for 문을 이용해 비교한다.
```java
for (int i = 0; i < sArray.length; i++) {
    char sChar = sArray[i];
    char tChar = tArray[i];

    if (sChar != tChar) {
        result = false;
        break;
    }
}
```

단, 정렬을 해주어야 정상적으로 비교가 되므로 정렬도 잊지말고 해주어야 한다.
```java
Arrays.sort(sArray);
Arrays.sort(tArray);
```

#### 결과

![Valid Anagram_result](/img/Valid Anagram_result.png)

### 회고

첫번째 풀이임에도 꽤나 높은 효율성이 나와서 꽤 만족스러운 풀이 방법이었던 것 같다. 

이렇게 브레인 스토밍으로 잘 풀리는 문제들만 만나면 알고리즘도 꽤나 재밌는 학문일텐데..😢 

