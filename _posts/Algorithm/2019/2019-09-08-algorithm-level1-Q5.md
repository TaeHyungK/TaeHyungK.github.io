---
layout: post
title:  "[알고리즘] 프로그래머스 > 2016년"
categories: [Algorithm]
tags: [프로그래머스, Level1]
---

> Java API를 알고 있어 쉽게 풀 수 있었던 문제였다.

[프로그래머스 문제](https://programmers.co.kr/learn/courses/30/lessons/12901) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/programmers/level1/PGLevel1Q5.java)

**문제 설명**

  2016년 1월 1일은 금요일입니다. 2016년 a월 b일은 무슨 요일일까요? 
  두 수 a ,b를 입력받아 2016년 a월 b일이 무슨 요일인지 리턴하는 함수, solution을 완성하세요. 
  
  요일의 이름은 일요일부터 토요일까지 각각 "SUN,MON,TUE,WED,THU,FRI,SAT" 입니다. 
  
  예를 들어 a=5, b=24라면 5월 24일은 화요일이므로 문자열 TUE를 반환하세요.

**제한사항**

 - 2016년은 윤년입니다.
 - 2016년 a월 b일은 실제로 있는 날입니다. (13월 26일이나 2월 45일같은 날짜는 주어지지 않습니다)

**입출력 예**
- ![스크린샷 2020-01-04 오후 1.07.22](/img/스크린샷 2020-01-04 오후 1.07.22.png)

**소스 코드**

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

class PGLevel1Q5 {
    public static final String[] week = {"SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"};

    public String solution(int a, int b) {
        String answer = "";

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        try {
            Date date = sdf.parse("2016-" + a + "-" + b);

            Calendar cal = Calendar.getInstance();
            cal.setTime(date);

            int dayNum = cal.get(Calendar.DAY_OF_WEEK);
            answer = week[dayNum - 1];
        } catch (ParseException e) {
            e.printStackTrace();
        }

        System.out.println(answer);

        return answer;
    }
}
```