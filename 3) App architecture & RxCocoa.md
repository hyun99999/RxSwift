### 내용

- RxSwift 를 적용하는 App architechure 에 대해서 간단하게 알아봅시다.
- RxCocoa 에 대해서 간단하게 알아봅시다.

[RxSwift: Reactive Programming with Swift, Chapter 1: Hello, RxSwift!](https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/1-hello-rxswift)

***위의 글을 번역 및 요약한 글입니다.***

## App architecture

RxSwift 는 어떤 식으로든 앱의 아키텍처를 바꾸지 않는다는 것을 언급할 가치가 있습니다. 대부분 이벤트, 비동기 데이터 시퀀스, 그리고 보편적인 통신 계약을 다룹니다.

reactive app 으로 만들기 위해 프로젝트를 처음부터 시작할 필요가 없다는 점도 중요합니다. 기존 프로젝트의 일부를 반복적으로 리팩토링하거나 앱의 새로운 기능을 빌드할 때 RxSwift 를 사용하면 됩니다.

MVC 아키텍처, MVP 또는 MVVM 또는 다른 패턴을 구현하여 Rx 로 앱을 만들 수 있습니다.

RxSwift 와 MVVM 은 특히 잘 어울립니다. 그 이유는 ViewModel 을 사용하면 View Controller 의 glue code 에서 UIKit 컨트롤에 직접 바인딩할 수 있는 `Observable` 프로퍼티를 노출할 수 있기 때문입니다. 이렇게 모델 데이터를 UI 에 바인딩하고 표현하기가 간단해집니다.

<img src="https://user-images.githubusercontent.com/69136340/176762380-2e22e7b1-d743-4225-ae69-a44d075e6986.png" width ="700">

## RxCocoa

RxSwift 는 플랫폼에 구애받지 않는 공통의 Rx 내용을 구현한 것입니다. 일부 고급 클래스를 제공하는 것 외에도 RxCocoa 는 다양한 UI 이벤트를 즉시 구독할 수 있도록 많은 UI 컴포넌트들에 reactive extensions 을 추가합니다.

예를 들어, UISwitch 의 상태 변경을 구독하기 위해서 RxCocoa 를 사용하는 것은 매우 쉽습니다.

```swift
toggleSwitch.rx.isOn
  .subscribe(onNext: { isOn in
    print(isOn ? "It's ON" : "It's OFF")
  })
```

RxCocoa 는 UISwitch 클래스에 `rx.isOn` 프로퍼티를 추가하므로 이벤트를 reactive `Observable` sequences 로 구독할 수 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/176762415-37f4dd0a-3c2e-4d28-a13a-2e40dce069e8.png" width ="700">

또한, RxCocoa `rx` 네임스페이스를 `UITextField`, `URLSession`, `UIViewController` 등에 추가하고 이 네임스페이스 아래에 자신만의 reactive extensions 정의할 수도 있습니다.

<div align = center>

  <img src="https://user-images.githubusercontent.com/69136340/176763436-06f286f9-ea8b-4d82-a281-e726aa93e83d.png" width ="500">
  
**사진 출처:** [[RxSwift] 1. RxSwift의 개념](https://ios-development.tistory.com/95?category=916618)
</div>
