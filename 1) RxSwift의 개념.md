### 내용

- RxSwift 가 무엇인지 알아봅시다.
- 왜 사용하는지 알아봅시다.

[https://github.com/ReactiveX/RxSwift](https://github.com/ReactiveX/RxSwift)

[RxSwift: Reactive Programming with Swift, Chapter 1: Hello, RxSwift!](https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/1-hello-rxswift)

***아래의 내용은 RxSwift 의 깃허브와 raywenderlich 의 글을 정리한 내용입니다.***

<div align = center>

<img src="https://user-images.githubusercontent.com/69136340/176526989-bbb50c5a-bf0e-4169-9e4d-3aecac3d1540.png" width ="200">

</div>

Rx 는 Observable<Element> 인터페이스를 통해 표현된 계산의 generic abstraction 으로, 이를 통해 observable 스트림에서 값 및 이벤트를 braodcast 하고 subscirbe 할 수 있습니다.

RxSiwft 는 Reactive Extensions 의 Swift 전용 구현입니다.

다른 Rx 구현과 마찬가지로 RxSwift 의 의도는 Observable 객체의 형태로 비동기 작업과 데이터 스트림을 쉽게 구성하고, 이러한 비동기 작업을 변환하고 구성하는 메서드 모음입니다.

`KVO observation`, `asnyc operations`, `UI Events` 와 기타 데이터 스트림은 모두 abstraction of sequence 아래 통합됩니다. 이것이 Rx 가 매우 단순하고, 우아하며 강력한 이유입니다.

## 비동기 프로그래밍의 소개

iOS 앱에서 다음과 같이 일어나는 작업들을 상상해볼까요?

- 버튼을 눌렀을 때 반응하는 것.
- 텍스트 필드가 포커스를 잃었을 때 키보드가 사라지는 것.
- 대용량의 사진을 다운로드 하는 것.
- 데이터를 디스크에 저장하는 것.
- 오디오를 재생하는 것.

이 모든 일들이 동시에 일어나는 것처러 보입니다. 키보드가 사라질 때 오디오는 일시 중지 되지 않습니다. 그렇죠?
<img src="https://user-images.githubusercontent.com/69136340/176527351-18136a3a-fdaa-423e-9717-74e3ce926f07.png" width ="600">

iOS 는 다양한 실행 context 에 걸쳐 서로 다른 스레드에서 서로 다른 작업을 수행하고, CPU 의 다른 코어에서 수행할 수 있는 다양한 종류의 API 를 제공합니다.

## Cocoa and UIKit asynchronous APIs

iOS 에서 제공하는 비동기 처리를 위한 API 들은 다음과 같습니다.

- NotificationCenter
- Delegate Pattern
- GCD
- Closures
- Combine
- async/await

사용하기로 한 API 에 따라 앱을 일관된 상태로 유지하는 어려움이 크게 달라집니다. 예를 들어, delegate pattern 이나 notification center 와 같은 오래된 API 는 앱의 상태를 일관되게 유지하기 위해서는 많은 노력을 기울여야 합니다.

동기 코드와 비동기 코드의 두 가지 코드를 비교해봅시다.

## Synchronous code

아래의 코드는 두 가지를 보장하기 때문에 매우 간단하지만 견고한 로직입니다. 동기적(**synchronously**)으로 실행되며 컬렉션을 반복하는 동안 컬렉션은 변경할 수 없습니다(**immutable**).

```swift
var array = [1, 2, 3]
for number in array {
  print(number)
  array = [4, 5, 6]
}
print(array)

/* 결과
1
2
3
[4, 5, 6]
*/
```

for 문 내에서 array 를 변경할 수 있나요? 루프가 반복되는 컬렉션이 변경되나요? 실행순서는 어떻게 되나요? 필요한 경우 number 을 수정할 수 있나요?

위의 질문에 알아보기 위해서 코드를 조금 수정해보겠습니다.

```swift
import Foundation

var array = [1, 2, 3]
for number in array {
    // number = 1  #Cannot assign to value: 'number' is a 'let' constant
    print(number)
    array = [4, 5, 6]
    print(array)
}
print(array)

/* 결과
1
[4, 5, 6]
2
[4, 5, 6]
3
[4, 5, 6]
[4, 5, 6]
*/
```

## Asynchronous code

유사한 코드를 고려하되, 각 반복이 버튼 탭에 대한 반응으로 발생한다고 가정하겠습니다. 사용자가 버튼을 반복적으로 탭하면 앱은 array 의 다음 요소를 인쇄합니다.

```swift
var array = [1, 2, 3]
var currentIndex = 0

// This method is connected in Interface Builder to a button
@IBAction private func printNext() {
  print(array[currentIndex])

  if currentIndex != array.count - 1 {
    currentIndex += 1
  }
}
```

사용자가 버튼을 탭하면 배열의 모든 요소가 출력될까요? 확신할 수 없을거에요. 다른 비동기 코드가 인쇄되기 전에 마지막 요소를 제거할 수 있습니다. 또는 다른 코드는 컬렉션 시작 부분에 새 요소를 삽입할 수 있습니다.

또한, currentIndex 가 printNext 메서드에 의해서만 변경된다고 가정하지만, 다른 코드 조각도 currentIndex 수정하는 경우가 있을 수 있습니다. 비동기 코드 작성의 핵심 문제 중 일부에 대해서 깨달았을 것입니다.

- 작업이 수행되는 순서
- 변경 가능한 데이터를 공유

## RxSwift 는 다음의 문제를 해결하고자 합니다.

### ****1. State, and specifically, shared mutable state****

(상태, 특히 공유된 변경 가능한 상태)

state 는 정의하기 어렵습니다.

예를 들어 당신이 랩톱을 시작하면 정상적으로 실행되지만 며칠 또는 몇주 동안 사용한 후에는 이상하게 작동하거나 갑자기 멈출 수 있습니다. **하드웨어와 소프트웨어는 동일하게 유지되지만 변경된 것은 state 입니다.** 당신이 다시 시작하자마자 동일한 하드웨어와 소프트웨어가 제대로 작동합니다.

또한, 메모리의 데이터, 디스크에 저장된 데이터, 사용자의 입력에 대한 반응의아티팩트, 클라우드 서비스에서 데이터를 가져온 후 남아있는 모든 추적들의 합이 랩탑의 state 입니다.

여러 비동기 구성 요소 간에 공유될 때 앱의 상태를 관리하는 것은 RxSwift 를 통해서 배울 이슈 중 하나 입니다.

### ****2. Imperative programming****

(명령형 프로그래밍)

명령형 프로그래밍은 명령문을 사용하여 프로그램의 상태를 변경하는 프로그래밍의 패러다임입니다. 명령형 코드를 사용하여 작업을 수행하는 시기와 방법을 앱에 정확히 알려줍니다.

문제는 인간이 복잡한 비동기식 앱을 위한 명령형 코드를 작성하는 것이 어렵다는 것입니다. 특히, shated mutable state 가 관련되어 있을 때 그렇습니다.

```swift
override func viewDidAppear(_ animated: Bool) {
  super.viewDidAppear(animated)

  setupUI()
  connectUIControls()
  createDataSource()
  listenForChanges()
}
```

위의 코드는 순서를 바꾸면 전혀 다른 메소드가 될 수 있습니다. 즉, 비동기 코드를 작성하는 것은 어렵습니다.

### ****3. Side effects****

(부작용)

사이드 이펙트는 코드의 현재 범위를 벗어난 상태의 모든 변경 사항을 나타냅니다. 예를 들어, 위에 예시에서 코드의 마지막 부분을 고려해봅시다.

`connectUIControls()` 는 아마도 일부 UI 컴포넌트에 일종의 이벤트 핸들러를 연결할 것입니다. 이로 인해 뷰의 상태가 변경되므로 부작용이 발생합니다. 앱이 `connectUIControls()` 를 실행하기 전과 후가 다르게 동작하기 때문입니다.

부작용 생성의 중요한 측면은 통제된 방식으로 수행하는 것입니다. 어떤 코드 조각이 부작용을 일으키는지, 어떤 코드가 단순히 데이터를 처리하고 출력하는지 결정할 수 있어야 합니다.

### ****4. Declarative code****

(선언적 코드)

명령형 프로그래밍에서는 마음대로 상태를 변경하지만, 함수형 프로그래밍에서는 부작용을 일으키는 코드를 최소화하는 것을 목표로 합니다. 

선언적 코드를 사용하면 동작을 정의할 수 있습니다. RxSwift 는 관련 이벤트가 있을 때마다 동작을 실행하고, 작업할 immutable 하고 격리된 데이터 조각을 제공합니다.

이렇게 하면 비동기 코드로 작업할 수 있지만 간단한 for 루프에서와 동일한 가정을 합니다. 즉, 변경할 수 없는 데이터로 작업하고 순차적의고 결정적인 방식으로 코드를 실행할 수 있습니다.

### 5. Reactive systems

(반응 시스템)

- Responsive(반응형): 최신 앱 상태를 나타내는 UI 를 항상 최신 상태로 유지한다.
- Resilent(회복력): 각 동작은 개별적으로 정의되며 유연한 오류 복구를 제공한다.
- Elastic(탄력성): 코드는 lazy pull-driven data 수집, event throttling, 리소스 공유와 같은 기능을 구현하는 다양한 작업을 처리.
- Message-driven: 컴포넌트들은 향상된 재사용성과 분리를 위해 메시지 기반 통신을 사용하여 수명주기와 클래스 구현을 분리합니다.

---
  
**참고:**

[RxSwift](https://github.com/ReactiveX/RxSwift)

[RxSwift: Reactive Programming with Swift, Chapter 1: Hello, RxSwift!](https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/1-hello-rxswift)

[[RxSwift] 1. RxSwift의 개념](https://ios-development.tistory.com/95?category=916618)

[import RxSwfit를 보고 도망치지 않는 방법 101가지](https://www.notion.so/import-RxSwfit-101-b7a10795a1b3457886cc882b1a1443a5)
