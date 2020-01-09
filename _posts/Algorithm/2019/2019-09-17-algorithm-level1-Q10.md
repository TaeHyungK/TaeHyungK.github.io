---
layout: post
title:  "[알고리즘] 프로그래머스 > 문자열 내 마음대로 정렬하기"
categories: Algorithm
tags: Algorithm Level1
author: TaeHyungK
---

* content
{:toc}

[프로그래머스 문제](https://programmers.co.kr/learn/courses/30/lessons/12915) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/programmers/level1/PGLevel1Q10.java) 

**문제 설명**

  - 문자열로 구성된 리스트 strings와, 정수 n이 주어졌을 때, 각 문자열의 인덱스 n번째 글자를 기준으로 오름차순 정렬하려 합니다. 
  <br>예를 들어 strings가 [sun, bed, car]이고 n이 1이면 각 단어의 인덱스 1의 문자 u, e, a로 strings를 정렬합니다.

**제한사항**

 - strings는 길이 1 이상, 50이하인 배열입니다.
 - strings의 원소는 소문자 알파벳으로 이루어져 있습니다.
 - strings의 원소는 길이 1 이상, 100이하인 문자열입니다.
 - 모든 strings의 원소의 길이는 n보다 큽니다.
 - 인덱스 1의 문자가 같은 문자열이 여럿 일 경우, 사전순으로 앞선 문자열이 앞쪽에 위치합니다.

**입출력 예**
- ![스크린샷 2020-01-09 오후 9.59.39](/img/스크린샷 2020-01-09 오후 9.59.39.png)


**입출력 예 설명**

- 입출력 예 1
  - sun, bed, car의 1번째 인덱스 값은 각각 u, e, a 입니다. 이를 기준으로 strings를 정렬하면 [car, bed, sun] 입니다.

- 입출력 예 2
  - abce와 abcd, cdx의 2번째 인덱스 값은 c, c, x입니다. 따라서 정렬 후에는 cdx가 가장 뒤에 위치합니다. abce와 abcd는 사전순으로 정렬하면 abcd가 우선하므로, 답은 [abcd, abce, cdx] 입니다.
  
**소스 코드**

```java
import java.util.*;

class PGLevel1Q10 {
    public String[] solution(String[] strings, int n) {
        Arrays.sort(strings);

        ArrayList<String> list = new ArrayList<String>(Arrays.asList(strings));
        Collections.sort(list, new MyComparator(n));

        System.out.println(list);

        return convertListToArray(list);
    }

    private class MyComparator implements Comparator<String> {
        int mNumber;

        public MyComparator(int n) {
            mNumber = n;
        }

        @Override
        public int compare(String first, String second) {
            char charFirst = first.charAt(this.mNumber);
            char charSecond = second.charAt(this.mNumber);

            if (charFirst > charSecond) {
                return 1;
            } else if (charFirst < charSecond) {
                return -1;
            } else {
                return 0;
            }
        }
    }

    private String[] convertListToArray(ArrayList<String> arrayList) {
        String[] result = new String[arrayList.size()];

        for (int i = 0; i < arrayList.size(); i++) {
            result[i] = arrayList.get(i);
        }

        return result;
    }
}
```