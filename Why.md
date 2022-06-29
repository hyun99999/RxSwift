[RxSwift/Why.md at main · ReactiveX/RxSwift](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/Why.md)

***GitHub - RxSwift document 를 번역 및 정리한 내용입니다.***

# Why

Rx 를 사용하면 declarative way(선언적 방식)으로 앱을 빌드할 수 있습니다.

## Bindings

```swift
Observable.combineLatest(firstName.rx.text, lastName.rx.text) { $0 + " " + $1 }
    .map { "Greetings, \($0)" }
    .bind(to: greetingLabel.rx.text)
```

이것은 UITableView 와 UICollectionView 에서도 작동합니다.

```swift
viewModel
    .rows
    .bind(to: resultsTableView.rx.items(cellIdentifier: "WikipediaSearchCell", cellType: WikipediaSearchCell.self)) { (_, viewModel, cell) in
        cell.title = viewModel.title
        cell.url = viewModel.url
    }
    .disposed(by: disposeBag)
```

공식적인 제안은 단순한 바인딩에서 필요하지 않더라도 `.disposed(by: disposeBag)`  을 사용하는 것입니다.

## Retries(재시도)

API 가 실패하지 않으면 좋겠지만, 불행하게도 실패합니다. 다음과 같은 API 메서드가 있다고 해보겠습니다.

```swift
func doSomethingIncredible(forWho: String) throws -> IncredibleThing
```

이 기능을 그대로 사용하면 실패할 경우 재시도하기가 정말 어렵습니다. exponential backoffs 모델리의 복잡성은 말할 것도 없습니다. 물론 가능하지만, 코드에는 당신이 신경 쓰지 않는 일시적인 상태가 많이 포함되어 있고, 이는 재사용할 수 없습니다.

이상적으로, 재시도의 본질을 파악하고, 이를 모든 작업에서 적용할 수 있기를 원할 것입니다.

이것이 Rx 로 간단한 재시도를 할 수 있는 방법입니다.

```swift
doSomethingIncredible("me")
    .retry(3)
```

사용자 지정 재시도 연산자 역시 쉽게 만들 수 있습니다.

## Delegates

지루하고 non-expressive 한 일을 하는 대신

```swift
public func scrollViewDidScroll(scrollView: UIScrollView) { [weak self] // what scroll view is this bound to?
    self?.leftPositionConstraint.constant = scrollView.contentOffset.x
}
```

다음과 같이 작성할 수 있습니다.

```swift
self.resultsTableViewㅡ
    .rx.contentOffset
    .map { $0.x }
    .bind(to: self.leftPositionConstraint.rx.constant)
```

## KVO(Key-Value-Observing)

Instead of:

```swift
`TickTock` was deallocated while key value observers were still registered with it. Observation info was leaked, and may even become mistakenly attached to some other object.
// key value observers 가 등록되어 있는 동안 'TickTock' 이 할당 해제되었다고 하겠습니다. 이때 observation info 가 유출되어서 다른 object 에 연결될 수도 있습니다.
```

and

```swift
-(void)observeValueForKeyPath:(NSString *)keyPath
                     ofObject:(id)object
                       change:(NSDictionary *)change
                      context:(void *)context
```

`rx.observe` 와 `rx.observeWeakly` 를 사용하여 다음과 같이 사용할 수 있습니다.

```swift
view.rx.observe(CGRect.self, "frame")
    .subscribe(onNext: { frame in
        print("Got new frame \(frame)")
    })
    .disposed(by: disposeBag)
```

or

```swift
someSuspiciousViewController
    .rx.observeWeakly(Bool.self, "behavingOk")
    .subscribe(onNext: { behavingOk in
        print("Cats can purr? \(behavingOk)")
    })
    .disposed(by: disposeBag)
```

## Notifications

Instead of using:

```swift
@available(iOS 4.0, *)
public func addObserverForName(name: String?, object obj: AnyObject?, queue: NSOperationQueue?, usingBlock block: (NSNotification) -> Void) -> NSObjectProtocol
```

just write

```swift
NotificationCenter.default
    .rx.notification(NSNotification.Name.UITextViewTextDidBeginEditing, object: myTextView)
    .map {  /*do something with data*/ }
    ....
```

## Transient state(일시적인 상태)

비동기 프로그램을 작성할 때 transient state(일시적인 상황) 에도 많은 문제가 있습니다. 일반적인 예는 자동 **완성 검색 상자**입니다.

