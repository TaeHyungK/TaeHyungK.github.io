---
layout: post
title:  "[알고리즘] 구름 > 애틋한 친구"
categories: [Algorithm, goorm]
tags: [구름EDU]
---

[문제 출처](https://edu.goorm.io/learn/lecture/15551/%EB%A7%A4%EC%A3%BC-%EB%B0%B0%EC%86%A1%EB%B0%9B%EB%8A%94-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%ED%94%84%EB%A6%AC%EB%AF%B8%EC%97%84-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EC%9C%84%ED%81%B4%EB%A6%AC-%EB%B9%84%ED%83%80%EC%95%8C%EA%B3%A0-%EC%8B%9C%EC%A6%8C2/lesson/761937/09%EC%9B%94-2%EC%A3%BC%EC%B0%A8-%EC%95%A0%ED%8B%8B%ED%95%9C-%EC%B9%9C%EA%B5%AC-1) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/goorm/Q761937.java)

> 순간 두 평면 좌표 사이의 거리 구하는 공식이 생각이 나지 않아 당황했었던 문제다.(~~수학 공부도 추가로 더 해야되는 것인가..?~~)
> <br>double 타입을 정수형으로 바꿀 때에 오차가 생길 수 있다는 점과 이 문제에서는 정확한 거리를 구하는 것이 아니기 때문에 굳이 두 좌표 간의 거리를 구할 때에 루트를 씌우지 않아도 된다는 점을 생각해야 쉽게 풀 수 있는 문제였다.
> 
> 확실히 아직은 알고리즘 실력이 부족한 것 같다. 완전 탐색으로 푸는 것부터 생각나는 것을 보니...😭
>
> ***해설을 본 후)***
> 
> 해설도 완전 탐색을 사용하고 있다. 완전 탐색을 사용하더라도 이미 비교한 경우에 continue를 통해 스킵할 수 있는 경우를 생각하면 더 빠른 처리를 할 수 있을 것 같다.

**소스 코드**

```java
import java.io.*;
import java.util.HashMap;

public class Q761937 {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int count = Integer.parseInt(br.readLine());
        HashMap<Integer, String[]> friends = new HashMap<Integer, String[]>();

        for (int i = 0; i < count; i++) {
            String[] input = br.readLine().split(" ");
            friends.put(i, input);
        }

        long maxDistance = 0;
        int answer1 = -1;
        int answer2 = -1;
        // 완전 탐색
        for (int i = 0; i < friends.size(); i++) {
            for (int j = 0; j < friends.size(); j++) {
                // 같은 위치는 무시
                if (j == i) continue;

                int x1 = Integer.parseInt(friends.get(i)[0]);
                int y1 = Integer.parseInt(friends.get(i)[1]);
                int x2 = Integer.parseInt(friends.get(j)[0]);
                int y2 = Integer.parseInt(friends.get(j)[1]);
                long curDistance = getDistance(x1, y1, x2, y2);

                boolean isChanged = maxDistance < curDistance;
                if (isChanged) {
                    answer1 = i;
                    answer2 = j;
                    maxDistance = curDistance;
                }
            }
        }

        System.out.println((answer1 + 1) + " " + (answer2 + 1));
    }

    private static long getDistance(int x1, int y1, int x2, int y2) {
        // Math.pow(): 제곱
        // Math.abs(): 절대값
        long result = (long) (Math.pow(Math.abs(x2-x1), 2) + Math.pow(Math.abs(y2-y1), 2));
        return result;
    }
}
```