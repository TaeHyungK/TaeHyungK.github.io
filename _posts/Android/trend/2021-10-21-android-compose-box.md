---
layout: post
title:  "[Compose] Box"
categories: [Android, Compose]
tags: [Jetpack]
---

글을 시작하기에 앞서 무료로 볼 수 있는 좋은 유튜브 강의를 하나 소개하고자 한다.

Compose 의 각 요소들에 대해 공부를 좀 하려고 검색하던 중 우연히 보게 된 강의인데 쉽게 잘 설명해주셔서 튜토리얼로 좋은 것 같다.
이 글도 해당 유튜브 강의를 따라하면서 배운 것들을 정리해보고자 하는 취지로 작성하는 것이니 이 글을 보는 독자분들도 해당 유튜브를 보는 것이 큰 도움이 될 것 같다.

[개발하는 정대리: 안드로이드 Compose UI 뽀개기](https://www.youtube.com/watch?v=1apENzDbtCQ&list=PLgOlaPUIbynpFHXeEORmvIOoiNVgSsWeq)

### Box

앞서 학습하기로 Row는 X축, Column은 Y축으로 배치하는 것이라 배웠다.

그렇다면 Box는 뭘까?

남은 한 방향인 Z축으로 배치하는 것을 의미한다. Z축..? 말이 어렵다.<br>
쉽게 생각하면 위에 겹치는 것을 생각하면 된다.

기존 xml에서 `RelativeLayout`, `ConstraintLayout`, `FrameLayout` 처럼 겹칠 수 있는 것과 같은 것이라고 생각하면 된다.

<br><br>

#### Alignment

`Column`과 `Row`와는 다르게 `Box`는 Arrangement가 없다. 그래서 바로 Alignment를 봐보도록 하자.

```kotlin
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.material.MaterialTheme
import androidx.compose.material.Surface
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.study.compose.ui.theme.ComposeTheme
import kotlin.random.Random

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            ComposeTheme {
                // A surface container using the 'background' color from the theme
                Surface(color = MaterialTheme.colors.background) {
                    BoxContainer()
                }
            }
        }
    }
}

@Composable
fun BoxContainer() {
    Box(
        modifier = Modifier
            .background(Color.White)
            .fillMaxSize(),
        contentAlignment = Alignment.CenterEnd
    ) {
        DummyBox(
            modifier = Modifier.size(200.dp),
            color = Color.Green
        )
        DummyBox(
            modifier = Modifier.size(150.dp),
            color = Color.Yellow
        )
        DummyBox(
            color = Color.Blue
        )
    }
}

@Composable
fun DummyBox(modifier: Modifier = Modifier, color: Color? = null) {
    val red = Random.nextInt(256)
    val green = Random.nextInt(256)
    val blue = Random.nextInt(256)
    // color 값이 있으면 해당 값을 사용, 없으면 랜덤 값을 사용
    val randomColor = color?.let { it } ?: Color(red, green, blue)
    Box(
        modifier = modifier
            .size(100.dp)
            .background(randomColor)
    )
}

@Preview
@Composable
fun PreviewContainer() {
    BoxContainer()
}
```

위의 코드를 실행해보면 아래와 같은 결과물을 얻을 수 있다.

![스크린샷 2021-10-21 오후 11.29.10.png](/img/스크린샷 2021-10-21 오후 11.29.10.png)

코드를 하나하나 뜯어보자.

```kotlin
Box(
        modifier = Modifier
            .background(Color.White)
            .fillMaxSize(),
        contentAlignment = Alignment.CenterEnd
    ) { ... }
        
```

먼저 Box 선언부다.

`Modifier.fillMaxSize()` 에 의해 해당 Box는 가득찬 영역을 차지하겠다고 정의되어있다.<br>
그리고, `contentAlignment` 파라미터에 `Alignment.CenterEnd` 를 사용함으로써 가운데 우측으로 붙이도록 한 것을 볼 수 있다.

Alignment.CenterEnd 외에도 Center, TopCenter, Start 등.. 매우 다양한 Alignment가 있으니 꼭 테스트를 해보면서 이것저것 해보길 바란다.

<br><br>

다음으로 Box의 내부이다.

```kotlin
Box(
    ...
) {
    // DummyFirst
    DummyBox(
        modifier = Modifier.size(200.dp),
            color = Color.Green
    )
    // DummySecond
    DummyBox(
        modifier = Modifier.size(150.dp),
        color = Color.Yellow
    )
    // DummyThird
    DummyBox(
            color = Color.Blue
    )
}
```

이미 이미지 처럼 DummyBox들이 겹쳐져서 노출되는 것을 보았다.<br>
그렇다면 DummyBoxe들의 순서를 DummyThird -> DummySecond -> DummyFirst 순서로 바꿔서 해보면 어떻게 될까?

정답은 size가 가장 큰 DummyFirst에 가려져서 DummySecond, DummyThird가 보이지 않게 된다.

![스크린샷 2021-10-21 오후 11.35.06.png](/img/스크린샷 2021-10-21 오후 11.35.06.png)

이로써 알 수 있는 것은 `Box` 를 통해서 안에 어떠한 UI를 두었을 때 위에서 아래 순서로 겹쳐져서 draw 된다는 것이다.

#### propagateMinConstraints

`propagateMinConsraints` 옵션을 true로 주게되면, 박스 안에 있는 제일 작은 크기의 뷰를 컨테이너 박스의 크기만큼 컨스트레인트를 걸게 됩니다.

```kotlin
@Composable
fun BoxContainer() {
    Box(
        modifier = Modifier
            .background(Color.White)
            .fillMaxSize(),
        contentAlignment = Alignment.CenterEnd,
        propagateMinConstraints = true
    ) {
        DummyBox(
            modifier = Modifier.size(200.dp),
            color = Color.Green
        )
        DummyBox(
            modifier = Modifier.size(150.dp),
            color = Color.Yellow
        )
        DummyBox(
            color = Color.Blue
        )
    }
}
```

아래 캡쳐한 사진과 같이 최소 사이즈인 Blue DummyBox가 가득 찬 것을 볼 수 있다.

![스크린샷 2021-10-21 오후 11.40.37.png](/img/스크린샷 2021-10-21 오후 11.40.37.png)

> 추가로, `propagateMinConstraints` 가 이해가 잘 가지 않아서 jetpack 라이브러리 문서를 보았는데 최소 제약 조건..? 이라는 것 같다.<br>
> 그래도 해당 옵션이 잘 이해가 가지 않아 어떤 식으로 활용이 될지 잘 감이 안온다..🤔
> 
> 참고로 해당 옵션은 `false` 가 default 이다.  


##### 글을 마치며

Row, Column, Box 를 사용하면 웬만한 UI들은 작성할 수 있을 것으로 보인다..!

핵심이 되는 3가지를 여태까지 배워봤는데 컴포즈 꽤나 느낌이 좋다..! 꽤나 흥미롭고 재밌는 것 같다 🤭

###### 참고 사이트
* [Jetpack Compose 가이드](https://developer.android.com/jetpack/compose/documentation)
* [Jetpack Compose Foundation 라이브러리](https://developer.android.com/jetpack/androidx/releases/compose-foundation?hl=ko)
* [취준생을 위한 안드로이드 앱만들기 콤포즈UI Box - Android Kotlin jetpack Tutorial (2021) - JetPack Compose](https://www.youtube.com/watch?v=4W9rDwaiEPQ&list=PLgOlaPUIbynpFHXeEORmvIOoiNVgSsWeq&index=5)