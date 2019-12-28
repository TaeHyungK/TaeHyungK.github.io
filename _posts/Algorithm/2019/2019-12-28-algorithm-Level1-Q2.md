---
layout: post
title:  "[알고리즘] 모의고사"
categories: Algorithm
tags: Algorithm Level1
author: TaeHyungK
---

* content
{:toc}

[프로그래머스 문제](https://programmers.co.kr/learn/courses/30/lessons/42840) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/programmers/level1/PGLevel1Q2.java)

**문제 설명**

  - 수포자는 수학을 포기한 사람의 준말입니다. 수포자 삼인방은 모의고사에 수학 문제를 전부 찍으려 합니다. 수포자는 1번 문제부터 마지막 문제까지 다음과 같이 찍습니다.
    - 1번 수포자가 찍는 방식: 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, ...
    - 2번 수포자가 찍는 방식: 2, 1, 2, 3, 2, 4, 2, 5, 2, 1, 2, 3, 2, 4, 2, 5, ...
    - 3번 수포자가 찍는 방식: 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, ...
    - 1번 문제부터 마지막 문제까지의 정답이 순서대로 들은 배열 answers가 주어졌을 때, 가장 많은 문제를 맞힌 사람이 누구인지 배열에 담아 return 하도록 solution 함수를 작성해주세요.

**제한사항**

 - 시험은 최대 10,000 문제로 구성되어있습니다.
 - 문제의 정답은 1, 2, 3, 4, 5중 하나입니다.
 - 가장 높은 점수를 받은 사람이 여럿일 경우, return하는 값을 오름차순 정렬해주세요.


**입출력 예**
![스크린샷 2019-12-28 오후 7.29.17](/img/스크린샷 2019-12-28 오후 7.29.17.png)

**입출력 예 설명**

- 입출력 예 #1
  - 수포자 1은 모든 문제를 맞혔습니다.
  - 수포자 2는 모든 문제를 틀렸습니다.
  - 수포자 3은 모든 문제를 틀렸습니다.
  - 따라서 가장 문제를 많이 맞힌 사람은 수포자 1입니다.

- 입출력 예 #2
  - 모든 사람이 2문제씩을 맞췄습니다.

**소스 코드**

```
import java.util.ArrayList;

class PGLevel1Q2 {
    public static int[] solution(int[] answers) {
        ArrayList<Integer> scores = new ArrayList<Integer>();
        ArrayList<Integer> result = new ArrayList<Integer>();

        int[] a = {1, 2, 3, 4, 5}; // 5
        int[] b = {2, 1, 2, 3, 2, 4, 2, 5}; // 8
        int[] c = {3, 3, 1, 1, 2, 2, 4, 4, 5, 5}; // 10

        int scoreA = getScore(a, answers);
        scores.add(getScore(a, answers));
        int scoreB = getScore(b, answers);
        scores.add(getScore(b, answers));
        int scoreC = getScore(c, answers);
        scores.add(getScore(c, answers));

        int maxScore = Math.max(scoreA, Math.max(scoreB, scoreC));

        for (int i = 0; i < scores.size(); i++) {
            if (scores.get(i) == maxScore) {
                result.add(i + 1);
            }
        }

        return result.stream().mapToInt(i -> i).toArray();
    }

    public static int getScore(int[] check, int[] answers) {
        int score = 0;

        for (int i = 0; i < answers.length; i++) {
            if (check[i % check.length] == answers[i % answers.length]) {
                score++;
            }
        }

        return score;
    }
}
```