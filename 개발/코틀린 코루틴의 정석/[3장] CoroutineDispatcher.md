# CoroutineDispatcher

디스패처 : dispatch + -er → 무언가를 보내는 주체

CoroutineDispatcher 는 코루틴을 스레드로 보낸다. 코루틴은 일시 중단이 가능한 작업이기 때문에 스레드가 있어야 실행된다. 자신에게 실행 요청된 코루틴들을 작업 대기열에 넣고, 사용할 수 있는 스레드가 생기면 스레드로 코루틴을 보내 실행될 수 있게 하는 역할을 한다.

### 제한된 디스패처와 무제한 디스패처

CoroutineDispatcher는 2가지 종류가 있음. 제한된 디스패처, 무제한 디스패처.

제한된 디스패처 → 사용 가능한 스레드, 스레드 풀 제한

무제한 디스패처 → 스레드 제한 없음. 아무 스레드에서나 실행되는게 아니라, 실행 요청된 코루틴이 이전 코드가 실행되던 스레드에서 계속 실행되도록 한다. 때문에 스레드가 매번 달라질 수 있고. 특정 스레드로 제한되어 있지 않다.

### 부모 코루틴, 자식 코루틴

바깥쪽의 코루틴을 부모 코루틴, 내부에서 생성되는 새로운 코루틴을 자식 코루틴이라고 한다. 구조화는 코루틴을 계층 관계로 만드는 것 뿐만 아니라 부모 코루틴의 실행 환경을 자식 코루틴에 전달하는 데도 사용된다.

```kotlin
val multiThreadDispatcher = newFixedThreadPoolContext(
        2, "MultiThread"
    )
    launch(multiThreadDispatcher) {
        println("[${Thread.currentThread().name}] 실행")
        launch{
            println("[${Thread.currentThread().name}] 실행")
        }
        launch{
            println("[${Thread.currentThread().name}] 실행")
        }
    }
```

코루틴 라이브러리는 개발자가 직접 CoroutineDIspatcher 객체를 생성하는 문제의 방지를 위해 미리 정의된 CoroutineDispatcher의 목록을 제공한다.

Dispatchers.IO / Dispatchers.Default / Dispatchers.Main

### Dispatchers.IO

멀티 스레드 프로그래밍이 가장 많이 사용되는 작업은 입출력 작업이다.

Dispatchers.IO 가 최대로 사용할 수 있는 스레드의 수는 JVM에서 사용 가능한 프로세서의 수와 64 중 큰 값으로 설정된다. 즉 최소 64개의 스레드를 사용할 수 있다.

### Dispatchers.Default

CPU 바운드 작업 ( 대용량 데이터를 처리하는 작업 ) 이 필요할 때 사용한다.

### 공유 스레드풀