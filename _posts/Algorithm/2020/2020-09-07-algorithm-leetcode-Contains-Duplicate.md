---
layout: post
title:  "[알고리즘] leetcode > Contains Duplicate 풀이"
categories: [Algorithm, leetcode]
tags: [leetcode, easy]

---

[문제 출처](https://leetcode.com/explore/interview/card/top-interview-questions-easy/92/array/578/) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/leetcode/ContainsDuplicate.java)

중복되는 값이 있는지 체크하기

### 첫번째 풀이

#### 소스 코드
```java
public class ContainsDuplicate {
     public static void main(String[] args) {
         int[] nums = {1,2,3,4};
         System.out.println("result: " + containsDuplicate(nums));
     }
     public static boolean containsDuplicate(int[] nums) {
         boolean result = false;
         for (int i = 0; i < nums.length; i++) {
             int base = nums[i];
             for (int j = 0; j < nums.length; j++) {
                 if (i == j) continue;
 
                 int num = nums[j];
                 if (base == num) {
                     result = true;
                     break;
                 }
             }
         }
         return result;
     }
 }
```

#### 풀이

O(N^2)인 이중 for 문을 통해 모든 배열을 탐색한다.

이처럼 최적화 없이 모든 배열을 돌릴 경우 tiemout 으로 통과하지 못한다.


### 두번째 풀이

#### 소스 코드

```java
public class ContainsDuplicate {
     public static void main(String[] args) {
         int[] nums = {1,2,3,4};
         System.out.println("result: " + containsDuplicate(nums));
     }
     public static boolean containsDuplicate(int[] nums) {
         boolean result = false;
         for (int i = 0; i < nums.length; i++) {
             int base = nums[i];
             for (int j = i + 1; j < nums.length; j++) {
                 int num = nums[j];
                 if (base == num) {
                     result = true;
                     break;
                 }
             }
         }
         return result;
     }
}
```

#### 풀이

첫번째 풀이와 비슷하지만 중첩된 내부 for 문에서 바깥의 for문이 돌고 있는 `i` 이후로만 돌도록 하였기 때문에

적당히(?) 최적화가 된 상태의 풀이법이다.

이렇게 풀면 `Accepted` 되기는 하지만 Runtime이 835ms 로 매우 긴 시간을 사용하게 된다.
 

### 최종 풀이

```java
public class ContainsDuplicate {
     public static void main(String[] args) {
         int[] nums = {1,2,3,4};
         System.out.println("result: " + containsDuplicate(nums));
     }
     public static boolean containsDuplicate(int[] nums) {
         Set<Integer> data = Arrays.stream(nums).boxed().collect(Collectors.toSet());
         return nums.length != data.size();
     }
}
```

#### 풀이 

중복을 허용하지 않는 자료구조 `Set` 을 이용한 풀이이다.

input인 `nums` 배열의 갯수와 중복이 제거된 상태의 `data` Set의 갯수가 같지 않을 경우 중복된 값이 있다고 판단한다. 

#### 결과

![스크린샷 2020-09-08 오전 12.44.20](/img/스크린샷 2020-09-08 오전 12.44.20.png)

### 회고

알고리즘 공부를 할 때에는 정답을 맞췄다는 것에 의미를 두기보다는 문제를 풀기 위해, 더 좋은 효율성을 가지는 방법을 찾기 위해 생각하는 힘을 기르는 것이라고 생각한다.

엄청 뛰어난 효율성인 코드는 아니더라도 꾸준히 생각하는 힘을 기르고 있는 것 같다 !  