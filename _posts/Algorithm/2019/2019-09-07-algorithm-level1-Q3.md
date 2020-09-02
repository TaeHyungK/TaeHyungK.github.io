---
layout: post
title:  "[알고리즘] 프로그래머스 > 체육복"
categories: [Algorithm, 프로그래머스]
tags: [프로그래머스, Level1]
---

> 예외가 많아 쉽지 않았던 문제. 3중 for문 사용으로 복잡도가 너무 높게 푼 문제 같다...
> 추후 리팩토링으로 복잡도를 낮추면 더 좋을 것 같다.

[프로그래머스 문제](https://programmers.co.kr/learn/courses/30/lessons/42862) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/programmers/level1/PGLevel1Q3.java)

**문제 설명**

  - 점심시간에 도둑이 들어, 일부 학생이 체육복을 도난당했습니다. 다행히 여벌 체육복이 있는 학생이 이들에게 체육복을 빌려주려 합니다. 학생들의 번호는 체격 순으로 매겨져 있어, 바로 앞번호의 학생이나 바로 뒷번호의 학생에게만 체육복을 빌려줄 수 있습니다. 예를 들어, 4번 학생은 3번 학생이나 5번 학생에게만 체육복을 빌려줄 수 있습니다. 체육복이 없으면 수업을 들을 수 없기 때문에 체육복을 적절히 빌려 최대한 많은 학생이 체육수업을 들어야 합니다.
  
  - 전체 학생의 수 n, 체육복을 도난당한 학생들의 번호가 담긴 배열 lost, 여벌의 체육복을 가져온 학생들의 번호가 담긴 배열 reserve가 매개변수로 주어질 때, 체육수업을 들을 수 있는 학생의 최댓값을 return 하도록 solution 함수를 작성해주세요.

**제한사항**

 - 전체 학생의 수는 2명 이상 30명 이하입니다.
 - 체육복을 도난당한 학생의 수는 1명 이상 n명 이하이고 중복되는 번호는 없습니다.
 - 여벌의 체육복을 가져온 학생의 수는 1명 이상 n명 이하이고 중복되는 번호는 없습니다.
 - 여벌 체육복이 있는 학생만 다른 학생에게 체육복을 빌려줄 수 있습니다.
 - 여벌 체육복을 가져온 학생이 체육복을 도난당했을 수 있습니다. 이때 이 학생은 체육복을 하나만 도난당했다고 가정하며, 남은 체육복이 하나이기에 다른 학생에게는 체육복을 빌려줄 수 없습니다.

**입출력 예**
- ![스크린샷 2019-12-29 오후 1.00.38](/img/스크린샷 2019-12-29 오후 1.00.38.png)

**입출력 예 설명**

- 예제 #1
  - 1번 학생이 2번 학생에게 체육복을 빌려주고, 3번 학생이나 5번 학생이 4번 학생에게 체육복을 빌려주면 학생 5명이 체육수업을 들을 수 있습니다.

- 예제 #2
  - 3번 학생이 2번 학생이나 4번 학생에게 체육복을 빌려주면 학생 4명이 체육수업을 들을 수 있습니다.

**소스 코드**

```java
import java.util.ArrayList;

class PGLevel1Q3 {
    public int solution(int n, int[] lost, int[] reserve) {

        int ll = lost.length;
        int rl = reserve.length;
        ArrayList<Integer> ok = new ArrayList<Integer>();

        for (int i = 0; i < ll; i++) {
            for (int j = 0; j < rl; j++) {
                if (lost[i] == reserve[j]) {
                    for (int k = j; k < rl - 1; k++) {
                        reserve[k] = reserve[k + 1];
                    }
                    for (int l = i; l < ll - 1; l++) {
                        lost[l] = lost[l + 1];
                    }
                    ll--;
                    rl--;
                }
            }
        }

        for (int i = 0; i < ll; i++) {
            int j = 0;

            while (j < rl) {
                if (ok.size() == 0 || !ok.contains(reserve[j])) {
                    if ((lost[i] + 1) == reserve[j] || (lost[i] - 1) == reserve[j]) {
                        ll--;
                        // System.out.println("after answer=" + answer + "j=" + j);
                        ok.add(reserve[j]);
                        break;
                    }
                    j++;
                } else {
                    j++;
                }
            }
        }
        return n - ll;
    }
}
```