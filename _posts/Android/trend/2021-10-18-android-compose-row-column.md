---
layout: post
title:  "[Compose] Row와 Column"
categories: [Android, Compose]
tags: [Jetpack]
---

글을 시작하기에 앞서 무료로 볼 수 있는 좋은 유튜브 강의를 하나 소개하고자 한다.

Compose 의 각 요소들에 대해 공부를 좀 하려고 검색하던 중 우연히 보게 된 강의인데 쉽게 잘 설명해주셔서 튜토리얼로 좋은 것 같다.
이 글도 해당 유튜브 강의를 따라하면서 배운 것들을 정리해보고자 하는 취지로 작성하는 것이니 이 글을 보는 독자분들도 해당 유튜브를 보는 것이 큰 도움이 될 것 같다.

[개발하는 정대리: 안드로이드 Compose UI 뽀개기](https://www.youtube.com/watch?v=1apENzDbtCQ&list=PLgOlaPUIbynpFHXeEORmvIOoiNVgSsWeq)

### Row

Row는 각 요소들을 **가로**로 나열하고 싶을 때에 사용하는 Composable 이다.

```kotlin
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.size
import androidx.compose.material.MaterialTheme
import androidx.compose.material.Surface
import androidx.compose.runtime.Composable
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
                    Container()
                }
            }
        }
    }
}

@Composable
fun Container() {
    Row(
        modifier = Modifier
            .background(Color.White)
            .fillMaxSize()
    ) {
        DummyBox()
        DummyBox()
        DummyBox()
    }
}

@Composable
fun DummyBox(modifier: Modifier = Modifier) {
    val red = Random.nextInt(256)
    val green = Random.nextInt(256)
    val blue = Random.nextInt(256)

    Box(
        modifier = modifier
            .size(100.dp)
            .background(Color(red, green, blue))
    )
}

@Preview
@Composable
fun PreviewContainer() {
    Container()
}
```

이 `Column`과 `Row`에서 핵심적으로 사용되는것이 2가지 있는데 이는 아래와 같다.
* Arrangement
* Alignment

(참고로 `Column`은 아래에 나오니 조금만 기다려보자.)

#### Arrangement

컨테이너 성격을 가진 `Column`, `Row` 와 같은 컴포저블에서 요소들을 정렬할 때 사용되는 설정이다.

Row는 가로로 나열하는 컴포저블이므로 `horizontalArrangement` 가 있고 사용은 아래처럼 사용하면 된다.

```kotlin
// 이전 코드에서 Container 부분만 아래와 같이 바꿔보자.
@Composable
fun Container() {
    Row(
        modifier = Modifier
            .background(Color.White)
            .fillMaxSize(),
        horizontalArrangement = Arrangement.Center
    ) {
        DummyBox()
        DummyBox()
        DummyBox()
    }
}
```

![스크린샷 2021-10-19 오전 12.32.54.png](/img/스크린샷 2021-10-19 오전 12.32.54.png)

왼쪽에 바싹! 붙어있던 `DummyBox` 들이 가운데로 이동한 것을 볼 수 있다.

Center 뿐만 아니라 여러가지를 사용할 수 있다. 이런 옵션(?)들은 미리 알고 있으면 나중에 UI를 짤 때 굉장히 편하게 만들어주니 
각 옵션들마다 어떻게 적용되는지 `DummyBox`의 갯수를 늘리고 줄여보면서 테스트 해보길 바란다..!

* Arrangement.Start: 왼쪽으로 붙임.
* Arrangement.End: 오른쪽으로 붙임.
* Arrangement.Center: 가운데로 붙임.
* Arrangement.SpaceBetween: 사이에 공간을 밀어넣음.
* Arrangement.SpaceAround: 사이사이에 빈 공간 남겨둠.
* Arrangement.SpaceEvenly: 요소 사이에 공간 모두 동일하게 간격을 줌.

#### Alignment

말 그대로 해당 컨테이너 안에 들어있는 요소들의 위치를 어떠한 위치에 둘 것인지를 결정하는 설정이다.

친숙한 `LinearLayout` 에서 `gravity` 와 동일하다고 생각하면 된다.

`Row` 에서는 `verticalAlignment` 가 있고 사용은 아래와 같이 하면 된다.

```kotlin
// 이전 코드에서 Container 부분만 아래와 같이 바꿔보자.
@Composable
fun Container() {
    Row(
        modifier = Modifier
            .background(Color.White)
            .fillMaxSize(),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.Bottom
    ) {
        DummyBox()
        DummyBox()
        DummyBox()
    }
}
```

![스크린샷 2021-10-19 오전 12.31.45.png](/img/스크린샷 2021-10-19 오전 12.31.45.png)

* Alignment.Bottom: 컨테이너의 하단에 두기
* Alignment.Top: 컨테이너의 상단에 두기
* Alignment.CenterVertically: 컨테이너의 수직방향으로 가운데에 두기

<br><br><br><br>

### Column

Column은 각 요소들을 **세로**로 나열하고 싶을 때에 사용하는 Composable 이다.

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
                    VerticalContainer()
                }
            }
        }
    }
}

