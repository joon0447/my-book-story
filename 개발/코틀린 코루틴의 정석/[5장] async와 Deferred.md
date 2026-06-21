# async와 Deferred

launch 코루틴 빌더를 통해 생성되는 코루틴은 기본적으로 작업 실행 후 결과를 반환하지 않는다. 하지만 우리가 코루틴을 다룰 때는 코루틴으로부터 결과를 수신해야 하는 경우가 빈번하다.

코루틴 라이브러리는 비동기 작업으로부터 결과를 수신해야 하는 경우를 위해 async 코루틴 빌더를 통해 코루틴으로부터 결과값을 수신받을 수 있도록 한다. async 함수를 사용하면 결괏값이 있는 코루틴 객체인 Deferred가 반환되며, Deferred 객체를 통해 코루틴으로부터 결과값을 수신할 수 있다.

### async를 사용해 Deferred 만들기

```kotlin
public fun <T> CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> T
): Deferred<T> 
```

launch 함수와 마찬가지로 async 는 CoroutineDispatcher를 설정할 수 있고, start 인자로 CoroutineStart.LAZY를 설정할 수 있으며, block 람다식을 가진다.

launch는 코루틴이 결괏값을 직접 반환할 수 없는 반면에 async 는 코루틴이 결괏값을 직접 반환할 수 있다는 것이다. launch 코루틴 빌더는 코루틴에서 결괏값이 반환되지 않기 때문에 Job 객체를 반환하는데 async 코루틴 빌더는 코루틴에서 결괏값을 담아 반환하기 위해 Deferred<T> 타입의 객체를 반환한다.

```kotlin
val networkDeferred: Deferred<String> = async(Dispatchers.IO) {
        delay(1000L)
        return@async "OK"
    }
```

이 코드는 1초 지연 후에 “OK”를 반환한다.

#### await를 사용한 결괏값 수신

Deferred 객체는 미래의 어느 시점에 결괏값이 반환될 수 있음을 표현하는 코루틴 객체이다. Deferred 객체는 결괏값 수신의 대기를 위해 await 함수를 제공한다. await 함수는 await의 대상이 된 Deferred 코루틴이 실행 완료될 때까지 await 함수를 호출한 코루틴을 일시 중단하며, Deferred 코루틴이 실행 완료되면 결괏값을 반환하고 호출부의 코루틴을 재개한다.

```kotlin
fun main() = runBlocking<Unit> {
    val networkDeferred: Deferred<String> = async(Dispatchers.IO) {
        delay(1000L)
        return@async "OK"
    }

    val result = networkDeferred.await()
    println(result) // "출력 : OK "
}

```

### Deferred는 특수한 형태의 Job이다

Deferred 객체는 코루틴으로부터 결괏값 수신을 위해 Job 객체에서 몇가지 기능이 추가됐을 뿐, 여전히 Job 객체의 일종이다.

### await을 사용해 복수의 코루틴으로부터 결괏값 수신하기

### awaitAll을 사용한 결괏값 수신

만약 10개의 사이트에서 관람객을 등록받았다면 열 줄에 거쳐 await 함수를 호출해야 한다. 이렇게 같은 코드를 반복하는 것 가독성 좋지 않음… 이 문제를 해결하기 위해 awaitAll 함수를 제공한다.

awaitAll 함수는 가변 인자로 Deferred 타입의 객체를 받아 인자로 받은 모든 Deferred 코루틴으로부터 결과가 수신될 때까지 호출부의 코루틴을 일시 중단한 후 결과가 모두 수신되면 Deferred 코루틴들로부터 수신한 결괏값들을 List로 만들어 반환하고 호출부의 코루틴을 재개한다.

### withContext로 async-await 대체하기

async-await 쌍은 새로운 코루틴을 생성해 작업을 처리하지만, withContext 함수는 실행중이던 코루틴을 그대로 유지한 채로 코루틴의 실행환경만 변경해 작업을 처리한다.

### withContext 사용 시 주의점

withContext 함수는 새로운 코루틴을 만들지 않기 때문에 하나의 코루틴에서 withContext 함수가 여러 번 호출되면 순차적으로 실행된다. 즉, 복수의 독립적인 작업이 병렬로 실행되어야 하는 상황에 withContext를 사용할 경우 성능에 문제를 일으킬 수 있다.