---
layout: post
title:  "[DI] Hilt 로 Dagger 를 쉽게 쓰자 !"
categories: [Android, DI]
tags: [Kotlin]
---

### Hilt

Hilt는 Dagger 기반의 DI 라이브러리로 Annotation을 이용한 컴파일 타임 generated code로 의존성 주입을 구현하였습니다.

기존 Dagger는 오류를 컴파일 타임에 검증이 가능하고 퍼포먼스가 준수하다는 장점이 있지만 과도하게 많은 Annotation과 보일러 플레이트 코드 때문에 러닝커브가 높다는 단점이 있습니다.

Hilt는 기존 Dagger의 장점은 그대로 활용하면서 보일러 플레이트 코드를 줄이고 진입장벽을 낮추기 위해 만든 라이브러리 입니다.





<br><br>
##### 먼저 보면 좋은 글
* [[DI] 의존성 주입? Dagger?](https://taehyungk.github.io/posts/android-DI-Dagger/)

<br><br>


#### Hilt를 써보자

먼저, Hilt를 프로젝트에 적용하기 위해 plugin과 dependency를 설정해주어야 합니다.

```
// project 단의 build.gradle에 플러그인을 추가
buildscript {
    ...
    repositories { ... }
    dependencies {
        ...
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.28-alpha'
        ...
    }
}

// app 단의 build.gradle에 플러그인을 추가
apply plugin: 'dagger.hilt.android.plugin'

// app 단의 build.gradle에 dependency 추가
dependencies {
    ...
    implementation "com.google.dagger:hilt-android:2.28-alpha"
    kapt "com.google.dagger:hilt-android-compiler:2.28-alpha"

    //viewModel 관련된 확장 라이브러리
    implementation 'androidx.hilt:hilt-lifecycle-viewmodel:1.0.0-alpha02'
    kapt 'androidx.hilt:hilt-compiler:1.0.0-alpha02'

    // Fragment
    implementation "androidx.fragment:fragment-ktx:$fragment_version"
    ...
}
```

<br>

위와 같이 project, app 단의 build.gradle에 plugin과 dependency를 설정해주면 기본적인 셋업은 완료가 되었습니다.

다음으로 본격적으로 Hilt를 프로젝트에 활용해보도록 하겠습니다.

##### Hilt Application

`@HiltAndroidApp`은 컴파일 타임 시 표준 컴포넌트 빌딩에 필요한 클래스들을 초기화를 해줍니다. 따라서 Hilt를 사용하는 모든 앱은 `@HiltAndroidApp` 이 달린 Application 클래스를 반드시 포함해야 합니다.

```kotlin
// Application을 상속받는 클래스에서 사용한 예시
@HiltAndroidApp
class ApplicationController : Application() {
    companion object {
        lateinit var INSTANCE: ApplicationController
    }

    override fun onCreate() {
        super.onCreate()
        INSTANCE = this
    }
}
```

<br><br>

##### @AndroidEntryPoint

Application에게 멤버 주입이 가능하게 설정하고 나면, 다른 안드로이드 클래스들에서도 `@AndroidEntryPoint` 어노테이션을 사용하여 멤버 주입을 하는 것이 가능해집니다.

즉, 의존성을 받아보고자 하는 안드로이드 컴포넌트에 작성해주면 됩니다.

단, 현재 alpha 버전에서 `@AndroidEntryPoint`를 사용할 수 있는 타입은 다음 5개로 한정적으로 제공하고 있습니다.

* Activity
* Fragment
* View
* Service
* BroadcastReceiver

```kotlin
// Main 화면에서 사용한 예시
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    ...
    private lateinit var binding: ActivityMainBinding
    private var mCurType: Int = DataConst.BOARD
    // ApplicationComponent 또는 ActivityComponent로 부터 의존성이 주입됩니다.
    @Inject lateinit var defaultRepo: DefaultRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        // super.onCreate() 에서 의존성 주입이 발생하게 됩니다.
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this,
            R.layout.activity_main
        )
        binding.lifecycleOwner = this

        init()
    }
    ...
    private fun changeTab(type: Int) {
        ...
        when (type) {
            ...
            DataConst.LOGOUT -> {
                defaultRepo.setLoginData("")
            }
        }
        ...
    }
```

MainActivity에서 `DefaultRepository` 를 주입받아 사용하고 있습니다.<br>
(예시에서는 간단하게 사용하기 위해 ViewModel을 거치지 않고 View에서 바로 Repository를 사용하도록 하였습니다.)


> 참고로, 현재 Hilt 에서는 `ComponentActivity`를 상속한 Activity만 지원하고 있습니다.
> Fragment 같은 경우 androidx 라이브러리의 Fragment를 상속한 경우에만 지원합니다. (안드로이드 플랫폼에 있는 Fragment는 deprecated 되었으므로 더이상 지원하지 않습니다.)


<br>

##### @InstallIn

Hilt의 모듈은 표준 Dagger 모듈로 `@InstallIn`이라는 추가적인 어노테이션을 가지고 있습니다.

`@InstallIn`은 Hilt의 표준 컴포넌트들 중 어떤 컴포넌트에 모듈을 설치할지를 결정합니다.

Hilt 컴포넌트가 생성될 때 모듈들은 추가된 `@InstallIn`과 함께 알맞은 컴포넌트 또는 서브컴포넌트에 설치가 됩니다.


```kotlin
@Module
@InstallIn(ApplicationComponent::class)
abstract class RepositoryModule {

    // Binds 어노테이션을 통해 interface와 그 구현체를 묶어줌
    @Binds
    abstract fun provideDefaultRepository(defaultRepo: DefaultRepositoryImpl) : DefaultRepository
}
```

`RepositoryModule` 이라는 module을 `ApplicationComponent`에 install 하여 사용하였습니다.

<br>

만약 하나의 module을 여러개의 component에 install 하고 싶은 경우, 콤마(,) 를 통해 여러 component에 install이 가능합니다.

```kotlin
@InstallIn(ViewComponent::class, ViewWithFragmentComponent::class)
```

이처럼 여러 component에 하나의 module을 install 하는데에는 몇가지 규칙이 있습니다.

1. Provider는 다중 component가 모두 동일한 scope에 속해있을 경우에만 scope를 지정할 수 있다.

  > 위의 예시에서 `ViewComponent`와 `ViewWithFragmentComponent` 가 동일한 `ViewScoped`에 속하기 때문에 provider에게 동일한 `ViewScoped`를 지정할 수 있다.

2. Provider는 다중 component가 서로 간 요소에게 접근이 가능한 경우에만 주입이 가능합니다.

  > 위의 예시에서 동일한 `ViewScoped`에 속하기 때문에 서로 간의 요소에 접근이 가능해 View에게 주입이 가능하다. 하지만 한 모듈에서 `FragmentComponent`와 `ServiceComponent` 를 install 한 경우 `Fragment` 또는 `Service`에게 주입이 불가능하다.

3. 부모 component와 자식 component에 동시에 install 될 수 없으며 자식 component는 부모 component의 module에 대한 접근이 가능합니다.


`@InstallIn` 어노테이션에 대해 알아보면서 `Component의 상하 관계` 와 `Scope`에 따라 module에 접근이 가능/불가능 하다는 것을 알게 되었습니다.

또한 Hilt는 표준적으로 사용되는 Component들을 기본적으로 제공해주고 있습니다. 이 덕분에 기존 Dagger2에서 선언해주어야 하는 Component를 선언하지 않아도 사용할 수 있습니다.

기본적으로 제공해주는 표준 Component에 대해 알아보겠습니다.


<br><br>

##### 표준 Component 계층 구조

![hilt_component_hierarchy](/img/hilt_component_hierarchy.svg)


컴포넌트가 생성되고 종료될 때, 해당 스코프 어노테이션이 지정된 바인딩 또한 수명을 함께하게 되고 컴포넌트 수명은 주입된 값들이 사용될 수 있는 시기를 나타내기 때문에 컴포넌트의 수명을 알고 있는 것이 중요합니다.

> @Inject 필드를 사용할 때 null이면 안된다 !!

<br>
다음은, 각 컴포넌트에 따른 관련 스코프 어노테이션과 생성/소멸되는 시점을 나타내는 표입니다.


|Component|Scope|Created at|Destroyed at|
|----------|------------|----------|---------|
|ApplicationComponent|@Singleton|Application#onCreate()|Application#onDestroy()
|ActivityRetainedComponent|@ActivityRetainedScoped|Activity#onCreate()|Activity#onDestroy()|
|ActivityComponent|@ActivityScpoed|Activity#onCreate()|Activity#onDestroy()|
|FragmentComponent|@FragmentScoped|Fragment#onAttach()|Fragment#onDestroy()|
|ViewComponent|@ViewScoped|View#super()|view **destroyed**|
|viewWithFragmentComponent|@ViewScoped|View#super()|view **destroyed**|
|ServiceComponent|@ServiceScoped|Service#onCreate()|Service#onDestroy()|


##### ApplicationComponent

Application 전체의 생명주기를 lifetime으로 갖습니다. Application의 onCreate() 시점에 함께 생성되고 onDestroy() 되는 시점에 함께 파괴되게 됩니다.

##### ActivityRetainedComponent

`ApplicationComponent`의 하위 컴포넌트이며 Activity의 생명주기를 lifetime으로 갖습니다. 단, Acitivity의 configuration change(화면 가로/세로 전환 등) 시에는 파괴되지 않고 유지됩니다.

##### ActivityComponent

`ActivityReatinedComponent`의 하위 컴포넌트이며 Activity의 생명주기를 lifetime으로 갖습니다. Activity의 onCreate() 시점에 함께 생성되고 onDestroy() 되는 시점에 함께 파괴됩니다.

##### FragmentComponent

`ActivityComponent`의 하위 컴포넌트이며 Fragment의 생명주기를 lifetime으로 갖습니다. Fragment가 Activity의 붙는 onAttach() 시점에 함께 생성되고 onDestroy() 되는 시점에 함께 파괴됩니다.

##### ViewComponent

`ActivityComponent`의 하위 컴포넌트이며 View의 생명주기를 lifetime으로 갖습니다. View가 생성되는 시점에 함께 생성되고 파괴되는 시점에 함께 파괴됩니다.

##### ViewWithFragmentComponent

`FragmentComponent`의 하위 컴포넌트이며 Fragment의 view 생명주기를 lifetime으로 갖습니다. View가 생성되는 시점에 함께 생성되고 파괴되는 시점에 함께 파괴됩니다.

##### ServiceComponent

`ApplicationComponent`의 하위 컴포넌트이며 Service의 생명주기를 lifetime으로 갖습니다. Service의 onCreate() 시점에 함께 생성되고 onDestroy 시점에 함께 파괴됩니다.


<br><br><br>


##### Hilt with ViewModel

Hilt에서 ViewModel을 어떻게 사용하는지 살펴보겠습니다.

<br>

Hilt는 기본적으로 Jetpack에서 제공하는 ViewModel에 대해 의존성 주입을 제공하기 때문에, Jetpack의 ViewModel을 사용하는 경우 쉽게 구현할 수 있습니다.

먼저 ViewModel Injection을 사용하기 위해 app 단의 `build.gradle`에 dependency를 추가해줍니다.

```gradle
implementation 'androidx.hilt:hilt-lifecycle-viewmodel:1.0.0-alpha02'
kapt 'androidx.hilt:hilt-compiler:1.0.0-alpha02'
```

<br>

##### ViewModel Injection

Jetpack의 ViewModel은 Android SDK 내부적으로 ViewModel에 대한 lifecyle을 관리하고 있기 때문에 ViewModel의 생성 또한 Jetpack에서 제공하는 `ViewModelFactory` 를 통해 이루어져야 합니다.

Hilt에는 `ViewModelFactory` 가 이미 내부에 정의되어 있고, `ActivityComponent`와 `FragmentComponent`에 자동으로 install 되도록 되어있습니다.

위 기능을 사용하기 위해서는 `@ViewModelInject` 어노테이션을 사용하면 됩니다.


```kotlin
// 로그인 ViewModel 예시
class SignInViewModel @ViewModelInject constructor(
    private val repository: DefaultRepository,
    @Assisted private val savedStateHandle: SavedStateHandle
) : BaseViewModel() {
    ...
}
```

```kotlin
// 로그인 ViewModel을 사용한 SignInActivity 예시
@AndroidEntryPoint
class SignInActivity : AppCompatActivity(), View.OnClickListener {
    ...
    private val viewModel: SignInViewModel by viewModels()
    ...
}
```

위와 같이 `@ViewModelInject` 어노테이션 하나로 ViewModel을 간편하게 생성하는 모습을 볼 수 있습니다.

<br><br><br>

###### 참고 사이트
* [Android Developer](https://developer.android.com/training/dependency-injection/hilt-android)
* [알고싶은 승민: [Hilt] 안드로이드 Hilt 사용해보기, (아직 쓰지 말자...)](https://greedy0110.tistory.com/60)
* [루크의 코드테라피: Dagger - Hilt 간보기](https://two22.tistory.com/4)
* [allocProc: Android DI with Hilt - Hilt를 이용한 의존성 주입](https://medium.com/@jaeyeong951/android-di-with-hilt-hilt%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85-5076d9e9c46b)
* [찰스의 안드로이드: [Hilt] 4. 프로젝트에 Hilt 적용하기](https://www.charlezz.com/?p=44322)
* [HYPERCONNECT 기술 블로그: Dagger Hilt로 안드로이드 의존성 주입 시작하기](https://hyperconnect.github.io/2020/07/28/android-dagger-hilt.html)