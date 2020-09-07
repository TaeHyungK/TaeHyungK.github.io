---
layout: post
title:  "[알고리즘] leetcode > Single Number 풀이"
categories: [Algorithm, leetcode]
tags: [leetcode, easy]

---

[문제 출처](https://leetcode.com/explore/interview/card/top-interview-questions-easy/92/array/549/) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/leetcode/SingleNumber.java)

중복되지 않는 값을 찾아 반환하기

### 첫번째 풀이

#### 소스 코드
```java
public class SingleNumber {
    public static void main(String[] args) {
        int[] nums = {4,1,2,1,2};
//        int[] nums = {1,3,1,-1,3};
        System.out.println("result: " + singleNumber(nums));
    }
    
    public static int singleNumber(int[] nums) {
        if (nums.length == 1) return nums[0];

        Integer result = null; // 초기값을 -1로 할 경우 result 가 실제로 -1일 때에 대한 구분을 할 수 없어 객체화 함.
        Arrays.sort(nums);

        int findingCount = 0;
        int comparingItem = nums[0];
        for (int i = 1; i < nums.length - 1; i++) {
            if (i == 1 && comparingItem != nums[i]) {
                // 처음 비교하자마자 다른 값인 경우
                result = comparingItem;
                break;
            }

            if (comparingItem != nums[i]) {
                findingCount++;

                if (findingCount >= 2) {
                    result = comparingItem;
                    break;
                }
            } else {
                // 같을 경우 0으로 초기화
                findingCount = 0;
            }
            comparingItem = nums[i];
        }

        // 마지막 값이 혼자인 경우
        result = result == null ? nums[nums.length - 1] : result;
        return result;
    }
}
```

#### 풀이

`nums` 배열을 정렬한 후 for 문을 돌며 비교할 값(`comparingItem`)과 현재 값(`nums[i]`) 이 다른지 체크하여 다를 경우 `findingCount` 값을 증가시킨다.

반복하여 위 처럼 비교하여 `findingCount`가 2가 된 경우 앞과 뒤 모두 다른 값이라 판단하고 `compairingItem` 이 Single 인 값이 된다.


### 최종 풀이

#### 소스 코드

```java
public class SingleNumber {
    public static void main(String[] args) {
        int[] nums = {4,1,2,1,2};
//        int[] nums = {1,3,1,-1,3};
        System.out.println("result: " + singleNumber(nums));
    }
    
    public static int singleNumber(int[] nums) {
        if (nums.length == 1) return nums[0];
        Set<Integer> data = new HashSet<>();
        for (int num : nums) {
            if (data.contains(num)) {
                data.remove(num);
            } else {
                data.add(num);
            }
        }

        int result = -1;
        for (int num : data) {
            result = num;
        }
        return result;
    }
}
```

#### 풀이

자료구조 `Set` 을 이용해 같은 값이 있는 경우 `remove()`, 없는 경우 `add()` 한다.

`data` Set을 완성한 후에는 forEach(iterator를 위함) 를 이용해 Single 인 값을 받아온다. 

#### 결과

![스크린샷 2020-09-08 오전 1.09.27](/img/스크린샷 2020-09-08 오전 1.09.27.png)

### 회고

효율성이 좋은 알고리즘을 풀기 위한 공부를 하고 있지만.. 자꾸 일차원적인 생각으로 푸는 것 같다. 좀 더 생각하면서 풀어 보자 ! 😢