---
layout: post
title:  "[RxJava2] Disposable - 메모리 누수를 막아보자"
categories: [Android, RxJava2]
tags: [Kotlin]
---

* content
{:toc}

### Disposable

RxJava1에서 사용되던 `Subscription`이 RxJava2에서 **Disposable**로 바뀌었습니다.

옵저버블을 통해 데이터 스트림을 발행하고 `subscribe()` 함수들을 사용하여 구독할 때에
이 함수가 반환하는 값이 Disposable 인터페이스의 객체 입니다.

Disposable의 `dispose()` 함수를 통해 옵저버블에게 더이상 데이터를 발행하지 않도록 할 수 있습니다.

즉, 이를 통해 옵저버블을 사용함으로서 발생할 수 있는 메모리 누수를 막을 수 있습니다.





##### dispose()
* 옵저버블에게 더이상 데이터를 발행하지 않도록 구독을 해지하는 함수입니다.
* 옵저버블이 onComplete() 알림을 보냈을 때 자동으로 `dispose()`를 호출해 옵저버블과 구독자의 관계를 끊게 됩니다.

##### clear()
* add된 옵저버블을 해지합니다.
* 단, `isDisposed()`가 false이기 때문에 재사용이 가능합니다.

---

```kotlin
// BoardFragment 파괴되는 시점에 Disposable을 통해 리소스 해제
override fun onDestroy() {
    super.onDestroy()

    viewModel.run {
        clearCompositeDisposable()
    }
}

//
abstract class BaseViewModel : ViewModel() {
    protected val compositeDisposable = CompositeDisposable()

    /**
     * 쌓인 compositDisposable을 clear
     *
     * Activity/Fragment의 onDestroy 타이밍에 리소스 정리
     * dispose() 호출 시 compositeDisposable 재사용이 불가능하므로 clear()만 수행
     */
    fun clearCompositeDisposable() {
        compositeDisposable.clear()
    }

    /**
     * ViewModel이 제거될 때 호출되는 onCleared()
     *
     * compositeDisposable을 재사용할 일이 없으므로 dispose() 수행
     */
    override fun onCleared() {
        super.onCleared()
        compositeDisposable.dispose()
    }
}
```

---

```kotlin
// 게시글 목록 요청 API 예시
compositeDisposable.add(
    repository.getAllBoardList()
        .observeOn(AndroidSchedulers.mainThread())
        .subscribeWith(object : DisposableObserver<List<Board>>() {
            override fun onComplete() {
                // do nothing.
                Log.d(TAG, "onComplete() called.")
            }

            override fun onNext(boards: List<Board>) {
                boardList.postValue(boards)
            }

            override fun onError(e: Throwable) {
                Log.w(TAG, "onError() | ", e)
            }
        })
)
```

옵저버블의 subscribe의 반환값은 Disposable 이기 때문에 전역으로 선언해둔 `compositeDisposable`에 add를 통해 disposable을 하나의 객체로 모아줍니다.

해당 ViewModel을 포함하고 있는 View(BoardFragment)가 `onDestroy()` 될 때 `compositDisposable.clear()`를 호출해 리소스를 해제시키고, ViewModel이 `onCleared()` 될 때 `compositDisposable.dispose()`를 호출해 add 된 옵저버블들에 대한 리소스를 모두 해제시켜줍니다.

이와 같이 사용할 경우 Activity나 Fragment가 비정상적으로 종료되었을 경우 혹은 옵저버블의 `onComplete()` 나 `onError()` 를 타지 않는 경우에도 메모리 누수에 대한 방어가 가능합니다.

<br><br><br>

###### 참고 사이트
* [Beom Dev Log: 안드로이드의 RxJava 활용 - 12(메모리누수 막기)](https://beomseok95.tistory.com/60)
* [IOS를 Java: [RxAndroid2] 메모리 누수를 잡아보자. Disposable, RxLifeCycle, CompositeDisposable](https://altongmon.tistory.com/768)
* [ReactiveX: JavaDocs](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/disposables/Disposable.html)
* [레이니스트(뱅크샐러드) 기슬블로그: RxJava2를 도입하며](https://www.theteams.kr/teams/238/post/64047)
* [Niklas Baudy: RxJava 2 Disposable — Under the hood](https://medium.com/@vanniktech/rxjava-2-disposable-under-the-hood-f842d2373e64)