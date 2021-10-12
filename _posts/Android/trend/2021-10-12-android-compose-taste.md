---
layout: post
title:  "[Compose] Jetpack Compose 맛보기"
categories: [Android, Compose]
tags: [Jetpack]
---

### Jetpack Compose

안드로이드 개발자라면 눈에 불을 켜고 유심히 보고있는 Jetpack.. 그 중에서도 요즘 굉장히 핫한(?) `Compose` 에 대해서 알아보고자 한다.

`Compose` 는 Native UI를 코드레벨로 구현할 수 있는 최신 도구 키트이다.

이 방식은 기존 xml을 사용하던 안드로이드 개발자들에게는 생소한 방식 (나한테서만 그럴지도.. 😅)이다.

기존의 방식도 불편함이 딱히 없는데.. 굳이 왜 배우고 왜 써야 하는가?

<br><br><br><br>

#### Compose의 이점

안드로이드에서 Compose를 채택하는 이유는 크게 4가지가 있다.

* Less code : 코드 감소
  * 코드를 적게 작성하여 버그 발생 가능성이 줄어들고 코드 유지 관리할 코드 또한 줄어들게 된다.
  * 코드를 더이상 Kotlin, xml로 나누지 않고 Kotlin으로만 작성하므로 코드 추적도 용이하다. (이점은 사실 크게 공감이 가진 않는다. 지금도 딱히 어렵진 않으니.. 🤪)
* Intuitive : 직관적
  * Compose는 선언적 API를 사용하므로 UI만 잘 작성해두면 Compose가 대부분의 것들을 알아서 처리해준다.
  * API는 직관적이므로 찾아서 사용하기가 쉽다. 
* Accelerate Development : 빠른 개발 과정
  * Compose는 기존의 모든 코드와 호환된다. (기존 앱들이 Compose로 빠르게 넘어갈 수 있는 강력한 이유 중 하나가 아닐까 싶다.)
  * 즉, Compose에서 Views를, Views에서 Compose 코드를 호출할 수 있다.
  * 실시간 미리보기와 같은 기능을 통해 빌드 시간을 절약해서 빠르게 확인할 수 있다. 
* Powerful : 강력한 성능
  * Android 플랫폼 API에 직접 액세스가 가능하며 머테리얼 디자인, 다크테마, 애니메이션 등을 기본적으로 지원한다.

4가지 이점 중에 나는 가장 이상적이라고 생각했던 부분은 `Compose -> Views, Views -> Compose` 가 모두 호환된다는 점이다.<br>
신기술이 나오더라도 기존 작성된 코드에 호환이 되지 않으면 다음 step으로 넘어가기가 쉽지 않은 것이 사실인데 `Compose` 는 이를 완전히 깨부쉈다고 생각한다.<br>
정말 대단하다..! 👊

<br><br><br><br>

#### Compose 사용하기

##### Pre-condition

Compose를 사용하기에 앞서 Compose를 사용하기 위한 제약 조건이 존재한다.

