# CoroutineContext

```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job 

public fun <T> CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> T
): Deferred<T> 
```

CoroutineContext는 코루틴을 실행하는 실행 환경을 설정하고 관리하는 인터페이스로 CoroutineContext 객체는 CoroutineDispatcher, CoroutineName, Job 등의 객체를 조합해 코루틴의 실행 환경을 설정한다. 즉, 코루틴을 실행하고 관리하는 데 핵심적인 역할을 하며 코루틴의 실행과 관련된 모든 설정은 CoroutineContext 객체를 통해 이뤄진다.

## CoroutineContext의 구성 요소

- CoroutineName : 코루틴의 이름을 설정한다.
- CoroutineDispatcher : 코루틴을 스레드에 할당해 실행한다.
- Job : 코루틴의 추상체로 코루틴을 조작하는데 사용된다.
- CoroutineExceptionHandler : 코루틴에서 발생한 예외를 처리한다.

## CoroutineContext가 구성 요소를 관리하는 방법

CoroutineContext 객체는 키-값 쌍으로 구성 요소를 관리한다. 각 구성 요소는 고유한 키를 가지며, 키에 대해 중복된 값은 허용되지 않는다. 

## CoroutineContext 구성

CoroutineContext 객체는 키 - 값 쌍으로 구성 요소를 관리하지만 키에 값을 직접 대입하는 방법을 사용하지 않는다. 대신 CoroutineContext 객체 간에 더하기 연산자를 사용해 CoroutineContext 객체를 구성한다.

## CoroutineContext 구성 요소 덮어 씌우기

만약 CoroutineContext 객체에 같은 구성 요소가 둘 이상 더해진다면 나중에 추가된 구성요소가 이전의 값을 덮어씌운다.

## CoroutineContext 구성 요소의 키