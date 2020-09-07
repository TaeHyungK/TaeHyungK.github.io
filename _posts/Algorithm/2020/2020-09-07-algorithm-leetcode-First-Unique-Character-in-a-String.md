---
layout: post
title:  "[알고리즘] leetcode > First Unique Character in a String 풀이"
categories: [Algorithm, leetcode]
tags: [leetcode, easy]

---

[문제 출처](https://leetcode.com/explore/interview/card/top-interview-questions-easy/127/strings/881/) | [소스코드](https://github.com/TaeHyungK/algorithm/blob/master/src/leetcode/FirstUniqueCharacterInAString.java)

주어진 String 에서 유일한 character 위치 찾기 

### 첫번째 풀이

#### 소스 코드
```java
public class FirstUniqueCharacterInAString {
    public static void main(String[] args) {
        String s = "leetcode";
//        String s = "loveleetcode";
//        String s = "";
//        String s = "aadadaad";
        System.out.println("result: " + firstUniqChar(s));
    }
    
    public static int firstUniqChar(String s) {
        if (s != null && s.isEmpty()) return -1;
        String[] splitS = s.split("");
        if (splitS.length == 1) return 0;

        int result = -1;
        Set<String> data = new LinkedHashSet<>();
        Set<String> already = new HashSet<>();
        for (String str : splitS) {
            if (data.contains(str)) {
                data.remove(str);
            } else {
                if (!already.contains(str)) data.add(str); // 기존에 있던 애라면 단일 스트링으로 판단하지 않도록
                already.add(str);
            }
        }

        String target = null;
        for (String str : data) {
            target = str;
            break;
        }

        for (int i = 0; i < splitS.length; i++) {
            if (splitS[i].equals(target)) {
                result = i;
                break;
            }
        }
        return result;
    }
}
```

#### 풀이

자료구조 `Set` 을 이용해 중복되지 않는 데이터를 저장한다.

단, 유일한 character 들 중 first를 반환해야 하므로 `LinkedHashSet` 을 사용해 순서를 유지시킨다.

Set 변수 `already` 를 두어 기존에 있었던 값인지를 체크하여 유니크한 캐릭터만을 저장할 수 있도록 유도한다.


### 최종 풀이

#### 소스 코드

```java
public class FirstUniqueCharacterInAString {
    public static void main(String[] args) {
        String s = "leetcode";
//        String s = "loveleetcode";
//        String s = "";
//        String s = "aadadaad";
        System.out.println("result: " + firstUniqChar(s));
    }
    
    public static int firstUniqChar(String s) {
        int result = -1;
        Map<Character, Integer> data = new HashMap<>();
        for (char key : s.toCharArray()) {
            int value = data.getOrDefault(key, 0); // key에 밸류가 없을 때 Exception 방어
            data.put(key, value + 1);
        }

        for (int i = 0; i < s.length(); i++) {
            char key = s.charAt(i);

            if (data.get(key) == 1) {
                result = i;
                break;
            }
        }

        return result;
    }
}
```

#### 풀이

자료구조 `Map` 을 이용해 각 글자의 갯수를 파악한다.

데이터가 완성된 이후에는 `key(character)` 를 이용해 1개인 key 중 첫번째를 구한다.

#### 결과

![스크린샷 2020-09-08 오전 1.17.23](/img/스크린샷 2020-09-08 오전 1.17.23.png)

### 회고

대부분의 문제들이 다양한 자료구조를 이용해 풀 수 있지만 어떤 것을 쓰느냐에 따라 효율성이 판가름 나게 되고 로직도 더 쉽게 구현할 수 있는 것 같다.

어떤 자료구조를 사용할 지에 대해 고민해보는 좋은 문제였던 것 같다. 👍 