### 내용

- Rx 의 3요소(Observables, Operators, Schedulers) 에 대해서 알아보자.

[RxSwift: Reactive Programming with Swift, Chapter 1: Hello, RxSwift!](https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/1-hello-rxswift)

***위의 글을 번역 및 요약한 글입니다.***

Rx 코드의 세 가지 building blocks 은 Observables, Operators, Schedulers 입니다.

## Operators

`ObservableType` 과 `Observable` 클래스의 구현에는 비동기 작업과 이벤트 조작의 개별적인 부분을 추상화하는 많은 메서드가 포함되어 있으며 함께 구성되어 보다 복잡한 로직을 구현할 수 있습니다. highly decoupled and composable 하기 때문에 이러한 메서드들을 operators 라고 합니다.

이러한 operator 들은 대부분 비동기적인 입력을 받고 side effects 없이 출력만 생성하기 때문에 퍼즐 조각처럼 쉽게 서로 맞물려서 큰 그림을 그릴 수 있습니다.

예를 들어, `(5 + 6) * 10 - 2` 를 살펴봅시다.

명확하고 결정적인 방법으로 여러분들은 `*, (), + , -` 를 사전에 정의된 순서대로 데이터들에 적용하고 출력을 가져와서 해결될 때까지 표현식을 계속 처리할 수 있습니다.

비슷한 방식으로 Rx operators 를 `Observable` 에서 방출한 이벤트에 적용하여 표현식이 최종 값으로 해결될 때까지 입력과 출력을 처리할 수 있습니다. 그런 다음 side effect 를 일으킬 수 있습니다.

아래는 Rx operators 를 사용하도록 조정된 이전의 방향 변경 관찰에 대한 코드입니다.

```swift
UIDevice.rx.orientation
  .filter { $0 != .landscape }
  .map { _ in "Portrait is the best!" }
  .subscribe(onNext: { string in
    showAlert(text: string)
  })
```

`UIDevice.rx.orientation` 이 `.landscape` 또는 `.portrait` 값을 생성할 때마다 RxSwift 는 `filter` 와 `map` 을 적용하여 방출된 데이터 조각에 적용합니다.

<div align = center>

  <img src="https://user-images.githubusercontent.com/69136340/176653117-2ff096be-4dd8-45c6-b0f9-c7100ddc7bd3.png" width ="400">

</div>

마지막에 subscribe 를 사용하면, 결과로 나오는 next 이벤트를 구독하고 String 값을 전달하고 화면에 텍스트와 함께 경고를 표시하는 메서드를 호출합니다.

operators 는 highly composable 합니다. 항상 데이터를 input 으로 사용하고, output 를 result 로 가져가므로 단일 연산자가 수행할 수 있는 것 보다 훨씬 많은 것을 달성하기 위해서 쉽게 체이닝하여 사용할 수 있습니다.