Rx 없이 코드를 작성한다면 첫 번째 문제는 `abc` 의 `c` 가 입력되고 ab 에 대한 보류 중인 요청이 있을 때 이 요청이 취소된다는 것 입니다. 그리 어렵지 않습니다. 보류 중인 요청에 대한 참조를 가질 추가 변수를 생성하면 됩니다.

다음 문제는 요청이 실패하면 지저분한 재시도 로직을 수행해야 합니다. 하지만, 정리해야 하는 재시도 횟수를 캡처하는 필드가 몇개 더 있습니다.

프로그램이 서버에 대한 요청을 실행하기 전에 일정 시간 기다리면 좋을 것 같습니다. 누군가가 매우 긴 내용을 입력하는 과정을 대비하여 서버에 스팸 메일을 보내고 싶지 않습니다. 추가적인 타이머 필드가 있을까요?

검색이 실행되는 동안 화면에 표시되어야 하는 항목과 모든 재시도에도 실패할 경우에 표시되어야 하는 항목은 어떻게 해야할까요?

이 모든 것을 작성하고 테스트하는 것은 지루할 것입니다. Rx 로 동일한 논리를 작성해 보겠습니다.

```swift
searchTextField.rx.text
    .throttle(.milliseconds(300), scheduler: MainScheduler.instance)
    .distinctUntilChanged()
    .flatMapLatest { query in
        API.getSearchResults(query)
            .retry(3)
            .startWith([]) // clears results on new search term
            .catchErrorJustReturn([])
    }
    .subscribe(onNext: { results in
      // bind to ui
    })
    .disposed(by: disposeBag)
```

추가적인 플래그나 필드가 필요하지 않습니다. Rx 는 일시적인 혼란을 처리합니다.

## Compositional disposal(구성 폐기)

table view 에 흐린 이미지를 표시하려는 시나리오를 가정하겠습니다.

- 먼저, 이미지를 URL 에서 가져온 다음 디코딩한 다음 흐리게 처리해야 합니다.
- 블러 처리를 위한 비용이 존재하기 때문에 셀이 보이는 table view 영역을 벗어나면 프로세스가 취소될 수 있다면 좋을 것입니다.
- 사용자가 매우 빠르게 스와이프하면 많은 요청들이 시작되고 취소될 수 있기 때문에 셀이 보이는 영역에 들어가면 이미지를 가져오는 것을 즉시 시작하지 않아도 좋을 것입니다.
- 또한, 동시에 이미지 작업의 수를 제한할 수 있다면 좋을 것입니다.

아래는 Rx 를 사용하여 구현한 것입니다.

```swift
// this is a conceptual solution
let imageSubscription = imageURLs
    .throttle(.milliseconds(200), scheduler: MainScheduler.instance)
    .flatMapLatest { imageURL in
        API.fetchImage(imageURL)
    }
    .observeOn(operationScheduler)
    .map { imageData in
        return decodeAndBlurImage(imageData)
    }
    .observeOn(MainScheduler.instance)
    .subscribe(onNext: { blurredImage in
        imageView.image = blurredImage
    })
    .disposed(by: reuseDisposeBag)
```

이 코드는 모든 작업을 수행하며 `imageSubscription` 이 삭제되면, 종속된 모든 비동기 작업ㅇ르 취소하고 불량 이미지가 UI 에 바인딩되지 않도록 합니다.

## Aggregating network requests(네트워크 요청 집계)

두 개의 요청을 실행하고 둘 다 완료되었을 때 결과를 집계해야 하는 경우는 어떻게 해야할 까요?

`zip` operator 가 있습니다.

```swift
let userRequest: Observable<User> = API.getUser("me")
let friendsRequest: Observable<[Friend]> = API.getFriends("me")

Observable.zip(userRequest, friendsRequest) { user, friends in
    return (user, friends)
}
.subscribe(onNext: { user, friends in
    // bind them to the user interface
})
.disposed(by: disposeBag)
```

그렇다면 이러한 APIs 가 백그라운드 스레드에서 결과를 반환하고 main UI 스레드에서 바인딩이 발생해야 하는 경우에는 어떻게 할까요?

`observeOn` 이 있습니다.

```swift
let userRequest: Observable<User> = API.getUser("me")
let friendsRequest: Observable<[Friend]> = API.getFriends("me")

Observable.zip(userRequest, friendsRequest) { user, friends in
    return (user, friends)
}
.observeOn(MainScheduler.instance)
.subscribe(onNext: { user, friends in
    // bind them to the user interface
})
.disposed(by: disposeBag)
```

