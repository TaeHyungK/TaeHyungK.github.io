---
layout: post
title:  "[알고리즘] leetcode > Best Time to Buy and Sell Stock II 풀이"
categories: [Algorithm, leetcode]
tags: [leetcode, easy]

---

[문제 출처](https://leetcode.com/explore/interview/card/top-interview-questions-easy/92/array/564/) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/leetcode/BestTimetoBuyandSellStockII.java)

가장 큰 이득을 볼 수 있는 타이밍에 주식 구매/판매 하기

### 최종 풀이

#### 소스 코드

```java
public class BestTimetoBuyandSellStockII {
    public static void main(String[] args) {
        int[] prices = {7,1,5,3,6,4};
//        int[] prices = {1,2,3,4,5};
//        int[] prices = {7,6,4,3,1};
//        int[] prices = {6,1,3,2,4,7};
        System.out.println("benefit: " + maxProfit(prices));
    }

    public static int maxProfit(int[] prices) {
        int benefit = 0;
        int lastIndex = prices.length - 1;
        int index = 0;
        for (int i = 0; i < lastIndex; i++) {
            int curIndex = index;
            if (prices[i] > prices[i + 1]) {
                index = i + 1;
            } else if (i == lastIndex - 1 &&
                    prices[index] < prices[lastIndex]) {
                index = lastIndex + 1;
            }

            System.out.println();
            if (curIndex != index) {
                benefit += prices[index - 1] - prices[curIndex];
            }
        }

        if (benefit < 0) benefit = 0;

        return benefit;
    }
}
```



#### 풀이

오늘은 문제를 꼼꼼히 읽어보고 충분히 생각한 다음 문제를 풀어나가기 시작했다.

핵심은 **떨어지기 직전까지의 값 중 가장 싼 값에 구매해서 떨어지기 직전에 판매** 하는 것이다.

#### 결과

![스크린샷 2020-09-03 오후 10.43.49](/img/스크린샷 2020-09-03 오후 10.43.49.png)

![스크린샷 2020-09-03 오후 10.44.41](/img/스크린샷 2020-09-03 오후 10.44.41.png)

### 회고

운이 좋게도(?) 한 번에 푼 답이 맞기는 했지만 결과를 보면 알 수 있다시피 더 효율적으로 생각을 한다면 Runtime, Memory 사용량 모두 더 줄일 수 있다. 심지어 비슷한 Runtime, Memory 사용을 하는 코드들도 내가 짠 것 보다 더 가독성이 좋게 짰다. 특히나 변수명도... 알고리즘이라 하더라도 알기 쉬운 변수명을 짓는 연습도 필요할 것 같다.

그래도 한번에 풀어서 기분은 좋다 ~