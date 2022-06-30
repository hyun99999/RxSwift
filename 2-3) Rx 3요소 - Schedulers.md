### 내용

- Rx 의 3요소(Observables, Operators, Schedulers) 에 대해서 알아보자.

[RxSwift: Reactive Programming with Swift, Chapter 1: Hello, RxSwift!](https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/1-hello-rxswift)
***위의 글을 번역 및 요약한 글입니다.***

Rx 코드의 세 가지 building blocks 은 Observables, Operators, Schedulers 입니다.

## Shedulers

Schedulers 는 Rx 에서 dispatch queues 또는 operation queues 에 해당하며 사용이 훨씬 간편합니다.  shedulers 는 특정 작업의 execution context 를 정의할 수 있습니다.

RxSwift 는 use cases 의 99% 를 다루는 사전 정의된 여러 스케줄러와 함께 제공되며 자신만의 스케줄러를 만들 필요가 없기를 바랍니다.

예를 들어, GCD 를 사용하여 주어진 큐에서 코드를 직렬(serial)로 실행하는 `SerialDispatchQueueScheduler` 에서 `next` 이벤트를 관찰하도록 지정할 수 있습니다.

`ConcurrentDispatchQueueScheduler` 는 코드를 동시에(concurrently)하게 실행하고, `OperationQueueScheduler` 는 주어진 OperationQueue 에서 구독을 예약할 수 있도록 합니다.

RxSwift 덕분에 다른 스케줄러에서 동일한 구독의 다른 작업을 예약하여 use case 에 맞는 최상의 성능을 얻을 수 있습니다.

RxSwift 는 subscriptions 와 schedulers 사이에서 dispatcher 역할을 하여 작업 조각을 올바른 context 로 보내고 서로의 ouput 과 원할하게 작동하도록 합니다.

<img src="https://user-images.githubusercontent.com/69136340/176757153-d12610a1-d42d-4c66-9c5d-476ccb60a1ca.png" wdith ="700">

- 파란색 network subscription 은 사용자 지정 `OperationQueue` 기반 스케줄러에서 실행되는 코드 1로 시작합니다.
- 이 블록(fetch JSON)의 데이터 output 은 concurrent background GCD queue 에 있는 다른 스케줄러에서 실행되는 next block 2 의 input 으로 사용됩니다.
- 마지막으로 파란색 코드 3 의 마지막 부분은 새 데이터로 UI 를 업데이트하기 위해 Main thread scheduler 에 예약됩니다.
