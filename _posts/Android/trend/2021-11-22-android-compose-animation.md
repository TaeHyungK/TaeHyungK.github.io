---
layout: post
title:  "[Compose] Animation 사용해보기"
categories: [Android, Compose]
tags: [Jetpack]
---

최근 GDG dev 2021을 통해서 Compose Codelab 공부를 하고 있는데, Animation을 사용하는 예제들을 따라해보고자 한다.

[Jetpack Compose Animation](https://www.youtube.com/watch?v=uJ7MhPhIHso)

[작성된 전체 코드](https://github.com/TaeHyungK/ComposeFest2021/blob/main/week%203-2-Jetpack%20Compose%20Animation/start/src/main/java/com/example/android/codelab/animation/ui/home/Home.kt)



## Animation

### Animating a simple value change

`animate*AsState` 라는 API를 이용하면 값이 변경될 때 값이 변경된 값을 애니메이션화 해주는 것이라고 생각하면 된다.

```kotlin
@Composable
fun Home() {
    ...
    val backgroundColor by animateColorAsState(targetValue = if (tabPage == TabPage.Home) Purple100 else Green300)
    ...
}
```

위와 같은 코드를 통해 targetValue가 변화하는 것에 따라 background의 color가 변화하는 것을 볼 수 있다.

<br><br><br>

### Animating visibility

boolean 값을 통해서 UI를 노출할지/말지에 대해 정하게 되고 이 상태가 변할 때 애니메이션을 보여주고자 할 때에 사용하게 된다.

또한, 머테리얼 디자인의 권장 사항에 맞게 상위 수준의 애니메이션의 api를 제공해주고 있다고 한다.

다만, 이 부분은 실험용 기능으로 사용되고 있는 api들이기 때문에 나중에 수정되거나 삭제될 수 있음을 유의하여야 한다.

코드랩에서는 Transition을 다루고 있는데 그 외에도 FadeIn, Sliding, Scaling 등의 애니메이션을 제공하고 있다고 한다.

```kotlin
@OptIn(ExperimentalAnimationApi::class)
@Composable
private fun HomeFloatingActionButton(
        ...
) {
  ...
  AnimatedVisibility(extended) {
    Text(
            text = stringResource(R.string.edit),
            modifier = Modifier
                    .padding(start = 8.dp, top = 3.dp)
    )
  }
}
```

`AnimatedVisibility` api를 통해서 extended라는 boolean 값의 변화에 따라서 해당 Text가 스르륵 등장했다가 스르륵 사라지는 애니메이션 효과가 생기는 것을 볼 수 있다.

```kotlin
@OptIn(ExperimentalAnimationApi::class)
@Composable
private fun EditMessage(shown: Boolean) {
    AnimatedVisibility(
        visible = shown,
        enter = slideInVertically(
            initialOffsetY = { fullHeight -> -fullHeight },
            animationSpec = tween(durationMillis = 150, easing = LinearOutSlowInEasing)
        ),
        exit = slideOutVertically(
            targetOffsetY = { fullHeight -> -fullHeight },
            animationSpec = tween(durationMillis = 250, easing = FastOutLinearInEasing)
        )
    ) {
        ...
    }
}
```

위와 같이 `AnimatedVisibility` api에 enter, exit 파라미터를 통해서 커스텀도 가능하다.

위 예시에서는 edit 버튼을 누르면 상단에서 스르륵 글자가 나왔다가 위로 스르륵 사라지는 애니메이션을 구현한 모습을 볼 수 있다.

<br><br><br>

### Animating content size Change

content의 사이즈가 변경될 때에 애니메이션 효과를 줄 수 있는 `animateContentSize` 함수에 대해 학습하고자 한다.

```kotlin
@OptIn(ExperimentalMaterialApi::class)
@Composable
private fun TopicRow(topic: String, expanded: Boolean, onClick: () -> Unit) {
  ...
  Surface(
          ...
  ) {
    Column(
            modifier = Modifier
                    .fillMaxWidth()
                    .padding(16.dp)
                    .animateContentSize()
    ) {
        ...
    }
  }
  ...
}
```

`animateContentSize` api는 위와 같이 Modifier에 체인을 통해 사용할 수 있다.

api 이름처럼 content의 size가 변화할 때에 애니메이션이 동작하며 위 예시처럼 파라미터로 아무것도 넘겨주지 않으면 기본적으로 spring 애니메이션을 보여주게 된다.

> 이렇게 플랫폼에서 제공해주는 api들을 사용할 때에는 내부 구현을 한번쯤은 보는 것을 추천한다.
> <br>
> 기본적으로 파라미터를 넘기지 않아도 spring 애니메이션이 나오는 것을 알 수 있는 이유도 내부 구현을 보았기 때문이다..!

<br><br><br>

### Multiple value animation

`updateTransition` 함수를 통해서 Transition 인스턴스를 만들고, 상태를 저장하고, 상태를 업데이트 하는 과정을 거치게 된다.

이 때, `transitionSpec` 이라는 매개변수를 통해서 애니메이션 동작을 커스텀할 수 있다.

```kotlin
@Composable
private fun HomeTabIndicator(
    tabPositions: List<TabPosition>,
    tabPage: TabPage
) {
  val transition = updateTransition(targetState = tabPage, label = "Tab indicator")
  val indicatorLeft by transition.animateDp(
          transitionSpec = {
            if (TabPage.Home isTransitioningTo TabPage.Work) {
              // Indicator moves to the right.
              // The left edge moves slower than the right edge.
              spring(stiffness = Spring.StiffnessVeryLow)
            } else {
              // Indicator moves to the left.
              // The left edge moves faster than the right edge.
              spring(stiffness = Spring.StiffnessMedium)
            }
          },
          label = "Indicator left"
  ) { page ->
    tabPositions[page.ordinal].left
  }
  val indicatorRight by transition.animateDp(
          transitionSpec = {
            if (TabPage.Home isTransitioningTo TabPage.Work) {
              // Indicator moves to the right
              // The right edge moves faster than the left edge.
              spring(stiffness = Spring.StiffnessMedium)
            } else {
              // Indicator moves to the left.
              // The right edge moves slower than the left edge.
              spring(stiffness = Spring.StiffnessVeryLow)
            }
          },
          label = "Indicator right"
  ) { page ->
    tabPositions[page.ordinal].right
  }
  val color by transition.animateColor(label = "Border color") { page ->
    if (page == TabPage.Home) Purple700 else Green800
  }
  ...
}
```

위와 같은 코드를 사용해서 빌드를 해보면 탭 영역의 네모 박스가 더 역동적(?)으로 움직이는 것을 볼수 있다.

> 아직 위의 코드를 완벽히 이해하지 못했다... 😰

<br><br><br>

### Repeated animation

InfiniteTransition 을 사용하면 **MultiPle value animation** 에서 학습한 것 처럼 여러 애니메이션을 묶어서 사용하는 것 처럼 비슷한 형태로 사용이 가능하다.

다만, 다른점은 애니메이션이 즉시 시작하며 삭제되지 않는한 중단되지 않는 애니메이션을 만들 수 있다.

```kotlin
@Composable
private fun LoadingRow() {
  val infiniteTransition = rememberInfiniteTransition()
  val alpha by infiniteTransition.animateFloat(
          initialValue = 0f,
          targetValue = 1f,
          animationSpec = infiniteRepeatable(
                  animation = keyframes {
                    durationMillis = 1000
                    0.7f at 500
                  },
                  repeatMode = RepeatMode.Reverse
          )
  )
  ...
}
```

<br><br><br>

### Gesture animation

가장 하위 수준인 Animatable 은 터치, 드래그 등의 이벤트가 발생했을 때 애니메이션을 스탑하거나 하는 등의 제어를 할 수 있게 한다.

해당 코드는 길고 주석이 중요해서 링크를 통해 보는 것이 편할 것 같아 따로 코드를 적지 않았다.<br>
[코드 링크](https://github.com/TaeHyungK/ComposeFest2021/blob/e8d5e2b7c97ce40093e38923a84ff4505b29fa4d/week%203-2-Jetpack%20Compose%20Animation/start/src/main/java/com/example/android/codelab/animation/ui/home/Home.kt#L671) 를 참고하길.. 🙏

<br><br><br>


##### 글을 마치며

지난번 커스텀 Layout에 이어서 이번엔 애니메이션이다. 
안드로이드 개발을 하면서 어렵다고 생각한 부분 중 하나가 애니메이션인데 너무 간단하게(?) 코드 랩이 종료되는 것 같아 아쉽다.<br>
그리고 실제 개발에서도 TouchEvent 같은 것들을 다룰 때에는 굉장히 조심스럽고 쉽지 않은데 코드랩에서는 생각보다 간단하게 넘어가서 이해도 쉽게 되지 않는다.<br>
이 부분은 직접 사용해보고 깨우치는 과정이 추가적으로 필요할 것 같다. 역시나 어렵다 애니메이션.. 😰


###### 참고 사이트
* [Jetpack Compose 가이드](https://developer.android.com/jetpack/compose/documentation)
* [Jetpack Compose Animation : Codelab](https://developer.android.com/codelabs/jetpack-compose-animation?authuser=4&continue=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fcompose%3Fhl%3Den%26authuser%3D4%23codelab-https%3A%2F%2Fdeveloper.android.com%2Fcodelabs%2Fjetpack-compose-animation&hl=en#8)
* [Jetpack Compose Animation : Youtube](https://www.youtube.com/watch?v=uJ7MhPhIHso)
* [작성된 코드](https://github.com/TaeHyungK/ComposeFest2021/blob/main/week%203-2-Jetpack%20Compose%20Animation/start/src/main/java/com/example/android/codelab/animation/ui/home/Home.kt)