---
layout: post
title:  "[기타] jekyll Liquid error (line 6): comparison of Array with Array failed in ~"
categories: [Others]
tags: [tips]

---

오늘 겪은 jekyll build failed 에 대해 간략하게 해결 방법을 적고자 한다.

먼저 내가 마주한 build failed 의 원인은

### 에러 문구

![스크린샷 2020-09-04 오전 1.45.31](/img/스크린샷 2020-09-04 오전 1.45.31.png)

```command
Liquid Exception: Liquid error (line 6): comparison of Array with Array failed in tabs/categories.md
...
/{사용자 경로}/.rbenv/versions/2.7.1/lib/ruby/gems/2.7.0/gems/jekyll-4.1.1/lib/jekyll/filters.rb:308:in `sort': Liquid error (line 6): comparison of Array with Array failed (Liquid::ArgumentError)
```

명시 되어 있는 `tabs/cateogires.md` 파일을 가봐도 6번째 열에는 주석처리가 된 텍스트밖에 없었다.



이 이슈 때문에 몇 시간을 분노하며 구글링을 하고.. 결국 못찾아서 커밋을 하나하나 reset 해가며 무엇이 잘못되었는지 찾았는지 모른다.

(~~지금도 생각하면 화난다..🤬~~)

커밋 하나하나 reset 하며 확인해본 결과 해결 방법은 매우매우 간단했다. (~~애초에 내가 잘못했다.~~)

### 해결 방법

수정한 md 파일의 `categories`에는 **숫자만 존재할 수 없다.** 아마 `tags` 영역도 똑같을 것 같다.

#### 문제 발생 md 파일

```markdown
---
layout: post
title:  "[회고] 2019년을 돌아보며"
categories: [회고, 2019]
tags: [2019 회고]
---
```

위에 보이는 것 처럼 `categories: [회고, 2019]` 의 **2019**와 같이 숫자만 존재할 수 없다..!


#### 해결 후 md 파일

```markdown
---
layout: post
title:  "[회고] 2019년을 돌아보며"
categories: [회고, Me]
tags: [2019 회고]
---
```


아무리 구글링을 해도 나와 같은 문제를 해결한 사람은 없었다. 아마 다들 jekyll 테마의 설명서(?) 를 잘 읽어보고 포스팅을 하시거나 파워 블로거 분들이라 이러한 문제를 애초에 마주하지 않는 것 같다.

혹시나 나와 같은 초보 블로거가 이러한 문제를 발견한다면 이 글을 보고 해결하길 바라는 마음에 정리하였다.

초보 블로거 화이팅..!