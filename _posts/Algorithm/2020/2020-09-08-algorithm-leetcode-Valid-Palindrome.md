---
layout: post
title:  "[알고리즘] leetcode > Valid Palindrome 풀이"
categories: [Algorithm, leetcode]
tags: [leetcode, easy]

---

[문제 출처](https://leetcode.com/explore/interview/card/top-interview-questions-easy/127/strings/883/) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/leetcode/ValidPalindrome.java)

팰린드롬(Palindrome) 찾기

팰린드롬 문제는 알고리즘에서 꽤 자주 나오므로 용어를 한번 정리해두려고 한다.

> **팰린드롬**이란?
> 
> 한국어로는 "회문" 이라고 한다. 말 뜻 그대로 앞으로 읽어도, 뒤로 읽어도 같은 글자를 뜻한다.
> 예를 들면, 한국어로 "토마토" 는 팰린드롬이다.

### 첫번째 풀이

#### 소스 코드
```java
public class ValidPalindrome {
    public static void main(String[] args) {
        String s = "A man, a plan, a canal: Panama";
//        String s = "0P";
        System.out.println("result: " + isPalindrome(s));
    }

    public static boolean isPalindrome(String s) {
        String str = s.replaceAll("[^A-Za-z0-9]", "");
        String reverseStr = new StringBuilder(str).reverse().toString();
        return str.equalsIgnoreCase(reverseStr);
    }
}
```

#### 풀이

약간 치트키와 같은 풀이 방법이다.

먼저 정규표현식을 통해 문자와 숫자를 제외한 값들을 제거한다.

이후 해당 문자열을 `reverse()` 를 통해 뒤집은 후 대소문자 구분 무시를 위해 `equalsIgnoreCase()` 를 통해 비교한다.

### 최종 풀이

#### 소스 코드

```java
public class ValidPalindrome {
    public static void main(String[] args) {
        String s = "A man, a plan, a canal: Panama";
//        String s = "0P";
        System.out.println("result: " + isPalindrome(s));
    }

    public static boolean isPalindrome(String s) {
        if (s.length() <= 1) return true;
        int left = 0;
        int right = s.length() - 1;
        while (left < right) {
            while (left < s.length() && !(Character.isLetterOrDigit(s.charAt(left)))) {
                left++;
            }
            while (right >= 0 && !(Character.isLetterOrDigit(s.charAt(right)))) {
                right--;
            }
            if (left >= right) break;

            char leftChar = s.charAt(left);
            char rightChar = s.charAt(right);
            if (Character.toLowerCase(leftChar) != Character.toLowerCase(rightChar)) {
                return false;
            }
            left++;
            right--;
        }
        return true;
    }
}
```

#### 풀이

솔직히 팰린드롬 문제는 예전에 이직 준비를 할 때에 거의 외우다시피 했던 적이 있어서 기억을 더듬어서 풀었다.

left, right를 두고 `while` 문을 통해 서로가 교차하기 전까지 반복한다.

숫자와 문자를 제외한 값은 비교대상이 되지 않기 때문에 아래와 같은 코드를 통해 넘어가도록 처리한다.

```java
while (left < s.length() && !(Character.isLetterOrDigit(s.charAt(left)))) {
    left++;
}
while (right >= 0 && !(Character.isLetterOrDigit(s.charAt(right)))) {
    right--;
}
```

left와 right에 대한 index 를 구한 후에는 그 값이 유효한 값인지(서로 교차되지 않았는지)를 체크한다.

```java
if (left >= right) break;
```

도출해낸 left와 right를 통해 해당 index 에 있는 값을 서로 비교해 같지 않은 경우 `false`를 반환하며 종료하고 같을 경우 반복한다.

#### 결과

![Valid Palindrome_result](/img/Valid Palindrome_result.png)

### 회고

예전에 어느 스타트업에 면접을 보러 가기 전에 기출문제를 풀다가 팰린드롬을 외우다시피 했던 경험이 있어서 수월하게 풀었던 문제였다.

또 알고리즘을 안풀어버릇 하면 잊어버릴테니.. 알고리즘 꾸준히 풀수 있도록 의미 부여가 되는 문제였다.