## State(상태)

mutation 을 허용하는 언어를 사용하면 전역 상태에 쉽게 접근할 수 있고 변경 할 수 있습니다. 공유된 전역 상태의 제어되지 않은 mutation 은 쉽게 combinatorial explosion(문제의 조합이 받는 영향에 따라 문제의 복잡성이 급격히 증가)을 일으킬 수 있습니다.

그러나 다른 한편으로는, 스마트한 방식으로 사용될 때 명령형 언어는 하드웨어에 더 가깝게 더 효율적인 코드를 작성할 수 있습니다.

combinatorial explosion 과 싸우는 일반적인 방법은 상태를 가능한 단순하게 유지하고, unidirectional data flows(단방향 데이터 흐름)을 사용하여 파생 데이터를 모델링하는 것입니다.

이것은 Rx 가 정말 빛나는 곳입니다. Rx 는 기능적 세계와 명령적 세계 사이의 sweet spot 입니다. 변경할 수 없는 정의와 순수 함수를 사용하여 안정적인 구성 가능한 방식으로 변경 가능한 상태의 스냅샷을 처리할 수 있습니다.

실제적인 예시가 있나요?

## Easy integration(쉬운 통합)

자신만의 observable 을 생성해야 한다면?

꽤나 쉽습니다. 아래의 코드는 RxCocoa 에서 가져왔으며 URLSession 으로 HTTP 요청을 래핑하는데 필요한 모든 것입니다.

```swift
extension Reactive where Base: URLSession {
    public func response(request: URLRequest) -> Observable<(Data, HTTPURLResponse)> {
        return Observable.create { observer in
            let task = self.base.dataTask(with: request) { (data, response, error) in
            
                guard let response = response, let data = data else {
                    observer.on(.error(error ?? RxCocoaURLError.unknown))
                    return
                }

                guard let httpResponse = response as? HTTPURLResponse else {
                    observer.on(.error(RxCocoaURLError.nonHTTPResponse(response: response)))
                    return
                }

                observer.on(.next(data, httpResponse))
                observer.on(.completed)
            }

            task.resume()

            return Disposables.create(with: task.cancel)
        }
    }
}
```

## Benefits(이익)

요약하면 Rx 를 사용하면 코드가 다음과 같이 만들어 집니다.

- Composable(구성 가능한) ← Rx 는 composition 의 별명이기 때문입니다.
- Reuable(재사용 가능) ← composable 하기 때문입니다.
- Declarative(선언적) ← 정의는 변경할 수 없고 오직 데이터만 변경되기 떄문입니다.
- Understandable and concise(이해하기 쉽고 간결합니다.) ← 추상화 수준을 높이고 일시적인 상태를  제거합니다.
- Stable(안정성) ← Rx 코드는 철저하게 단위 테스트를 거쳤기 때문입니다.
- Less stateful ← 애플리케이션을 단방향 데이터 흐름으로 모델링하기 때문입니다.
- Without leaks(누수 없음) ← 리소스 관리가 쉽기 때문입니다.

## It’s not all or nothing

Rx 를 사용하여 최대한 많은 애플리케이션을 모델링하는 것이 좋습니다.

그러나 모든 operator 를 알지 못하고 특정 사례의 모델링하는 연산자가 있는지 여부도 모른다면 어떻게 될까요?

Rx 연산자는 수학을 기반으로 하며 직관적이어야 합니다. 좋은 소식은 약 10-15명의 operators 가 일반적인 사용사례를 다룬다는 것입니다. map, filter, zip, observeOn 등이 있습니다.

[all Rx operators](https://reactivex.io/documentation/operators.html) 의 목록이 있습니다.

각 operators 에 대해 작동 방식을 설명하는데 도움이 되는 [marble diagram](https://reactivex.io/documentation/operators/retry.html) 이 있습니다.

하지만 해당 목록에 없는 operator 가 필요하면 어떻게 할까요? 나만의 operator 를 만들 수 있습니다.

어떤 이유로 그런 종류의 operator 를 만드는 것이 정말 어렵거나 작업해야하는 일부 레거시 상태 저장 코드가 있는 경우 어떻게 할까요? [Rx monads](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/GettingStarted.md#life-happens) 에서 쉽게 빠져나와 데이터를 처리하고 다시 돌아올 수 있습니다.
