---
layout: post
title:  "[알고리즘] 프로그래머스 > 같은 숫자는 싫어"
categories: Algorithm
tags: Algorithm Level1
author: TaeHyungK
---

* content
{:toc}

[프로그래머스 문제](https://programmers.co.kr/learn/courses/30/lessons/12906) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/programmers/level1/PGLevel1Q7.java)

**문제 설명**

  배열 arr가 주어집니다. 배열 arr의 각 원소는 숫자 0부터 9까지로 이루어져 있습니다. 
  이때, 배열 arr에서 연속적으로 나타나는 숫자는 하나만 남기고 전부 제거하려고 합니다. 
  단, 제거된 후 남은 수들을 반환할 때는 배열 arr의 원소들의 순서를 유지해야 합니다. 
  예를 들면,
  
  - arr = [1, 1, 3, 3, 0, 1, 1] 이면 [1, 3, 0, 1] 을 return 합니다.
  - arr = [4, 4, 4, 3, 3] 이면 [4, 3] 을 return 합니다.
  
  배열 arr에서 연속적으로 나타나는 숫자는 제거하고 남은 수들을 return 하는 solution 함수를 완성해 주세요.

**제한사항**

 - 배열 arr의 크기 : 1,000,000 이하의 자연수
 - 배열 arr의 원소의 크기 : 0보다 크거나 같고 9보다 작거나 같은 정수

**입출력 예**
- ![스크린샷 2020-01-04 오후 1.18.00](/img/스크린샷 2020-01-04 오후 1.18.00.png)

**소스 코드**

```java
import java.util.*;

public class PGLevel1Q7 {
    public int[] solution(int[] arr) {
        ArrayList<Integer> list = new ArrayList<>();

        list.add(arr[0]);
        for (int i = 0; i < arr.length - 1; i++) {
            int j = i + 1;
            if (arr[i] != arr[j]) {
                list.add(arr[j]);
            }
        }

        return convertListToArray(list);
    }

    private int[] convertListToArray(ArrayList<Integer> arrayList) {
        int[] result = new int[arrayList.size()];

        for (int i = 0; i < arrayList.size(); i++) {
            result[i] = arrayList.get(i);
        }

        return result;
    }
}
```