@Composable
fun VerticalContainer() {
    Column(
        modifier = Modifier
            .background(Color.White)
            .fillMaxSize(),
    ) {
        DummyBox()
        DummyBox()
        DummyBox()
    }
}

@Composable
fun DummyBox(modifier: Modifier = Modifier) {
    val red = Random.nextInt(256)
    val green = Random.nextInt(256)
    val blue = Random.nextInt(256)

    Box(
        modifier = modifier
            .size(100.dp)
            .background(Color(red, green, blue))
    )
}

@Preview
@Composable
fun PreviewContainer() {
    VerticalContainer()
}
```

앞서서 `Row` 를 만들었던 것 처럼 `Column` 도 동일하게 만들 수 있다.

#### Arrangement 와 Alignment

`Column`도 `Row`와 동일하게 Arrangement 와 Alignment 를 가지고 각 요소들의 정렬을 어떻게 할 것인지, 어디에 위치시킬 것인지를 정할 수 있다.

```kotlin
// 이전 코드에서 VerticalContainer 부분만 아래와 같이 바꿔보자.
@Composable
fun VerticalContainer() {
    Column(
        modifier = Modifier
            .background(Color.White)
            .fillMaxSize(),
        verticalArrangement = Arrangement.SpaceEvenly,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        DummyBox()
        DummyBox()
        DummyBox()
    }
}
```

![스크린샷 2021-10-19 오전 12.43.17.png](/img/스크린샷 2021-10-19 오전 12.43.17.png)

**Arrangement**에 `SpaceEvenly`를 주어 각 요소들이 일정한 간격을 가지도록 하였고,<br>
**Alignment**에 `CenterHorizontally`를 주어 컨테이너의 수평방향으로 가운데에 가도록 하였다.

`Row` 와 다른 점은 `Column` 에서는 `Arrangement` 는 **verticalArrangment**, `Alignment`는 **horizontalAlignment** 이라는 점이고,<br>
이외에는 `Row` 와 모두 동일하다.

`Column` 도 이리저리 옵션을 바꿔가면서 UI가 어떻게 변화되는지, 요소들이 어떻게 위치되는지를 꼭 확인하고 이해하도록 하자.

<br><br>

##### 글을 마치며

Row, Column을 시작으로 유튜브 강의를 통해 컴포즈에서 많이 사용되는 Composable들을 다루어볼 생각이다.

누군가는 '완전 기초인데 이걸 하나하나 뭐하러 하지..?' 라고 생각할 수도 있지만 모든 지식에는 기초가 근간이 된다고 생각하기 때문에 하나하나 차근차근 학습해보고자 한다..! 💪

시간이 지나 맥부이 느려진 것인지.. Android Studio에서 컴포즈 Preview가 아직 최적화가 덜 된 것인지 이 글을 작성하는데 안드로이드 스튜디오가 느려서 
굉장히 답답함을 느끼면서 작성하였다.<br>
만약 스튜디오가 느린 이유가 후자라면 부디 빠르게.. 업데이트가 되길 바래본다..! 🙏

###### 참고 사이트
* [Jetpack Compose 가이드](https://developer.android.com/jetpack/compose/documentation)
* [취준생을 위한 안드로이드 앱만들기 콤포즈UI Row - Android Kotlin jetpack Tutorial (2021) - JetPack Compose](https://www.youtube.com/watch?v=kkT1Sos2p2k&list=PLgOlaPUIbynpFHXeEORmvIOoiNVgSsWeq&index=3)
* [취준생을 위한 안드로이드 앱만들기 콤포즈UI Column - Android Kotlin jetpack Tutorial (2021) - JetPack Compose](https://www.youtube.com/watch?v=j47ToujKtvQ&list=PLgOlaPUIbynpFHXeEORmvIOoiNVgSsWeq&index=4)