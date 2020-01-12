---
layout: post
title:  "[알고리즘] 구름 > 시공의 폭풍 속으로"
categories: Algorithm
tags: 구름EDU
author: TaeHyungK
---

* content
{:toc}

[문제 출처](https://edu.goorm.io/learn/lecture/15551/%EB%A7%A4%EC%A3%BC-%EB%B0%B0%EC%86%A1%EB%B0%9B%EB%8A%94-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%ED%94%84%EB%A6%AC%EB%AF%B8%EC%97%84-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EC%9C%84%ED%81%B4%EB%A6%AC-%EB%B9%84%ED%83%80%EC%95%8C%EA%B3%A0-%EC%8B%9C%EC%A6%8C2/lesson/726506/09%EC%9B%94-1%EC%A3%BC%EC%B0%A8-%EC%8B%9C%EA%B3%B5%EC%9D%98-%ED%8F%AD%ED%92%8D-%EC%86%8D%EC%9C%BC%EB%A1%9C-1) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/goorm/Q726506.java)

> 무식하게 2중 포문의 사용으로 시간 복잡도가 n의 제곱으로 풀었다.
> O(n)으로 풀 수 있는 방법을 더 생각해서 풀면 좋을 것 같다..

**소스 코드**

```java
import java.io.*;

public class Q726506 {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String[] curTeam = br.readLine().split(" ");
        String[] heros = br.readLine().split(" ");

        int count = 0;
        for (String member : curTeam) {
            for (String hero : heros) {
                if (member.equals(hero)) {
                    count++;
                }
            }
        }

        System.out.println(heros.length - count);
    }
}

```