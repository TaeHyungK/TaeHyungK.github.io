---
layout: post
title:  "[알고리즘] 프로그래머스 > 나누어 떨어지는 숫자 배열"
categories: Algorithm
tags: Algorithm Level1
author: TaeHyungK
---

* content
{:toc}

> Arrays의 stream, filter를 이용하면 불필요한 연산을 하지 않아도 되었던 문제
> 해당 함수를 구현할 줄 안 다음에 비로소 사용하는 것이 좋을 것으로 보인다.

[프로그래머스 문제](https://programmers.co.kr/learn/courses/30/lessons/12910) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/programmers/level1/PGLevel1Q8.java)

**문제 설명**

  array의 각 element 중 divisor로 나누어 떨어지는 값을 오름차순으로 정렬한 배열을 반환하는 함수, solution을 작성해주세요.
  divisor로 나누어 떨어지는 element가 하나도 없다면 배열에 -1을 담아 반환하세요.

**제한사항**

 - arr은 자연수를 담은 배열입니다.
 - 정수 i, j에 대해 i ≠ j 이면 arr[i] ≠ arr[j] 입니다.
 - divisor는 자연수입니다.
 - array는 길이 1 이상인 배열입니다.

**입출력 예**
- ![스크린샷 2020-01-04 오후 1.20.44](/img/스크린샷 2020-01-04 오후 1.20.44.png)

**입출력 예 설명**

- 입출력 예#1
  - arr의 원소 중 5로 나누어 떨어지는 원소는 5와 10입니다. 따라서 [5, 10]을 리턴합니다.

- 입출력 예#2
  - arr의 모든 원소는 1으로 나누어 떨어집니다. 원소를 오름차순으로 정렬해 [1, 2, 3, 36]을 리턴합니다.

- 입출력 예#3
  - 3, 2, 6은 10으로 나누어 떨어지지 않습니다. 나누어 떨어지는 원소가 없으므로 [-1]을 리턴합니다.

**소스 코드**

```java
import java.util.*;

class PGLevel1Q8 {
    public int[] solution(int[] arr, int divisor) {
        int[] answer = Arrays.stream(arr).filter(factor -> factor % divisor == 0).toArray();
        if (answer.length == 0) {
            answer = new int[]{-1};
        }
        Arrays.sort(answer);
        return answer;
		
	      /*ArrayList<Integer> list = new ArrayList<Integer>();
	      
	      Arrays.sort(arr);
	      for (int i = 0; i < arr.length; i++) {
	    	  if (arr[i] % divisor == 0) {
	    		  list.add(arr[i]);
	    	  }
	      }
	      
	      if (list.isEmpty()) { 
	    	  list.add(-1);
	      }
	      
	      System.out.println(list);
	      
	      return convertListToArray(list);*/
    }

    private static int[] convertListToArray(ArrayList<Integer> arrayList) {
        int[] result = new int[arrayList.size()];

        for (int i = 0; i < arrayList.size(); i++) {
            result[i] = arrayList.get(i);
        }

        return result;
    }
}

```