1. [Android Studio Canary](https://developer.android.com/studio/preview) 를 사용해야 한다.
2. 최신 버전의 코틀린 plugin 을 사용해야 한다.

##### 프로젝트 생성

Compose를 처음 써보니 어떻게 해야하는지 막막하다.<br>
라고 생각하는 사람들을 위해 Android Studio에 배려를 심어놓았다. 그 배려는 아래 이미지와 같다.

![스크린샷 2021-10-12 오후 11.42.49.png](/img/스크린샷 2021-10-12 오후 11.42.49.png)

프로젝트 생성 시에 `Empty Compose Activity` 라는 항목이 있다. 이 항목을 선택하면 Compose 샘플을 볼 수 있다.

프로젝트가 이미 생성되어 있는 사람들을 위해 gradle 설정 중 중요한 것들을 몇가지 추려왔다.<br>
기존 프로젝트에 적용하기 위해선 아래 gradle 을 참고하도록 하자.

```gradle
// build.gradle(Project) 에 compose_version='1.0.1' 가 선언되어 있다는 가정

android {
    ...
    buildFeatures {
        compose true
    }
    composeOptions {
            kotlinCompilerExtensionVersion compose_version
            kotlinCompilerVersion '1.5.21'
        }
    ...
}

dependencies {
    ...
    implementation "androidx.compose.ui:ui:$compose_version"
    implementation "androidx.compose.material:material:$compose_version"
    implementation "androidx.compose.ui:ui-tooling-preview:$compose_version"
    implementation 'androidx.activity:activity-compose:1.3.0-alpha06'
    ...
}
```

> 한가지 주의할 점은 아직 Compose는 완성본이 아니라는 점이다.
> composeOptions 설정에서 kotlinCompilerVersion에 따라서 조금씩 달라질 수 있으니 이 점은 유의하도록 하자.

##### 샘플 코드

```kotlin
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.material.MaterialTheme
import androidx.compose.material.Surface
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.tooling.preview.Preview
import com.study.compose.ui.theme.ComposeTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            ComposeTheme {
                // A surface container using the 'background' color from the theme
                Surface(color = MaterialTheme.colors.background) {
                    Greeting("Android")
                }
            }
        }
    }
}

@Composable
fun Greeting(name: String) {
    Text(text = "Hello $name!")
}

@Preview(showBackground = true)
@Composable
fun DefaultPreview() {
    ComposeTheme {
        Greeting("Android")
    }
}
```

먼저, 기존에는 `onCreate()`에서 `setContentView(layoutId: Int)` 가 호출되었는데 여기서는 `setContent()` 함수로 변한 것을 볼 수 있다.

두번째로는 `@Composable` 어노테이션이 붙은 함수를 볼 수 있다.<br>
이 함수는 Composable 이라는 것을 알리는 어노테이션으로 안드로이드 대문자로 함수명을 시작하는 것을 권장하고 있다. 
아마 Composable 함수(?) 자체가 뷰를 의미하기 때문에 대문자로 시작하는 것으로 보인다.

세번째로는 `@Preview` 어노테이션이 붙은 함수를 볼 수 있다.<br>
xml에서는 우측에 preview를 통해 빌드를 하기 전 내가 구현하고 있는 UI를 볼 수 있었다. 이 기능을 할 수 있게 하는 것이 `@Preview` 어노테이션이다.

<br><br>

> 뜬금 없는 사용기 넋두리..
> 
> 내가 사용해본 바로는 Compose를 사용하다 보면 컴포즈에 `viewModel` 과 같은 여러 디펜던시가 필요할 수 밖에 없어지는데,
> 이 때 Composable 들을 세분화해서 쪼개놓지 않으면 Preview를 보는 것에 대해 엄청난 어려움이 생길 수 있다.<br>
> Composable을 정의할 때에 어떻게 쪼개서 구현할지를 잘 고민하면서 짜는 것이 개발 속도 향상에 큰 도움이 될 것 같다.<br>
> (잘 쪼개두면 나중에 재사용성은 덤..!? 🎁)
> 
> 아직 Experimental API가 많다는 점도 유의해야 한다.<br>
> 최근 Compose에 손이 익게 하고 공부할 겸 해서 회사 코드에서 간단한 UI를 Compose로 개선하는 작업을 틈틈히 하고 있는데,
> BottomSheetDialog를 개선하는 과정에서 `BottomSheetScaffold` 라는 컴포저블이 있다는 것을 알게 되어서 사용할까 싶었는데
> 얘는 아직 Experimental 이여서 일부러 이것을 피해서 구현했던 경험이 있다.<br>
> 아직 프로덕트에서 사용하기에는 Experimental API는 무서움이 있어서 알아서 잘 우회해야 한다. 

<br><br>

##### 글을 마치며

Compose는 기존에 사용하던 UI 구현 패러다임(?)을 확 뒤집어 놓은 것 같다.

아무래도 기존 xml 구현에 익숙했던 개발자일수록 더 어려울 수도 있을 것 같다는 생각이 든다. 
하지만 나는 아직 주니어 개발자니까.. 라는 위안을 하며 새로운 패러다임에 빠르게 발을 디뎌보도록 하자..! 🦶  

###### 참고 사이트
* [Jetpack Compose 가이드](https://developer.android.com/jetpack/compose/documentation)
* [Compose를 채택하는 이유](https://developer.android.com/jetpack/compose/why-adopt)
* [Jetpack Compose Part 1 — Compose 소개 및 코드랩 따라하기](https://medium.com/android-deep-dive-study/jetpack-compose-part-1-compose-%EC%86%8C%EA%B0%9C-%EB%B0%8F-%EC%BD%94%EB%93%9C%EB%9E%A9-%EB%94%B0%EB%9D%BC%ED%95%98%EA%B8%B0-35f7a0e6c581)