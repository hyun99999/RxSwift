### 내용

- Rx 의 3요소(Observables, Operators, Schedulers) 에 대해서 알아보자.

[RxSwift: Reactive Programming with Swift, Chapter 1: Hello, RxSwift!](https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/1-hello-rxswift)

***위의 글을 번역 및 요약한 글입니다.***

Rx 코드의 세 가지 building blocks 은 Observables, Operators, Schedulers 입니다.

## Observables

Observable<Element> 는 Rx 코드의 기초를 제공합니다. Element 타입의  generic data 의 변경 불가한 스냅샷을 “carry(전달)” 할 수 있는 이벤트 시퀀스를 비동기적으로 생성하는 기능입니다. 요약하자면, consumers 는 시간이 지남에 따라 다른 객체에서 emit(방출) events, values 를 subscribe 할 수 있습니다.

`Observable` 클래스를 사용하면 한 개 이상의 observers 가 실시간으로 이벤트에 반응하고 앱의 UI 를 업데이트하거나 새로운 데이터 및 수신 데이터를 처리 및 활용할 수 있습니다.

`Observable` 이 준수하는 `ObservableType` 프로토콜은 매우 간단합니다. `Observable` 은 세 가지 유형의 이벤트만 방출할 수 있습니다.

- A `next` event : 최신(또는 `next`) 데이터 값을 “carreis” 하는 이벤트입니다. 이것은 observers 가 값을 “receive” 하는 방법입니다. `Observable` 은 종료 이벤트가 방출될 때까지 이러한 값의 무한한 양을 방출할 수 있습니다.
- A `completed` event : 이 이벤트는 event sequence 를 성공적으로 종료합니다. `Observable` 이 라이프사이클을 성공적으로 완료했으며 추가 이벤트를 방출하지 않음을 의미합니다.
- A `error` event : `Observable` 은 오류와 함께 종료되고 추가 이벤트를 방출하지 않습니다.

시간이 지남에 따라 발생하는 비동기 이벤트에 대해 이야기 할 때 다음과 같이 타임라인에서 관찰 가능한 정수의 스트림을 시각화 할 수 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/176644009-53c0aa6f-2e83-43c5-950d-fb4d7384f709.png" width ="700">

*위의 그림처럼 **Observable<Int> 가 event 들을 emit(방출)할 수 있습니다.***

`Observable` 이 방출할 수 있는 세 가지 가능한 이벤트의 간단한 contract 는 Rx 의 모든 것입니다. 매우 보편적이기 때문에 가장 복잡한 앱 로직도 생성하는데 사용할 수 있습니다.

observable contract 는 `Observalbe` 이나 observer 의 특성에 대해 어떤 가정도 하지 않기 때문에 event sequences 를 사용하는 것이 궁극적인 decoupling 방식(메서드가 서로 관계가 없음)입니다.

delegate protocols 을 사용하거나 class 들이 서로 통신할 수 있도록 closures 를 주입할 필요가 없습니다.

<img src="https://user-images.githubusercontent.com/69136340/176644160-b09b782d-c598-4206-b60a-0f42294953e4.png" width ="700">  
  
***이벤트들을 방출하고 있는 Observable 을 subscribe 하는 observers 들이 이 이벤트의 발생을 감지하게 됩니다. 이러한 이벤트를 감지하는 시점에서 observers 들은 특정 액션을 취할 수 있습니다.***

실제 상황에 대한 아이디어를 얻으려면 두 가지 다른 종류의 observable sequences 인 `finite` 와 `infinite` 를 살펴보면 됩니다.

### ****Finite observable sequences****

iOS 앱에서 인터넷에서 파일을 다운로드하는 코드를 고려해 보겠습니다.

- 다운로드를 시작하고 들어오는 데이터를 관찰하기 시작합니다.
- 그런 다음 파일의 일부가 도착할 때 data 의 chunk 를 반복적으로 수신합니다.
- 네트워크 연결이 끊어지면 다운로드가 중지되고 연결 시간이 초과되어 오류가 발생합니다.
- 또는, 파일의 모든 데이터를 다운로드하게 되면 성공적으로 완료됩니다.

이 워크플로우는 일반적인 observable 의 수명주기를 설명합니다. 아래의 관련  코드를 확인해봅시다.

```swift
API.download(file: "http://www...")
   .subscribe(
     onNext: { data in
      // Append data to temporary file
      // 🔥 you appedn the data to a temporary file stored on disk.
     },
     onError: { error in
       // Display error to user
       // 🔥 you can display the error.localizedDescription in an alert box or otherwise handle your error.
     },
     onCompleted: {
       // Use downloaded file
       // 🔥 you can push a new view controller to display the downloaded file or anything else your app logic dictates.
     }
   )
```

`API.dwonload(file:)`는 네트워크를 통해 가져온 data chunk 로 `Data` 값을 방출하는 `Observable<Data>` 인스턴스를 반환합니다.

### ****Infinite observable sequences****

자연스럽거나 강제적으로 종료되어야 하는 파일 다운로드 또는 비슷한 동작과 달리 단순하게 무한한 시퀀스가 있습니다. 종종 UI 이벤트들은 `infinite observable sequences` 입니다.

예를 들어, 앱의 기기 방향 변경에 반응하는 코드를 생각해 봅시다.

- `NotificationCenter` 로부터 `UIDeviceOrientationDidChanged` 알림에 대한 observer 로 클래스를 추가할 수 있습니다.
- 여러분은 그런 다음 방향 변경을 처리하기 위해 method callback 을 제공해야 합니다. `UIDevice` 에서 현재 방향을 가져와서 최신 값에 따라 반응해야 합니다.

이러한 방향 변경 시퀀스에는 끝이 없습니다. 또한, 시퀀스는 거의 무한대이고 상태를 저장하므로 관찰을 시작할 때 항상 초기 값이 있습니다.
  
<img src="https://user-images.githubusercontent.com/69136340/176644453-e5acaa83-859f-4e31-871a-7d4f904c08d9.png" width ="700">

사용자가 기기를 절대로 회전하지 않는 경우도 있지만, 그렇다고 이벤트의 시퀀스가 종료되는 것은 아닙니다. 이벤트가 발생하지 않는 것을 의미합니다.

RxSwift 에서 아래와 같이 코드를 작성하여 장치 방향을 핸들링할 수 있습니다.

```swift
UIDevice.rx.orientation
  .subscribe(onNext: { current in
    switch current {
    case .landscape:
      // Re-arrange UI for landscape
    case .portrait:
      // Re-arrange UI for portrait
    }
  })
```

`UIDevice.rx.orientation` 은 `Observable<Orientation>` 을 생성하는 가상의 컨트롤 프로퍼티입니다. 여러분들은 구독하고 현재 방향에 따라서 app UI 를 업데이트 할 수 있습니다. `onError` 와 `onCompleted` 인수를 생략할 수 있습니다. 이러한 이벤트들은 해당 observable 에서 방출할 수 없기 때문입니다.
