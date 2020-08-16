---
layout: post
title:  "[Android] DiffUtil을 이용해 RecyclerView 성능을 향상 시켜보자"
categories: [Android]
tags: [Kotlin]
---

### DiffUtil

#### RecyclerView의 성능을 향상시킬 수 있는 DiffUtil에 대해 알아보자.

### DiffUtil 이란?

oldList와 newList 리스트의 차이를 계산하고 oldList를 newList로 변환하는 업데이트 작업 목록을 출력할 수 있는 유틸성 클래스이다.







업데이트 작업 목록은 newList가 Insert, Remove, Update 모두 포함된다.

#### 리스트 아이템이 변경되는 갯수 마다의 시간값

|Action|avg|median|
|-----------------------|-----------|-----------|
|100 items and 10 modifications|0.39 ms|0.35 ms|
|100 items and 100 modification|3.82 ms|3.75 ms|
|100 items and 100 modifications without moves|2.09 ms|2.06 ms|
|1000 items and 50 modifications|4.67 ms|4.59 ms|
|1000 items and 50 modifications without moves|3.59 ms|3.50 ms|
|1000 items and 200 modifications|27.07 ms|26.92 ms|
|1000 items and 200 modifications without moves|13.54 ms|13.36 ms|


#### 사용법을 알아보도록 하자.

DiffUtil은 Eugene W.Myers의 difference algorithm 을 사용하여 하나의 목록을 다른 목록으로 변환하기 위한 최소 업데이트 수를 계산합니다.

이어서 사용법을 알아보겠습니다.

우선 아래와 같이 DiffUtil.Callback 을 상속받는 DiffCallback 클래스를 만들고 비교할 대상을 지정해 줍니다.

```kotlin
class DiffUtilCallback(
    private val oldData: List<Any>,
    private val newData: List<Any>
    ) : DiffUtil.Callback() {
    override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        val oldItem = oldData[oldItemPosition]
        val newItem = newData[newItemPosition]

        return if (oldItem is Board && newItem is Board) {
            oldItem.id == newItem.id
        } else {
            false
        }
    }

    override fun getOldListSize(): Int = oldData.size

    override fun getNewListSize(): Int = newData.size

    override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean =
        oldData[oldItemPosition] == newData[newItemPosition]
}
```

* getOldListSize(): 이전 목록의 갯수를 반환합니다.
* getNewListSize(): 새로운 목록의 갯수를 반환합니다.
* areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): 두 객체가 같은 항목인지 여부를 결정합니다.
* areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): 두 항목의 데이터가 같은지 여부를 결정합니다. `areItemsTheSame()` 함수가 **true**를 반환하는 경우에만 호출됩니다.
* getChangePayload(oldItemPosition: Int, newItemPosition: Int): 만약 `areItemTheSame()` 함수가 **true**를 반환하고 `areContentsTheSame()` 함수가 **false**를 반환하면 이 메소드가 호출되어 변경 내용에 대한 페이로드를 가져옵니다.


다음으로 RecyclerView Adapter 클래스의 `updateList()` 메소드를 보도록 하겠습니다.

```kotlin
fun updateList(items: List<ITEM>?) {
            items?.let {
                val diffCallback = DiffUtilCallback(this.items, items)
                val diffResult = DiffUtil.calculateDiff(diffCallback)

                this.items.run {
                    clear()
                    addAll(items)
                    diffResult.dispatchUpdatesTo(this@Adapter)
                }
            }
        }
```

`DiffUtil.calculateDiff()` 는 DiffCallback을 인자로 받고 결과 값으로 DiffUtil.DiffResult를 반환합니다.<br>
그 다음 RecyclerView Adapter의 아이템을 대체 시켜주고 dispatchUpdatesTo를 호출합니다. 여기서 this는 RecyclerView Adapter입니다.<Br>
RecyclerView Adapter를 파라미터로 받는 이유는 내부적으로 `dispatchUpdatesTo`를 호출하는 순간 뷰가 갱신되기 때문입니다.

즉, 아래 4가지 메소드 중 한가지를 호출하기 위해서라고 판단할 수 있습니다.
* mAdapter.notifyItemRangeInserted(position, count)
* mAdapter.notifyItemRangeRemoved(position, count)
* mAdapter.notifyItemMoved(fromPosition, toPosition)
* mAdapter.notifyItemRangeChanged(position, count, payload)

기존 방식은 OldList에서 변경된 포지션을 직접 위의 4가지 중 한가지를 불러서 뷰를 갱신 시켜줬지만 **DiffUtil** 을 사용하게되면 개발자가 신경쓰지 않아도 편리하게 갱신되기 때문에 좋은 것 같습니다.

#### 사용시 주의사항
* 리스트 사이즈가 큰 경우는 DiffResult를 가져올 때 Background Thread에서 실행하고 뷰 갱신할 때만 Main Thread에서 실행시켜야 합니다.
* DiffUtil 사용할 때 리스트의 Max Size는 2^26 (67,108,864) 입니다.



###### 참고 사이트
* [영훈 블로그: RecyclerView DiffUtil](https://kimyounghoons.github.io/android/android-diffUtil/)
* [꿈꾸는 개발자의 로 그: RecyclerView DiffUtil로 성능 향상하기](https://blog.kmshack.kr/RecyclerView-DiffUtil%EB%A1%9C-%EC%84%B1%EB%8A%A5-%ED%96%A5%EC%83%81%ED%95%98%EA%B8%B0/)