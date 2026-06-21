# 코루틴 빌더와 Job

모든 코루틴 빌더 함수는 코루틴을 만들고 코루틴을 추상화한 Job 객체를 생성한다. 

코루틴은 일시 중단할 수 있는 작업으로 실행 도중 일시 중단된 후 나중에 이어서 실행될 수 있다. Job 객체는 이에 대응해 코루틴을 제어할 수 있는 함수와 코루틴의 상태를 나타내는 상태 값들을 외부에 노출한다.

### join을 사용한 코루틴 순차 처리

데이터베이스 작업을 순차적으로 처리, 캐싱된 토큰 값이 업데이트 된 이후에 네트워크 요청을 해야하는 상황은 각 작업을 하는 코루틴이 순차적으로 처리되어야 한다.

Job 객체는 join 함수를 제공해서, 먼저 처리되어야 하는 코루틴의 완료를 기다리게 한다.

### 순차 처리가 안된다면 어떤 문제가 있을까?

네트워크 요청 시 인증 토큰이 업데이트 되지 않고 네트워크 요청이 실행된다면? → 문제 발생.

join 함수를 호출한 코루틴은 join의 대상이 된 코루틴이 완료될 때까지 일시 중단된다는 것이다. 이 때문에 join 함수는 일시 중단이 가능한 지점에서만 호출될 수 있다. ← ?

### joinAll을 사용한 코루틴 순차 처리

서로 독립적인 여러 코루틴을 병렬로 실행한 후 실행한 요청이 모두 끝날 때까지 기다렸다가 다음 작업을 진행하는 것이 효율적이다. ex) SNS 이미지 업로드 / 이런 작업을 위해 코루틴 라이브러리는 복수의 코루틴의 실행이 모두 끝날때까지 호출부의 코루틴을 일시 중단 시키는 joinAll 함수를 제공한다.

### CoroutineStart.LAZY 사용한 코루틴 지연 시작

```kotlin
fun main() = runBlocking<Unit> {
    val startTime = System.currentTimeMillis()
    val lazyJob = launch(start = CoroutineStart.LAZY) {
        println("${Thread.currentThread().name} [${getElapsedTime(startTime)}}]")
    }
    delay(1000L)
    lazyJob.start()
}
```

### 코루틴 취소하기

코루틴 실행 도중 코루틴을 실행할 필요가 없어지면 즉시 취소해야 한다. 코루틴이 실행될 필요가 없어졌음에도 취소하지 않고 계속해서 실행하도록 두며 코루틴은 계소해서 스레드를 사용하며 이는 애플리케이션의 성능저하로 이어진다.

### cancel 사용해 job 취소하기

### cancelAndJoin 사용한 순차 처리

cancel 함수를 호출한 이후에 곧바로 다른 작업을 실행하면 해당 작업은 코루틴이 취소되기 전에 실행될 수 있다.  Job 객체에 cancel을 호출하면 코루틴이 즉시 취소되는 것이 아니라 Job 객체 내부의 취소 확인용 플래그를 ‘취소 요청됨’으로 변경함으로써 코루틴이 취소되어야 한다고만 알린다. 이후 미래 어느 시첨에 코루틴 취소 요청이 되었는지 확인하고 취소한다. 따라서 cancelAndJoin을 사용하여 이전 코루틴 취소가 완료된 뒤 다른 코루틴을 실행한다.

### 코루틴의 취소 확인

코루틴이 취소를 확인하는 시점은 일반적으로 일시 중단 시점이나 코루틴이 실행을 대기하는 시점이며, 이 시점들이 없다면 코루틴은 취소되지 않는다.

```kotlin
fun main() = runBlocking<Unit>{
    val whileJob = launch(Dispatchers.Default) {
        while (true ) {
            println("작업중")
        }
    }
    delay(100L)
    whileJob.cancel()
}
```

위 코드는 취소되지 않는다. 코루틴의 취소를 확인할 수 있는 시점이 없기 때문이다/

취소 확인하는 시점은..

1. delay 사용 → while문 반복할때마다 딜레이 됨해야 함.
2. yield 사용 → 반복할때마다 일시 정지
3. CoroutineScope.isActive 사용 → 코루틴을 잠시 멈추지 않고도 확인 가능.

### 코루틴의 상태

코루틴은 생성, 실행 중, 실행 완료 중, 실행 완료, 취소 중, 취소 완료 상태를 가질 수 있다.

| 코루틴 상태 | isActive | isCanceled | isCompleted |
| --- | --- | --- | --- |
| 생성 | false | false | false |
| 실행 중 | true | false | false |
| 실행 완료 | false | false | true |
| 취소 중 | false  | true | false |
| 취소 완료 | false | true | true |

isActive는 코루틴이 실행 중일때만 true, isCancelled는 코루틴 취소 중, 취소 완료에서만.

isCompleted는 실행 완료, 취소 완료애서만 true가 된다.