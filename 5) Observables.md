### 내용

- Observable 을 생성하고 구독하는 몇가지 예를 살펴봅시다.

[RxSwift: Reactive Programming with Swift, Chapter 1: Hello, RxSwift!](https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/1-hello-rxswift)

[RxSwift/GettingStarted.md at main · ReactiveX/RxSwift](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/GettingStarted.md)

[[RxSwift] 2. Observables](https://ios-development.tistory.com/97?category=916618)

***위의 글을 번역 및 요약한 글입니다.***

## What is an observable?

**Observable 은 Rx 의 핵심입니다. observable 이 무엇인지 어덯게 생성하고, 어떻게 사용하는지 알아보겠습니다.** 

Rx 에서 언급되는 “observable”, “observable sequence”, “sequence”, “stream” 는 서로 같은 의미입니다. 

Observable 은 일정 기간 동안 이벤트를 생성하며 이를 방출이라고 합니다. 이벤트는 숫자나 사용자 정의 유형의 인스턴스와 같은 값을 포함하거나 탭과 같은 제스처를 포함할 수 있습니다.

이것을 개념화하는 가장 좋은 방법 중 하나는 아래와 같은 타임라인에 값이 표시되는 marble diagrams 를 사용하는 것입니다.

<img src="https://user-images.githubusercontent.com/69136340/178294982-ddefd292-2292-41a7-900a-bf573cbe9133.png" width ="700">


왼쪽에서 오른족 화살표는 시간을 나타내고 번호가 매겨진 원은 sequence 의 elements 를 의미합니다.

sequence 는 0개 이상의 element 를 가질 수 있습니다. `error` 와 `completed` 이벤트가 수신되면 시퀀스는 다른 요소를 생성할 수 없습니다.

## Event

### next

observable 이 observer 에게 elements 를 포함하는 next event 를 방출합니다.

### completed

observable 을 구독한 observer 에게 완료되었음을 알립니다. 이를 completed event 라고 합니다. 오른쪽 끝에 선이 있는 것은 tap 이벤트 3개를 방출한 후 종료되는 것음 의미합니다. 

<img src="https://user-images.githubusercontent.com/69136340/178295057-0810ce5c-f486-4af4-9067-c02b00a4d059.png" width ="700">

### error

observer 에게 오류가 포함된 error event 를 방출하여 알립니다. 에러 이벤트는 `x` 로 표시합니다.

<img src="https://user-images.githubusercontent.com/69136340/178295139-f6a9ef59-9036-48b9-9a37-098a5e6b7a03.png" width ="700">

Observable 이 종료되면 더 이상 이벤트를 생성할 수 없습니다. 

RxSwift 에서 Event 는  enumeration cases 로 다음과 같이 구현되어 있습니다.

```swift
enum Event<Element>  {
    case next(Element)      // next element of a sequence
    case error(Swift.Error) // sequence failed with error
    case completed          // sequence terminated successfully
}
```

## Creating observables

### 1) just

```swift
let one = 1
let two = 2
let three = 3

let observable = Observable<Int>.just(one)
```

`just` operator 는 하나의 element 만 포함하는 sequence 를 생성합니다. `Observable` 의 static method 입니다.

### 2) of

```swift
let observable2 = Observable.of(one, two, three)
```

이번에는 타입을 명시적으로 선언하지 않았습니다. 여러가지 정수를 제공하기 떄문에 `Observable<Int>` 라고 생각할 수 있습니다.

`observable2` 의 추론된 유형을 보게되면 배열이 아닌 Int 의 Observable 임을 알 수 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/178295251-d114aaa0-6b28-4c1c-a3e5-8956c41b2fd5.png" width ="500">

 `of` operator 는 가변 파라미터가 있고, Swift 는 이를 기반으로 `observable` 의 타입을 유추할 수 있습니다.
 
<img src="https://user-images.githubusercontent.com/69136340/178295279-aec3d76c-d57f-403e-81a9-8d298e2f1cc3.png" width ="500">

_가변 변수. elements: Int…_

observable 배열을 원할 경우 `of` 를 사용해서 전달할 수 있습니다.

```swift
let observable3 = Observable.of([one, two, three])
```

`observable3` 는 `[Int]` 의 `Observable` 입니다. `just` 연산자도 배열을 단일 element 로 사용할 수도 있습니다. 단순하게 생각해서 배열 자체가 단일 요소이지 안의 내용이 요소가 아닙니다.

### 3) from

```swift
let observable4 = Observable.from([one, two, three])
```

`from` operator 는 배열에서 개별적으로 관찰 가능한 elements 를 만듭니다. `observable4` 는 [Int] 가 아닌 Int 의 Observable 임을 알 수 있습니다. `from` 연산자는 배열만 사용합니다.

## Subscribing to observables

RxSwift 에서 subscribe 은 observers 에게 broadcoasts 하는 `NotificationCenter` 와 매우 유사합니다. `addObserver()` 대신에 `subscribe()` 를 사용하게 됩니다. 더 중요한 차이는 `**Observable` 은 `subscriber` 가 있을때까지 events 를 보내거나 작업을 수행하지 않는 다는 것입니다.**

observable 은 `next`, `error`, `completed` 이벤트를 방출한다는 것을 앞에서 알아보았습니다. `next` 이벤트는 핸들러에 방출되는 element 를 전달하고 `error` 이벤트는 error instance 가 포함됩니다.

```swift
let one = 1
let two = 2
let three = 3

let observable = Observable.of(one, two, three)

// 🔥 다음과 같이 observable 을 구독할 수 있습니다.
// observable 에서 방출하는 각 event 를 출력합니다.
observable.subscribe { event in
  print(event)
}

/*
next(1)
next(2)
next(3)
completed
*/
```

`subscribe` operator 를 살펴보면 Int 타입의 event 를 수신하고 아무것도 반환하지 않는 클로저 매개변수를 사용하고 Disposable 을 반환하는 것을 볼 수 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/178295453-4a2f454a-6946-474f-a04a-12e6dda47118.png" width ="500">

결과에 대해서 살펴보겠습니다. 

```swift
/*
next(1)
next(2)
next(3)
completed
*/
```

observable 은 각 요소들에 대해서 `next` 이벤트를 방출한 뒤 completed 이벤트를 방출하고 종료됩니다.

elements 에 직접 액세스하는 코드를 살펴 보겠습니다.

```swift
observable.subscribe { event in
  if let element = event.element {
    print(element)
  }
}

/*
1
2
3
*/
```

`Event` 에는 element 프로퍼티가 없습니다. `next` 이벤트에만 element 가 있기 때문에 optional 값입니다. 따라서 옵셔널 바인딩을 상요하여 래핑을 해제합니다.

이것은 좋은 패턴이고, 자주 사용하기 때문에 RxSwift 에서 shortcut 이 있습니다. observable 이 방출하는 각 타입의 이벤트마다 subscribe 연산자가 있습니다. 

위의 코드를 변경해보겠습니다.

```swift
observable.subscribe(onNext: { element in
  print(element)
})
```

이제 여러분들은 `next` 이벤트의 elements 만 핸들링하고 다른 것들은(옵셔널 바인딩과 같은 과정) 무시할 수 있습니다. `onNext` 클로저는 `next` 이벤트의 elements 를 인수로 받기 때문에 event 에서 수동으로 추출할 필요가 없습니다.

### 1) empty

한 개와 여러 element 의 observable 을 만드는 방법을 알아보았습니다. 그렇다면 요소가 없는 observable 은 어떨까요?

비어있는 observable 시퀀스는 `empty` 연산자로 생성할 수 있고 오직 `completed` 이벤트만 방출합니다.

```swift
let observable = Observable<Void>.empty()
```

`empty` 는 타입을 유추할 수 있는 것이 없기 때문에 위의 코드처럼 명시적으로 정의해야 합니다.

```swift
observable.subscribe(
  // 1
  onNext: { element in
    print(element)
  },

  // 2
  onCompleted: {
    print("Completed")
  }
)

// Completed
```

아무것도 없으니 element 가 출력되지 않고 completed 이벤트만 발생하게 됩니다.

그렇다면 빈 observable 의 용도가 무엇일까요? 즉시 종료되거나 의도적으로 값이 0인 observable 을 반환하려는 경우 사용됩니다.

### 2) never

`empty` 연산자와는 반대로 `never` 연산자는 아무것도 방출하지 않고 절대 종료되지 않는 observable 을 생성합니다. 무한으로 지속되는 시간을 나타낼 수 있습니다.

```swift
let observable = Observable<Void>.never()

observable.subscribe(
  onNext: { element in
    print(element)
  },
  onCompleted: {
    print("Completed")
  }
)

//
```

아무것도 출력되지 않습니다. Completed 도 출력되지 않습니다.

### 3) range

지금까지 특정 elements 또는 values 의 observables 으로 작업했습니다. 하지만, range of values 로 부터 observable 을 생성하는 것이 가능합니다.

```swift
// 🔥 시작 정수 값과 연속 개수를 사용하는 range 연산자를 사용하여 Observable 생성.
let observable = Observable<Int>.range(start: 1, count: 10)

// 🔥 방출된 element 에 대해 n 번째 피보나치 수를 계산하고 출력. 
observable
  .subscribe(onNext: { i in  
    // 2
    let n = Double(i)

    let fibonacci = Int(
      ((pow(1.61803, n) - pow(0.61803, n)) /
        2.23606).rounded()
    )

    print(fibonacci)
})
```

## Disposing and terminating

observable 의 작업을 트리거하는 subscription 은 `error` 또는 `completed` 이벤트가 observable 을 종료할 때까지 새 이벤트를 방출합니다.  그러나 **subscription 을 취소하여 observable 을 수동으로 종료할 수 있습니다.**

명시적으로 구독을 취소하기 위해서 `dispose()` 를 호출합니다. 구독이 취소되거나 dispose 된 후 observable 은 이벤트 방출을 중단합니다.

```swift

let observable = Observable.of("A", "B", "C")

let subscription = observable.subscribe { event in
  print(event)
}

// 🔥 종료
subscription.dispose()
```

**각 subscription 을 개별적으로 관리하는 것은 지루할 수 있으므로 RxSwift 는 `DisposeBag` 타입이 있습니다.** dispose bag 은 일반적으로 `disposed(by:)` 메서드를 사용하여 disposables 를 담고 있습니다. 그래서 dispose bag 이 할당 해제되려고 할 때 각각에 대해 `dispose()` 를 호출합니다.

```swift
let disposeBag = DisposeBag()

Observable.of("A", "B", "C").subscribe {
    print($0)
}
.disposed(by: disposeBag)
// 🔥 dispose bag 에 담긴 disposables 각각에 대해 dispose() 메서드를 호출.
```

이러한 observable 을 생성하고 구독하고 즉시 dispose bag 에 넣는 패턴을 자주 사용하게 될 것입니다.

**그렇다면 왜 disposables 을 신경쓸까요?**

dispose bag 에 추가하는 것을 잊었거나 subcription 이 끝났을 때 수동으로 `dispose` 를 호출하거나 다른 방식으로 observable 이 어떤 시점에 종료되도록 하면 메모리 누수가 발생할 수 있습니다.

***(RxSwift 깃허브에서 발췌)***

시퀀스가 유한한 시간에 종료되면 `dispose` 를 호출하지 않거나 `diposed(by: disposeBag)` 를 사용하지 않아도 영구적인 resource leaks 가 발생하지 않습니다. 하지만, 이러한 리소스는 elements 의 생산을 complete 하거나 error 를 반환하여 시퀀스가 완료될 때까지 사용됩니다. (즉, completed 와 error event 가 발생하기 전까지는 사용됩니다.)

일련의 버튼 탭과 같은 시퀀스가 자체적으로 종료되지 않으면 `dipose` 가 수동으로, `disposeBag`  내부에서 자동으로 , `takeUntil` operator 를 사용하여 또는 다른 방법으로 호출되지 않는 한 영구적으로 할당됩니다.

그래서 시퀀스가 유한한 시간에 종료되는 경우에도 사용하는 것이 좋습니다.

---

**잊어버리더라도 걱정마세요!** Swift 컴파일러가 사용되지 않는 disposables 에 대해 경고해줄 것입니다.

### create

이전 예시에서는 특정 `next` 이벤트 elelments 를 가진 observables 을 만들었습니다. `**create` 연산자를 사용하는 것은 subscribers 에게 방출할 모든 이벤트를 지정하는 또다른 방법입니다.**

```swift
let disposeBag = DisposeBag()

Observable<String>.create { observer in
  // ...
}
```

<img src="https://user-images.githubusercontent.com/69136340/178295596-c5f4f79c-6098-49e0-8e1f-e7c2832f0156.png" width ="500">

`subscribe` 매개변수는 `AnyObserver` 를 사용하고 `Disposable` 을 반환하는 탈출 클로저입니다. `AnyObserver` 는 observable sequence 에 값을 추가하는 것을 용이하게하는 제네릭 타입입니다.

이를 통해 observer 에게 이벤트를 클로저안에서 방출해줄 수 있습니다.

`create` 의 구현부를 변경해보겠습니다.

```swift
Observable<String>.create { observer in
  observer.onNext("1")

  observer.onCompleted()

  observer.onNext("?")

  // 🔥 observable 이 종료되거나 disposed 될 때 어떤 일이 발생하는지 정의하는 disposble 을 반환합니다.
  // 🔥 이 경우 별도의 정리가 필요하지 않으므로 empty disposble 리턴.
  return Disposables.create()
}
```

두 번째 `onNext` 의 element `?` 가 subscribers 에게 방출될 수 있을까요? 아래의 코드를 뒤에 추가해서  확인해봅시다.

```swift
.subscribe(
  onNext: { print($0) },
  onError: { print($0) },
  onCompleted: { print("Completed") },
  onDisposed: { print("Disposed") }
)
.disposed(by: disposeBag)

/*
1
Completed
Disposed
*/
```

결과는 첫 번째 next 이벤트 요소와 Completed 그리고 Disposed 가 출력됩니다. **두 번째 `next` 이벤트는 observable 이 completed 이벤트를 방출하고 종료되기 전에 추가되지 않았기 때문에 출력되지 않습니다.**

observer 에게 error 를 추가하면 어떻게 될까요? 

```swift
// 🔥 error 추가
enum MyError: Error {
  case anError
}

let disposeBag = DisposeBag()

Observable<String>.create { observer in
  observer.onNext("1")
  observer.onCompleted()

  // 🔥 error 추가
  observer.onError(MyError.anError)
  observer.onNext("?")

  return Disposables.create()
}
.subscribe(
  onNext: { print($0) },
  onError: { print($0) },
  onCompleted: { print("Completed") },
  onDisposed: { print("Disposed") }
)
.disposed(by: disposeBag)

/*
1
anError
Disposed
*/
```

**이제 observable 이 종료되기 전에 error 를 방출합니다.**

그렇다면 completed 또는 error 이벤트를 추가하지 않고 disposeBag 에 구독을 추가하지 않으면 어떻게 될까요? 주석처리를 해서 알아봅시다.

```swift
  enum MyError: Error {
    case anError
  }

  let disposeBag = DisposeBag()

  Observable<String>.create { observer in
    observer.onNext("1")

//    observer.onError(MyError.anError)

// 1 끝나지 않음.
//    observer.onCompleted()

    observer.onNext("?")

    return Disposables.create()
  }
  .subscribe(
    onNext: { print($0) },
    onError: { print($0) },
    onCompleted: { print("Completed") },
    onDisposed: { print("Disposed") }
  )
// 2 폐기되지 않음.
//  .disposed(by: disposeBag)

/*
1
?
*/
```

`onNext` 이벤트인 `?` 가 출력되었습니다.

메모리 누수(memory leak)가 발생하였습니다! observable 은 절대 끝나지 않을 것이고, disposable 은 절대 폐기되지 않을 것입니다.

## Creating observable factories

subscribers 를 기다리는 observable 을 만드는 대신 **각 subscriber 에게 새로운  observable 을 제공하는 observable factories 를 만드는 것이 가능합니다.**

### deferred

lazy var 변수와 같이 observable 이 subscribe 하는 순간 실행됩니다.

```swift
let disposeBag = DisposeBag()

var flip = false

let factory: Observable<Int> = Observable.deferred {

// 🔥 factory 가 구독될 때마다 toggle.
  flip.toggle()

// 🔥 flip 참, 거짓에 따라 다른 observable 를 반환합니다.
  if flip {
    return Observable.of(1, 2, 3)
  } else {
    return Observable.of(4, 5, 6)
  }
}
```

외부적으로 observble factory 와 일반 observable 을 구별할 수 없습니다. 다음의 코드를 하단에 추가하여 factory 를 4번 구독해봅시다.

```swift
for _ in 0...3 {
  // 🔥 4번을 구독.
  factory.subscribe(onNext: {
    // 한 줄 출력.
    print($0, terminator: "")
  })
  .disposed(by: disposeBag)
  // 줄 바꿈.
  print()
}

/*
123
456
123
456
*/
```

## Using Traits

traits 은 일반 observables 보다 좁은 범위의 observables 입니다. 사용은 선택적입니다. 목적은 코드를 읽는 API 를 소비하는 사람들에게 명확하게 전달할 수 있는 방법을 제공하는 것입니다. trait 를 사용하여 암시하는 context 는 코드를 보다 직관적으로 만드는데 도움이 될 수 있습니다.

`Single`, `Maybe` 와 `Completable` 세 가지가 있습니다. 

### 1) Single

`Single` 은 `success(value)` 또는 `error(error)` 이벤트를 방출합니다. `success(value)` 는 실제로 `next` 와 `completed` 이벤트의 조합입니다.

이는 데이터를 다운로드하거나 디스크에서 로등할 때와 같이 성공하여 값을 생성하거나 실패하는 일회성 프로세스에 유용합니다.

### 2) Completable

`Completable` 은 `completed` 또는 error(error) 이벤트만 방출합니다. 값은 방출하지 않습니다. 파일 쓰기와 같이 작업이 성공적으로 완료되었는지 실패했는지 관심이 있는 경우 사용할 수 있습니다.

### 3) Maybe

`Maybe` 는 `Single` 과 `Completable` 의 매쉬업입니다. `success(value)`, `completed` 또는 `error(error)` 방출할 수 있습니다. 만약 성공하거나 실패할 수 있는 작업을 구현하고 선택적으로 성공시 값을 반환해야하는 경우에 사용할 수 있습니다.

```swift
let disposeBag = DisposeBag()

// 🔥 디스크에서 데이터를 읽을 때 발생할 수 있는 오류들을 모델링.
enum FileReadError: Error {
  case fileNotFound, unreadable, encodingFailed
}

// 🔥 Single 을 반환하는 디스크에서 텍스트를 로드하는 함수 구현.
func loadText(from name: String) -> Single<String> {
  return Single.create { single in

  let disposable = Disposables.create()

  // 🔥 경로 혹은 파일을 찾을 수 없는 경우 error를 Single에 추가하고 disposable 반환.
  guard let path = Bundle.main.path(forResource: name, ofType: "txt") else {
    single(.error(FileReadError.fileNotFound))
    return disposable
  }

  // 🔥 경로의 파일에서 데이터를 가져오거나 읽을 수 없는 error를 Single에 추가하고 disposable 반환.
  guard let data = FileManager.default.contents(atPath: path) else {
    single(.error(FileReadError.unreadable))
    return disposable
  }

  // 🔥 데이터를 문자열로 변환하고, 인코딩 실패 error를 Single에 추가하고 diposable 반환.
  guard let contents = String(data: data, encoding: .utf8) else {
    single(.error(FileReadError.encodingFailed))
    return disposable
  }
  // 🔥 여기까지 다다랐다면 Single에 contents를 성공적으로 추가하고 disposable 반환.
  single(.success(contents))
  return disposable
  }
}
```

이제 아래의 코드를 붙여서 작동하게 할 수 있습니다.

```swift
loadText(from: "Copyright")
// 2 Single 구독.
  .subscribe {
    // 3
    switch $0 {
    case .success(let string):
      print(string)
    case .error(let error):
      print(error)
    }
  }
  .disposed(by: disposeBag)
```

디스크의 파일 텍스트가 출력되게 된다. 파일 이름을 다른 것으로 변경하게 되면 파일을 찾을 수 없다는 에러가 대신 출력되게 됩니다.

## do & debug

### do

observable 의 방출된 이벤트에 영향을 미치지 않는 부수작업을 수행하려는 경우에 유용한 다른 연산자가 있습니다. 

`**do` 연산자를 사용하면 side effects 를 삽입할 수 있습니다. 즉, 방출된 이벤트를 어떤식으로든 변경하지 않는 작업을 수행하는 핸들러입니다.** 대신 `do` 는 이벤트를 `next` 연산자의 체인으로 전달합니다. subscribe 과 달리 `do` 는 `onSubscribe` 핸들러도 포함합니다.(`onSubscribe` 핸들러는 구독하기 전에 호출할 액션입니다.)

기존의 `never` 예제 코드에 사용해보겠습니다. `never` 는 `subscribe()` 가 실행되지만 `completed` 이벤트를 방출하지 않아서 동작을 했는지 조차 모르는데 `do` 를 사용하면 알 수 있습니다.

```swift
let observable = Observable<Any>.never()
    
let disposeBag = DisposeBag()
    
observable.do(
    onSubscribe: { print("Subscribe")}
).subscribe(
  onNext: { (element) in
    print(element)
  },
  onCompleted: {
    print("Completed")
  })
.disposed(by: disposeBag)

/*
Subscribe
*/
```

### debug

side effects 를 수행하는 것은 Rx 코드를 디버그하는 한 가지 방법입니다. 하지만 더 나은 기능이 있습니다. `debug` 연산자는 observable 에 대한 모든 정보를 출력합니다.

몇 가지 옵셔널 매개변수가 있지만 가장 유용한 것은 출력되는 식별자 문자열을 포함하는 것입니다. 여러 위치에 `debug` 호출을 추가할 수 있는 Rx 체인에서 이는 각 출력물의 소스를 구별하는데 실제로 도움이 될 수 있습니다.

- 콘솔창 출력 형식: `날짜 시간: \(작성한문자열) -> <subscribed/isDisposed>`

```swift
let observable = Observable<Any>.never()
let disposeBag = DisposeBag()

observable.debug("debug")
          .subscribe()
          .disposed(by: disposeBag)

/*
2022-07-11 01:43:01.445: debug -> subscribed
2022-07-11 01:43:01.449: debug -> isDisposed
*/
```
