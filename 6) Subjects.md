### 내용

- 다양한 타입의 subjects 와 사용하는 방법에 대해서 알아봅시다.
- subjects 를 감싸는 wrappers 인 relays 에 대해서도 알아보겠습니다.

[RxSwift: Reactive Programming with Swift, Chapter 3: Subjects](https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/3-subjects)

[[RxSwift] 3. Subjects](https://ios-development.tistory.com/98?category=916618)

[RxSwift/GettingStarted.md at main · ReactiveX/RxSwift](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/GettingStarted.md)

[import RxSwfit를 보고 도망치지 않는 방법 101가지](https://www.notion.so/import-RxSwfit-101-b7a10795a1b3457886cc882b1a1443a5)

***위의 글을 번역 및 요약한 글입니다.***

## Berfore entering

observalbes 는 RxSwift 의 기본적이지만 본질적으로 read-only 입니다. observables 가 생성하는 새로운 이벤에 대한 알림을 받기 위해서만 구독할 수 있습니다.

**앱을 개발할 때 일반적인 요구 사항은 런타임 중에 observable 에 새로운 값을 수동으로 추가하여 subscribers 에게 방출하는 것입니다.**

**즉, 우리가 원하는 것은 observable 하고 observer 한 역할을 할 수 있는 것입니다. 이것이 바로 Subject 입니다.(스스로 이벤트를 방출할 수도 있고, 다른 옵저버블을 구독할 수도 있다.)**

## Getting started

`PublishSubject` 를 만들어 보겠습니다.

```swift
let subject = PublishSubject<String>()
```

문자열 타입이므로 문자열만 receive 하고 publish 할 수 있습니다. 초기화 된 이후에 문자열을 받을 준비가 되있습니다.

```swift
let subject = PublishSubject<String>()

subject.on(.next("Is anyone listening?"))
```

위의 코드를 추가하므로써 subject 에 새로운 문자열이 추가됩니다. observers 가 없기 때문에 아직 아무것도 출력되지 않습니다. 다음 코드를 추가하여 subject 를 구독하여 만들어보겠습니다. 

```swift
let subscriptionOne = subject
  .subscribe(onNext: { string in
    print(string)
  })
```

`next` 이벤트들을 출력하지 않고있습니다. 무슨 일일까요?

PublishSubject 는 오직 현재 subscribers 에게 방출합니다. 그래서 이벤트가 추가되기 전에 구독하지 않았다면 얻을 수 없습니다.

```swift
let subscriptionOne = subject
  .subscribe(onNext: { string in
    print(string)
  })

// 🔥 구독한 다음 이벤트 추가.
subject.on(.next("1"))

/*
1
*/
```

이제야 이벤트가 출력되게 됩니다. `on(.next(_:))` 는 `next` 이벤트를 추가하고 element 를 매개변수로 전달하는 방법입니다. subscribe 과 마찬가지로 shortcut syntax 가 있습니다.

```swift
subject.onNext("2")

/*
1
2
*/
```

## What are subjects?

그렇다면 subjects 는 무엇인가요? 좀 더 알아봅시다. 앞서 언급했듯이 observable 과 observer 역할을 합니다. 그리고 위에서 event 를 수신하고 subscribe 하는 것을 봤습니다.

RxSwift 에는 4가지 subject 타입이 있습니다.

- `PublishSubject` : empty 로 시작하고, subscribers 에게만 새 elements 방출합니다.(ex. 10시에 경매 시작일 경우, 10시 1분에 접속했을 때 경매 시작 알림이 보내질 필요가 없는 경우)
- `BehaviorSubject` : 초기값으로 시작하여 이 값 또는 최신 element 를 새 subscribers 에게 replay 합니다.(ex. 뷰를 가장 최신의 데이터로 미리 채우기에 용이)
- `ReplaySubject` : buffer size 로  초기화되고 해당 크기까지 elements 의 버퍼를 유지하고 새 subscribers 에게 replay 합니다.
- `AsnycSubject` : 시퀀스의 마지막 `next` 이벤트만 방출하고, `subject` 가 `completed` 이벤트를 수신할때만 방출합니다.

RxSwift 는 `Relays` 라는 개념도 제공합니다. RxSwift 는 이를 `PublishRelay` 와 `BehaviorRelay`  두 가지로 제공합니다. 이것들은 각각 subject 를 wrap 하지만 next 이벤트만 수락하고 전달합니다. completed 또는 error 이벤트는 realy 에 추가할 수 없기 때문에 non-terminating 시퀀스에 적합합니다.

## Working with publish subjects

publish subjects 는 구독을 취소하거나 `completed` 또는 `error` 로 subject 가 종료될때까지 구독자가 구독한 시점부터 새로운 이벤트를 알림 받기를 원할 때 유용합니다.

다음 마블 다이어그램에서 맨 윗줄은 publish subject 이고 두 번째, 세 번째 줄은 subscribers 입니다. 

*(위쪽을 가리키는 화살표는 구독, 아래쪽을 가리키는 화살표는 방출된 입네트를 나타냅니다.)*

<img src="https://user-images.githubusercontent.com/69136340/178919261-2ad3625d-e680-413a-ab08-88c2ac85c804.png" width ="600">

첫 번째 구독자(= 두 번째 줄)는 1 을 추가한 후 구독하므로 해당 이벤트를 수신하지 않습니다. 2, 3 을 얻을 수 있습니다.

```swift
let subject = PublishSubject<String>()

let subscriptionOne = subject
  .subscribe(onNext: { string in
    print(string)
  })

subject.on(.next("1"))
subject.onNext("2")

// 🔥 subject 를 구독.
let subscriptionTwo = subject
  .subscribe { event in
    print("2)", event.element ?? event)
  }

/*
1
2
*/
```

이벤트에는 next 이벤트를 위해 내보낸 옵셔널 element 가 있습니다. 예상대로 `subscriptionTwo` 는 1 과 2 가 발생한 후에 구독했기 때문에 아무것도 출력하지 않습니다.

```swift
subject.onNext("3")

/*
3
2) 3
*/

```

아래의 코드를 추가하여 subscriptionOne 을 종료하고 subject 에 `next` 이벤트를 추가해보겠습니다.

```swift
subscriptionOne.dispose()

subject.onNext("4")

/*
2) 4
*/
```

publish subject 가 `completed` 또는 `error` 이벤트를 수신하면 해당 중지 이벤트를 새 subscribers 에게 방출하고 더 이상 `next` 이벤트를 방출하지 않습니다. 하지만, 이후에 subscribers 에게 stop 이벤트를 다시 방출합니다.

```swift
// 🔥 on(.completed) 의 편리한 메서드를 사용하여 completed 이벤트를 subject 에 추가.
subject.onCompleted()

// subject 가 이미 종료되었기 때문에 방출되지 않습니다.
subject.onNext("5")

subscriptionTwo.dispose()

let disposeBag = DisposeBag()

// 🔥 subject 구독.
subject
  .subscribe {
    print("3)", $0.element ?? $0)
  }
  .disposed(by: disposeBag)

subject.onNext("?")

/*
2) Completed
3) Completed
*/
```

위의 코드를 통해서 새로운 subscriber 를 다시 실행할까요? 아니요! 하지만 `completed` 이벤트는 replay 됩니다. 

따라서 종료될 때 알림을 받을 뿐만 아니라 구독할 때 이미 종료된 경우에도 stop 이벤트에 대한 핸들러를 포함하는 것이 좋습니다. 이것은 때때로 미묘한 버그가 될 수 있습니다. 00000000000

때로는 element 가 구독하기 전에 방출했음에도 불구하고 새 subscirbers 에게 가장 최근의 방출된 element 무엇인지 알려주고 싶은 경우가 있습니다. 이를 위한 몇가지 옵션이 있습니다.

publish subjects 는 새 subscribers 에게 값을 replay 하지 않습니다.따라서 사용자가 무언가를 탭함 또는 알림이 방금 도착했다 와 같은 이벤트를 모델링하는 데 좋은 선택입니다.

## Working with behavior subjects

behavior subjects 는 새 subsvribers 에게 최신의 next 이벤트를 replay 하는 점을 제외하고 publish subjects 와 비슷합니다.

*(위쪽을 가리키는 화살표는 구독, 아래쪽을 가리키는 화살표는 방출된 입네트를 나타냅니다.)*

<img src="https://user-images.githubusercontent.com/69136340/178919334-fed4f337-13a2-423e-a08b-3e411b090a59.png" width ="600">

맨 위의 줄이 subject 입니다. 두 번째 줄은 첫 번째 구독자이고, 1 이후지만 2보다 먼저 구독하므로 구독할 때 최근 이벤트인 1을 수신하고 subject 로 부터 2와 3을 수신합니다.

```swift
enum MyError: Error {
  case anError
}

// 🔥 element 가 있는 경우 출력하고, error 가 있는 경우 error 를 출력하거나 event 자체를 출력하는 함수.
func print<T: CustomStringConvertible>(label: String, event: Event<T>) {
  print(label, (event.element ?? event.error) ?? event)
}

let subject = BehaviorSubject(value: "Initial value")
let disposeBag = DisposeBag()
```

`BehaviorSubject` 는 항상 최신 element 를 방출하기 때문에 초기 값을 제공하지 않고는 생성할 수 없습니다. 생성 시 초기 값을 제공할 수 없다면 `PublishSubject` 를 대신 사용하거나 element 를 optional 로 모델링해야합니다.

다음의 코드를 추가해보겠습니다.

```swift
subject
  .subscribe {
    // event 의 element 가 있으면 출력 없으면 error 혹은 event 그 자체 출력.
    print(label: "1)", event: $0)
  }
  .disposed(by: disposeBag)

/*
1) Inital value
*/
```

subject 에 다른 elements 가 추가되지 않았기 때문에 subscriber 에게 초기 값을 replay 합니다.

무조건 초기 값을 replay 하는 것이 아니라 최신 next 이벤트를 replay 하는 것이기 때문에 아래의 코드의 결과는 다음과 같습니다.

```swift
enum MyError: Error {
  case anError
}

func print<T: CustomStringConvertible>(label: String, event: Event<T>) {
  print(label, (event.element ?? event.error) ?? event)
}

let subject = BehaviorSubject(value: "Initial value")
let disposeBag = DisposeBag()

// 🔥 next 이벤트 추가
subject.onNext("X")

subject
  .subscribe {
    print(label: "1)", event: $0)
  }
  .disposed(by: disposeBag)

/*
1) X
*/
```

다음의 예제 코드도 살펴보겠습니다.

```swift
enum MyError: Error {
  case anError
}

func print<T: CustomStringConvertible>(label: String, event: Event<T>) {
  print(label, (event.element ?? event.error) ?? event)
}

let subject = BehaviorSubject(value: "Initial value")
let disposeBag = DisposeBag()

subject.onNext("X")

subject
  .subscribe {
    print(label: "1)", event: $0)
  }
  .disposed(by: disposeBag)

// subject 에 error 이벤트 추가
subject.onError(MyError.anError)

// 
subject
  .subscribe {
    print(label: "2)", event: $0)
  }
  .disposed(by: disposeBag)

/*
1) X
1) anError
2) anError
*/
```

그렇다면 최신 값 이상을 보여주고 싶다면 어떻게 할까요? 예를 들어 검색 화면에서 가장 최근 사용한 검색어 5개를 표시하고 싶어한다면요?

replay subjects 에 대해서 알아봅시다!

## Working with replay subjects

replay subjects 는 선택하고 지정된 사이즈까지 자신이 방출하는 최신 elements 를 일시적으로 캐시하거나 버퍼링합니다.

*(위쪽을 가리키는 화살표는 구독, 아래쪽을 가리키는 화살표는 방출된 입네트를 나타냅니다.)*

<img src="https://user-images.githubusercontent.com/69136340/178919463-6b663e5d-52cb-407d-8a37-c766ba5ddeaf.png" width ="600">

첫 번째 구독자는 이미 replay subject 를 구독했으므로 elements 를 방출할 때 가져옵니다. 두 번째 구독자는 2 이후에 구독했지만 1 과 2 가 replay 됩니다.

replay subject 를 사용할 때 버퍼가 메모리에서 유지되는 것을 명심하세요. 예를 들어 이미지와 같이 각각의 인스턴스가 많은 메모리를 차지하는 일부 타입의 replay subject 에 대해 큰 버퍼 사이즈를 설정하는 경우 발등을 찍을 수도 있습니다.

주의해야할 또 다른 사항은 아이템 배열의 replay subject 를 만드는 것입니다. 각 방출된 요소는 array 가 되므로 버퍼 사이즈는 많은 배열을 버퍼링(버퍼에 저장되는 작업)합니다. 주의하지 않으면 여기서 메모리 프레셔를 생성하기 쉽습니다.

```swift
enum MyError: Error {
  case anError
}

func print<T: CustomStringConvertible>(label: String, event: Event<T>) {
  print(label, (event.element ?? event.error) ?? event)
}

// 🔥 버퍼 사이즈 2의 replay subject 를 생성.
let subject = ReplaySubject<String>.create(bufferSize: 2)
let disposeBag = DisposeBag()

subject.onNext("1")
subject.onNext("2")
subject.onNext("3")

// 🔥 subject 의 구독자 두 개 생성.
subject
  .subscribe {
    print(label: "1)", event: $0)
  }
  .disposed(by: disposeBag)

subject
  .subscribe {
    print(label: "2)", event: $0)
  }
  .disposed(by: disposeBag)

/*
1) 2
1) 3
2) 2
2) 3
*/
```

버퍼 사이즈가 2인 replay subject 를 만들었기 때문에 2 와 3 가 replay 됩니다.

아래의 코드를 추가해보겠습니다.

```swift
// ...

subject.onNext("4")

subject
  .subscribe {
    print(label: "3)", event: $0)
  }
  .disposed(by: disposeBag)

/*
// ...

1) 4
2) 4
3) 3
3) 4
*/
```

첫 번째, 두 번째 subscription 은 이미 구독되어 있기 때문에 정상적으로 수신하는 반면 새 구독자는 버퍼링된 마지막 2개의 elements 가 replay 됩니다.

다음의 코드를 중간에 추가해볼게요!

```swift
// ...

subject.onNext("4")

// 🔥 에러 추가
subject.onError(MyError.anError)

subject
  .subscribe {
    print(label: "3)", event: $0)
  }
  .disposed(by: disposeBag)

/*
// ... 

1) 4
2) 4
1) anError
2) anError
3) 3
3) 4
3) anError
*/
```

**…? 네..?**

replay subject 는 error 와 함께 종료되며, 이는 새 구독자에게 re-emit 됩니다. 그러나 버퍼는 여전히 존재하므로  stop 이벤트가 다시 발생하기 전에 새 구독자에게도 re-emit 됩니다.

우리가 원하는 결과가 나오지 않았습니다. 다음의 코드를 error 다음에 추가하여 해결할 수 있습니다.

```swift
// ...

subject.onNext("4")

subject.onError(MyError.anError)

// 🔥 dispose
subject.dispose()

subject
  .subscribe {
    print(label: "3)", event: $0)
  }
  .disposed(by: disposeBag)

/*
// ...

1) 4
2) 4
1) anError
2) anError
3) Object `RxSwift.(unknown context at $12cab2d50).ReplayMany<Swift.String>` was already disposed.
*/

// ReplayMany 는 replay subject 를 생성할 때 사용하는 내부 타입입니다.
```

미리 replay subject 에 대해 명시적으로 `dispose()` 를 호출하면 새 구독자는 이미 dipose 된 subject 를 나타내는 error 이벤트만 수신합니다.(MyError 와 별개임.)

이와 같이 subject 에 명시적으로 `dispose()` 를 호출하는 것이 일반적인 작업은 아닙니다. subscription 을 dispose bag 에 추가한 경우 뷰 컨트롤러 혹은 뷰 모델이 deallocate 될 때 모든 것이 dipose 되고 deallocate 됩니다.

publish, behavior, replay subject 를 사용하여 거의 모든 모델링을 할 수 있습니다. 그러나 단순하게 observable 타입에게 “너의 현재 값이 무엇이니?” 라고 묻고 싶을 때가 있습니다.

## Working with relays

relay 가 replay behavior 를 유지하면서 subject 를 래핑한다는 것을 이전에 배웠습니다. 일반적으로 observable 한 subjects 와 달리 `accept(:)` 메서드를 사용하여 relay 에 값을 추가합니다. 즉, `onNext(_:)` 를 사용하지 않습니다.

`PublishRelay` 는 `PublishSubject` 를 래핑하고 `BehaviorRelay` 는 `BehaviorSubject` 를 래핑합니다. **relay 를 래핑된 subject 와 구분하는 것은 절대 종료되지 않는 것을 보장한다는 것입니다.**

### PublishRelay

```swift
let relay = PublishRelay<String>()

let disposeBag = DisposeBag()

relay.accept("Knock knock, anyone home?")
```

PublishSubject 에 새 값을 추가하려면 accept(_:) 메서드를 사용합니다. 아직 구독자가 없으므로 아무것도 방출하지 않습니다.

```swift
let relay = PublishRelay<String>()

let disposeBag = DisposeBag()

relay
  .subscribe(onNext: {
    print($0)
  })
  .disposed(by: disposeBag)

relay.accept("1")

/*
1
*/
```

출력은 이전에 publish subject 와 동일합니다.

error 또는 completed 이벤트를 relay 에 추가할 수 있는 방법은 없습니다. 다음과 같이 추가하면 컴파일 오류가 발생합니다.

```swift
relay.accept(MyError.anError)
relay.onCompleted()
```

### BehaviorRelay

Behavior relay 도 `completed` 또는 `error` 로 종료되지 않습니다. behavior subject 를 래핑하기 때문에 초기 값으로 생성되고 최신 또는 초기 값을 새 구독자에게 replay 합니다. behavior subject 의 특별한 점은 현재 값을 언제든지 요청할 수 있다는 것입니다. 

```swift
let relay = BehaviorRelay(value: "Initial value")
let disposeBag = DisposeBag()

relay.accept("New initial value")

relay
  .subscribe {
  print(label: "1)", event: $0)
  }
  .disposed(by: disposeBag)

/*
1) New initial value
*/
```

 아래의 코드를 추가해보겠습니다.

```swift
// 1
relay.accept("1")

// 2
relay
  .subscribe {
    print(label: "2)", event: $0)
  }
  .disposed(by: disposeBag)

// 3
relay.accept("2")

/*
1) 1
2) 1
1) 2
2) 2
*/
```

그리고 behavior relay 를 사용하여 현재 값에 직접 액세스 해보겠습니다.

```swift
print(relay.value)

// 2
```
