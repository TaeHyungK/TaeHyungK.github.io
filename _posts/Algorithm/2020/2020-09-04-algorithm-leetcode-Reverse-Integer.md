---
layout: post
title:  "[알고리즘] leetcode > Reverse Integer 풀이"
categories: [Algorithm, leetcode]
tags: [leetcode, easy]

---

[문제 출처](https://leetcode.com/explore/interview/card/top-interview-questions-easy/127/strings/880/) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/leetcode/ReverseInteger.java)

int 타입 거꾸로 출력하기

### 최종 풀이

#### 소스 코드

```java
public class ReverseInteger {
    public static void main(String[] args) {
//        int x = 123;
//        int x = -123;
        int x = 120;
        System.out.println("result: " + reverse(x));
    }

    public static int reverse(int x) {
        int result = 0;
        String[] splitX = String.valueOf(x).split("");
        boolean isNegaitve = false;
        if ("-".equals(splitX[0])) {
            isNegaitve = true;
        }

        StringBuilder strBuilder = new StringBuilder();
        int lastIdx = isNegaitve ? 1 : 0;
        for (int i = splitX.length - 1; i >= lastIdx; i--) {
            System.out.println(splitX[i]);
            strBuilder.append(splitX[i]);
        }

        System.out.println(strBuilder.toString());
        try {
            result = isNegaitve ? -Integer.parseInt(strBuilder.toString()) : Integer.parseInt(strBuilder.toString());
        } catch (Exception e) {
            result = 0;
        }

        return result;
    }
}
```



#### 풀이

int 타입의 input 데이터를 String[] 로 변환한다.
```java
String[] splitX = String.valueOf(x).split("");
```

이때, 들어온 int 값이 음수인지 체크를 진행한다.
```java
if ("-".equals(splitX[0])) {
    isNegaitve = true;
}
```

위 부분을 조금 더 리팩토링을 해보자면, 아래와 같이 나타낼 수 있다.
문제를 풀 때에 내가 왜... 굳이 String 비교로 체크했는지 모르겠다. (~~내가 부족한 탓이다~~)
```java
if (x < 0) {
    isNegaitve = true;
}
```

이후에는 for 문을 이용해 뒤에서부터 붙여 준 후 음수 유무에 다라 기호룰 붙여주면 끝이다.


#### 결과

![스크린샷 2020-09-08 오전 12.14.33](/img/스크린샷 2020-09-08 오전 12.14.33.png)

### 회고

String 문제에서 int와 같은 정수 타입들을 다루는 게 쉽지가 않은 것 같다. 특히나 속도, 메모리를 효율적으로 사용하면서 하는 데에 부족함이 많은 것 같다. 이번 문제에서도 역시나 효율성이 별로 좋지 않다. 효율적으로 생각하는 힘이 필요한데 그저 풀기만 하는 것 같다..