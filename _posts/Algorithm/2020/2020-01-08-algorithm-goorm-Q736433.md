---
layout: post
title:  "[알고리즘] 구름 > 수의 비밀"
categories: [Algorithm, goorm]
tags: [구름EDU]
---

[문제 출처](https://edu.goorm.io/learn/lecture/15551/%EB%A7%A4%EC%A3%BC-%EB%B0%B0%EC%86%A1%EB%B0%9B%EB%8A%94-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%ED%94%84%EB%A6%AC%EB%AF%B8%EC%97%84-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EC%9C%84%ED%81%B4%EB%A6%AC-%EB%B9%84%ED%83%80%EC%95%8C%EA%B3%A0-%EC%8B%9C%EC%A6%8C2/lesson/736433/09%EC%9B%94-1%EC%A3%BC%EC%B0%A8-%EC%88%98%EC%9D%98-%EB%B9%84%EB%B0%80-1) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/goorm/Q736433.java)

> 회귀를 이용해 입력받은 값을 계속 나누는 방법으로 문제를 해결하였다.
> 처음 10^18 까지 입력이 가능하다는 것을 보고 아무 생각이 풀었기에 int type으로 선언하여 풀었다가 RuntimeError가 발생하였다.
> int 형의 범위를 초과하는 입력값일 경우 당연히 고려해야할 사항이다. 반성하자!
>
> ***해설을 본 후)***
> 
> bit 연산자를 이용할 경우 회귀 없이 input 값 그대로 비트 연산만을 통해 값을 도출해 낼 수 있다는 것이 새삼 대단하다고 느껴졌다.
> 2^k인 경우 비트가 **"1인 bit가 1개"** 있다. 라는 점을 파악할 줄 알아야 한다.
> 비트 연산의 경우 주로 쓰지 않아서 복잡한 것 같다.. 코드는 확실히 간결해지긴 하나 쉽지 않다..
>
> input이 34인 경우,
>
> input = 0 0 1 0 0 0 1 0
>
> & -input = 1 1 0 1 1 1 1 0
>
>         0 0 0 0 0 0 1 0
>         0 0 1 0 0 0 1 0 
>         output => "No"
> input이 32인 경우,
>
>  input = 0 0 1 0 0 0 0 0
>
> & -input = 0 0 1 0 0 0 0 0
>
>          0 0 1 0 0 0 0 0
>          0 0 1 0 0 0 0 0 
>          output => "Yes"
>
> -(input)은 NOT 연산을 한 후 1을 더함
> -> 즉, 모든 비트를 반대로 바꾸고 1을 더해주면 됨

**소스 코드**

```java
import java.io.*;

public class Q736433 {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        long input = Long.parseLong(br.readLine());

        long value = devider(input);

        System.out.println(value == 1 ? "Yes" : "No");

        // bit 연산자를 이용하여 풀이
        System.out.println("bit : " + (input == (input & -(input)) ? "Yes" : "No"));
    }

    private static long devider(long value) {
        if (value == 1 || value % 2 != 0) {
            return value;
        }
        return devider(value / 2);
    }
}
```