---
layout: post
title:  "[디자인패턴] 싱글톤 패턴"
categories: [디자인패턴]
tags: [CS]
---

### 싱글톤(Singleton) 패턴

 - 한번 생성하면 해당 클래스를 사용하는 곳은 하나의 객체로 사용하도록 하는 패턴
 - 장점
   - 하나의 객체만 생성하기 때문에 메모리 낭비가 방지됨
   - 두번째 사용부터는 객체 로딩 속도가 줄어들어 성능 향상 효과가 있음
 - 단점
   - 하나의 인스턴스가 너무 많은 일을 하거나 공유될 경우 사용하는 다른 클래스들간의 결합도가 높아져 수정이 어려워지고 유지보수가 어려워짐

```java
public class SingletonSample {
   private static SingletonSample instance;
   private SingletonSampe() {
     
   }
   public static SingletonSample getInstance() {
       if (instance == null) {
           instance = new SingletoneSample();
       }
       return instance;
   }
}
 ```
> 내가 주로 사용했던 방식이다. 관습적으로 사용했던 싱글톤 패턴인데 이번 기회에 싱글톤 패턴을 학습하며 **멀티 쓰레드** 환경에서는 의도치 않게 2개의 인스턴스가 생길 여지가 있다는 것을 알게 되었다.








### Thread Safe 싱글톤 패턴

- LazyInitailization 싱글톤
    ```java
    public class LazyInitialization {
      private static LazyInitialization instance;
      private LazyInitialization() {
    
      }
      public static synchronized LazyInitialization getInstance() {
          if (instance == null) {
          instance = new LazyInitailization();
          }
          return instance;
      }
    }
    ```
- Thread Safe한 가장 쉽게 생각할 수 있는 싱글톤 패턴이다.
- 싱글톤 객체의 인스턴스를 호출할 때마다 synchronized 되며 synchronized 특성상 속도 저하가 발생할 수 있다.<br>
*(통상 100배 정도의 성능 저하가 발생할 수 있다고 한다..)*

- Double Checked Locking 싱글톤
    ```java
    public class DoubleCheckedLocking {
      private static DoubleCheckedLocking instance;
      private DoubleCheckedLocking() {
        
      }
      public static DoubleCheckedLocking getInstance() {
          if (instance == null) {
              synchronized (DoubleCheckedLocking.class) {
                  if (instance == null) {
                      instance = new DoubleCheckedLocking();
                  }
              }
          }
          return instance;
      }
    }
    ```
  - 인스턴스의 첫 생성 이후 synchronized를 타지 않기 때문에 성능 저하를 완화한 기법이다.
  - new 키워드를 통해 인스턴스를 생성할 때 아래와 같은 순서를 통해 생성하게 된다.
    1. 메모리 공간 확보
    2. 변수에 메모리 공간 링크
    3. 해당 메모리에 오브젝트 생성
  - 이때 재배치(reordering) 과정에 의해 잘못된 객체 참조 문제가 발생할 수 있다.
  - 그렇기 때문에 선호되지 않는 방식이다.
  
- volatile을 이용한 DoubleCheckedLocking 싱글톤
    ```java
    public class DoubleCheckedLocking {
      private volatile static DoubleCheckedLocking instance;
      private DoubleCheckedLocking() {
        
      }
      public static DoubleCheckedLocking getInstance() {
          if (instance == null) {
              synchronized (DoubleCheckedLocking.class) {
                  if (instance == null) {
                      instance = new DoubleCheckedLocking();
                  }
              }
          }
          return instance;
      }
    }
    ```
  - volatile 키워드를 이용하여 선언할 경우 오브젝트 생성, 메모리에 배치까지 바로 업데이트가 되어 reordering 문제가 해소될 수 있다.
  - 하지만 이 것은 컴파일러의 reordering이 해결된 것으로 cpu 레벨에서도 성능 향상을 위해 비순차 실행 기법을 활용하고 있으므로 컴파일러와 무관하게 런타임 중에 reordering 문제가 발생할 가능성은 여전히 존재한다.
  - 또한, volatile의 특성상 메모리에 직접 접근(캐시 미사용)하여 성능이 느리다는 단점이 있다.
  
- Lazy Holder 패턴 싱글톤
    ```java
    public class Single {
      private Single() {
        
      } 
      public static Single getInstance() {
          return LazyHolder.INSTANCE;
      }
      private static class LazyHolder {
          public static final Single INSTANCE = new Single();
      }
    }
    ```
  - 현재까지의 싱글톤 방법 중 가장 완벽하다고 이야기 하는 패턴이다.
  - Single 클래스에는 LazyHolder 클래스의 변수가 없기 때문에 Single 클래스 로딩 시 LazyHolder 클래스는 **초기화 되지 않는다**.
  - LazyHolder 클래스는 Single 클래스의 getInstance() 메소드가 호출 될 때에 해당 메소드에서 LazyHolder.INSTANCE 를 참조하는 순간 클래스가 로딩되며 초기화가 진행 된다.
  - 클래스 로더에 의한 초기화는 내부적으로 동기화가 이루어지므로 **Thread Safe한 싱글톤**을 안전하게 만들 수 있다.
  
- enum 을 이용한 싱글톤
    ```java
    public enum Singleton {
      INSTANCE;
      public static Singleton getInstance() {
          return INSTANCE;
      }
    }
    ``` 
  - enum 내 상수는 `static final`로 선언되어 있어 전역에서 접근이 가능하고 클래스가 로드될 때 초기화가 이루어진다. 그리고 enum은 컴파일 시에 값을 가지며 런타임 시에 값이 설정되면 안되기에 생성자로 인한 생성도 불가능하다.
  - 위의 다른 싱글톤에서 private 생성자를 사용하였지만 런타임에 **[Reflection](https://taehyungk.github.io/2019/12/11/android-reflection/)**을 통해 private 생성자를 깨트릴 수 있는 여지가 있지만 enum을 사용하게 되면 Reflection을 통해 private 생성자 깨트림 문제가 발생하지 않는다.
