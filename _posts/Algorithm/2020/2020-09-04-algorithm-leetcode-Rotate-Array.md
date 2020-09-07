---
layout: post
title:  "[알고리즘] leetcode > Rotate Array 풀이"
categories: [Algorithm, leetcode]
tags: [leetcode, easy]

---

[문제 출처](https://leetcode.com/explore/interview/card/top-interview-questions-easy/92/array/646/) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/leetcode/RotateArray.java)

int 타입의 배열과 int 타입의 정수가 주어질 때 정수만큼 배열을 밀기

### 첫번째 풀이

#### 소스 코드
```java
public class RotateArray {
    public static void main(String[] args) {
//        int[] nums = {1,2,3,4,5,6,7};
//        int[] nums = {-1};
        int[] nums = {1,2};
        int k = 3;

        rotate(nums, k);
    }
    public static void rotate(int[] nums, int k) {
        for (int i = 0; i < k; i++) {
            int lastValue = nums[nums.length - 1];
            for (int j = nums.length - 1; j > 0; j--) {
                nums[j] = nums[j - 1];
            }
            nums[0] = lastValue;
        }
    }
}
```

#### 풀이

정말 문제에 충실하게 푸는 방법이다.

이중 포문을 이용해서 맨 뒤에 값은 앞으로 빼고 나머지 값들은 한칸 씩 직접 미는 형태로 구현하였다.


### 최종 풀이

#### 소스 코드

```java
public class RotateArray {
    public static void main(String[] args) {
//        int[] nums = {1,2,3,4,5,6,7};
//        int[] nums = {-1};
        int[] nums = {1,2};
        int k = 3;

        rotate(nums, k);
    }
    public static void rotate(int[] nums, int k) {
        int length = k % nums.length;
        reverse(nums, 0, nums.length - 1);
        reverse(nums, 0, length - 1);
        reverse(nums, length, nums.length - 1);
    }
    
    private static void reverse(int[] nums, int left, int right) {
        while (left < right) {
            int temp = nums[left];
            nums[left] = nums[right];
            nums[right] = temp;

            left++;
            right--;
        }
    }

}
```



#### 풀이

효율성을 높이고 싶기도 하고 다른 사람들은 어떻게 풀었는지 확인 후 감탄한 풀이법이다.

1. 전체 배열을 한번 다 뒤집는다.
    ```java
    reverse(nums, 0, nums.length - 1);
    ```
2. 0 ~ length - 1 까지의 값을 다시 뒤집는다.
    ```java
    reverse(nums, 0, length - 1);
    ```
3. 남은 length ~ 마지막 까지의 값을 다시 뒤집는다.
    ```java
    reverse(nums, length, nums.length - 1);
    ```

여기서 `length` 라는 변수가 존재하는데 이 변수는 배열의 크기가 k 보다 작을 경우 예외 처리를 위해서 계산해준 것이다.

**첫번째 풀이**로 풀 경우 이중 for문으로 O(N^2)의 효율성이겠지만 **최종 풀이**로 풀 경우 반복문이 3개로 O(3N) 으로 풀 수 있다.
 

#### 결과

![스크린샷 2020-09-08 오전 12.30.52](/img/스크린샷 2020-09-08 오전 12.30.52.png)

### 회고

첫번째 풀이로 풀고나서 꽤나 잘풀어냈다고 생각했는데 다른 사람의 풀이를 보고나서 경악했다. 도대체 어떻게 저런 생각을 한번에 할 수 있는 것인가...

나에게도 한문제, 한문제 조금씩 풀어가다보면 그런 날이 올 거라고 믿고 풀어본다..!