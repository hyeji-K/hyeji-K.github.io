---
layout: post
# title: Swift Concurrency 공식문서 읽어보기
date: 2025-02-01
categories: geultto
tags: [geultto, refactoring]
---



- 비동기(Asynchronous) 코드는 일시 중단되고 나중에 재개될 수 있지만 한 번에 프로그램의 한 부분만 실행됨
- 프로그램에서 코드를 일시 중단하고 다시 시작하면 UI 업데이트와 같은 단기 작업을 계속 진행하면서 네트워크를 통해 데이터를 가져오거나 파일을 구문 분석하는 것과 같은 장기 실행 작업을 계속할 수 있음
- 병렬(Parallel) 코드는 여러 코드 조각이 동시에 실행됨을 의미함. 예를 들어 4코어 프로세서가 있는 컴퓨터는 각 코어가 하나의 작업을 수행하는 4개의 코드를 동시에 실행할 수 있음
- 병렬 및 비동기 코드를 사용하는 프로그램은 한 번에 여러 작업을 수행함
- 외부 시스템을 기다리는 작업을 일시 중단하고 이 코드를 메모리 안전한 방식으로 더 쉽게 작성할 수 있도록 함
- 병렬 또는 비동기 코드의 추가적인 스케줄링 유연성은 증가된 복잡성 비용과 함께 제공됨
- Swift를 사용하면 일부 컴파일 시간 검사를 가능하게 하는 방식으로 의도를 표현할 수 있음. 예를 들어 actor를 사용하여 변경 가능한 상태에 안전하게 액세스할 수 있음
- 느리거나 버그가 있는 코드에 동시성을 추가한다고 해서 코드가 빠르고 정확해진다는 보장은 없음. 사실, 동시성을 추가하면 코드를 디버그하기 더 어렵게 만들 수도 있음
- 그러나 동시성이 필요한 코드에서 동시성을 사용하면 Swift가 컴파일 시간에 문제를 포착하는 데 도움이 될 수 있음

## Defining and Calling Asynchronous Functions

- 비동기 함수 또는 비동기 메서드는 **실행 도중에 일시 중단될 수 있는 특수한 종류의 함수 또는 메서드**입니다.
- 함수 또는 메서드가 비동기임을 나타내려면 throw를 사용하여 throw 함수를 표시하는 방법과 유사하게 해당 선언에서 매개변수 뒤에 **`async`** 키워드를 작성합니다.
- 함수나 메서드가 값을 반환하면 반환 화살표(->) 앞에 **`async`**를 작성합니다.
    
    ```swift
    func listPhotos(inGallery name: String) **async** -> [String] {
        let result = // ... some asynchronous networking code ...
        return result
    }
    ```
    
    - asynchronous 및 throws 기능을 모두 수행하는 함수 또는 메서드의 경우 throws 전에 **`async`**를 작성합니다.
- 비동기 메서드를 호출하면 해당 메서드가 반환될 때까지 실행이 일시 중단됩니다.
- **가능한 정지 지점을 표시하기 위해 호출 앞에 `await`를 작성**합니다
    - 오류가 있을 수 있다고 표시하는 throw 함수를 호출할 때 try를 작성하는 것과 동일한 원리입니다.
    - 비동기 메서드 안에서 또 다른 비동기 메소드가 호출되었을 때, 실행 흐름은 중단되어집니다. (중단은 절대 암시적이거나 선점적이지 않음) 즉, 모든 가능한 중단 지점은 **`await`**로 표시되어집니다.
        - 선점적이다는 말은 어떤 스레드가 실행하고 있다면 완료될 때까지 다른 스레드가 실행되지 않는 것을 말합니다.
    
    ```swift
    let photoNames = **await** listPhotos(inGallery: "Summer Vacation")
    let sortedNames = photoNames.sorted()
    let name = sortedNames[0]
    let photo = **await** downloadPhoto(named: name)
    show(photo)
    ```
    
    - `listPhotos(inGallery:)` 와 `downloadPhoto(named:)` 메서드는 네트워크 요청을 필요로 하고, 상대적으로 오랜 시간이 걸림
    - **`async`** 를 사용하여 두 메서드 모두 비동기로 만들면 사진이 준비 될 때까지 기다리는 동안 앱의 나머지 코드가 계속 실행될 수 있습니다.
- **`await`** 키워드로 표시된 코드에서 가능한 중단 지점은 비동기 함수 혹은 메서드가 반환 값을 기다리는 동안 현재 코드가 실행을 중단할 수 있음을 나타냅니다.
- 이것은 ***yielding the thread(스레드 양보)*** 라고 불립니다. 왜냐하면 화면 뒤에서 스위프트는 현재 스레드 위의 코드의 실행을 중단하고 대신에 스레드 위에 다른 코드를 실행하기 때문입니다.
- **`await`**를 가진 코드는 실행을 중단할 수 있어야 하므로 특정 위치에서만 비동기 함수 혹은 메서드를 부를 수 있습니다.
    - 비동기 메서드 또는 프로퍼티에 있는 코드
    - @main으로 표시된 구조체, 클래스, 열거형의 static main() 메서드에 있는 코드
    - 이번 글의 이후에 나올 Unstructed Concurrency에서 보게 될 하위 작업의 코드
    
    <aside>
    💡 [`Task.sleep(nanoseconds:)`](https://developer.apple.com/documentation/swift/task/3862701-sleep) 메서드는 동시성 작동방식을 배우기 위해 간단한 코드를 작성할 때 유용합니다. 이 메서드는 아무 작업도 수행하지 않고 그냥 시간을 기다리는 메서드입니다. 네트워크 작업을 시뮬레이션하는 등의 실험을 해보고 싶을 때 이 메서드를 사용하면 됩니다.
    
    ```swift
    func listPhotos(inGallery name: String) async throws -> [String] {
        try await Task.sleep(nanoseconds: 2 * 1_000_000_000)  // Two seconds
        return ["IMG001", "IMG99", "IMG0404"]
    }
    ```
    
    </aside>
    

## **Asynchronous Sequences**

- 이전 섹션에서 본 `listPhotos(inGallery:)` 함수는 배열의 모든 요소가 준비가 된 후에 비동기적으로 전체 배열을 한 번에 반환합니다.
- 또 다른 접근 방식은 **Asynchronous Sequences(비동기 시퀀스)**를 사용하여 한 번에 컬렉션의 한 요소를 기다리는 것입니다.
    
    ```swift
    import Foundation
    
    let handle = FileHandle.standardInput
    **for** try **await** line **in** handle.bytes.lines {
        print(line)
    }
    ```
    
- for-in 루프를 사용하는 대신에 위의 예는 **`for`** 뒤에 **`await`**를 사용합니다.
- 비동기 함수 혹은 메소드를 호출할 때처럼 **`await`**를 작성하는 것은 **중단 지점**을 나타냅니다.
- **`for-await-in`** 루프는 각각의 반복에서 다음 요소를 사용할 수 있을 때까지 실행을 정지하고 기다리게 됩니다.
- Sequence 프로토콜에 적합성을 추가하여 for-in 루프에서 고유한 유형을 사용할 수 있는 것과 같은 방식으로, AsyncSequence 프로토콜에 적합성을 추가하여 for-await-in 루프에서 고유한 유형을 사용할 수 있습니다.

## **Calling Asynchronous Functions in Parallel**

- **`await`**를 사용하여 비동기 함수를 호출하면 **한 번에 한 코드만 실행**됩니다.
- 비동기 코드가 실행되는 동안 호출자는 다음 줄의 코드를 실행하려고 이동하기 전에 해당 코드가 완료되는 것을 기다리게 됩니다.
- 예를 들어, 갤러리에서 처음 세 개의 사진을 가지고 오려면 `downloadPhoto(named:)`에 대한 **`await`**를 세 번 불러옵니다.
    
    ```swift
    let firstPhoto = **await** downloadPhoto(named: photoNames[0])
    let secondPhoto = **await** downloadPhoto(named: photoNames[1])
    let thirdPhoto = **await** downloadPhoto(named: photoNames[2])
    
    let photos = [firstPhoto, secondPhoto, thirdPhoto]
    show(photos)
    ```
    
- 이러한 접근 방식은 중요한 약점은, 비록 다운로드가 비동기적이고 그것이 처리되는 동안 다른 작업을 할 수 있을지라도 `downloadPhoto(named:)`는 **한 시점에 하나만 실행된다는 것**입니다. 각각의 사진은 다음 사진이 다운로드를 시작하기 전에 완벽히 다운로드 됩니다.
- 그러나 이러한 기다림 없이 각 사진이 독립적으로 다운로드 되거나 혹은 동시에 다운로드 되게 할 수 있습니다.
- 비동기 함수를 호출하고 주변의 코드와 병렬로 실행하기 위해서는 **상수를 선언하기 전 `let` 키워드 앞에 `async`를 작성하고 상수를 사용할 때마다 `await`를 작성**하면 됩니다.
    
    ```swift
    **async** let firstPhoto = downloadPhoto(named: photoNames[0])
    **async** let secondPhoto = downloadPhoto(named: photoNames[1])
    **async** let thirdPhoto = downloadPhoto(named: photoNames[2])
    
    let photos = **await** [firstPhoto, secondPhoto, thirdPhoto]
    show(photos)
    ```
    
    - `downloadPhoto(named:)`를 세 번 호출하게 되는데, 모든 호출이 다른 호출의 완료를 기다리지 않고 실행되게 됩니다. 만약 이용가능한 충분한 시스템 자원이 있다면, 그들은 동시에 실행할 수 있습니다.
    - 코드가 함수의 결과를 기다리기 위해서 중단을 할 필요가 없기 때문에 await를 표시하지 않아도 됩니다.
    - 대신에, `photos`가 정의된 곳까지 실행은 계속됩니다. 그 시점에서 사진들이 모두 다운이 되어 있어야 하므로 **`await`**을 작성합니다.
- 여기에서 이 두 접근 방식의 차이점에 대해서 생각할 수 있습니다.
    - **다음 줄의 코드가 함수의 결과에 의존적**일 때 **`await`**를 사용하여 비동기 함수를 호출합니다. 이는 **순차적(sequentially)**으로 수행되는 작업을 생성합니다.
    - **코드의 결과가 필요하지 않은 경우**에 **`async-let`**을 사용하여 비동기 함수를 호출합니다. 이렇게 하면 **병렬**로 수행할 수 있는 작업이 생성됩니다.
    - **`await`**와 **`async-let`**은 모두 중단되는 동안 다른 코드를 실행할 수 있습니다.
    - 이 두 경우 모두 비동기 함수가 반환될 때까지 필요한 경우 중단을 해야하므로 해당 지점을 **`await`**로 표시합니다.
- 동일한 코드에서 두 가지 접근 방식을 모두 사용할 수도 있습니다.

## **Tasks and Task Groups**

- Task는 프로그램의 일부로 **비동기적으로 실행할 수 있는 작업의 단위**입니다.
- 모든 비동기적 코드는 task의 일부로써 실행되는데, 이전 섹션에서 본 **`async-let`** 구문은 자식 작업을 생성합니다.
- 또한 작업 그룹을 만들고 해당 그룹에 자식 작업을 추가할 수 있습니다. 이를 통해 우선 순위 및 취소를 더 잘 제어할 수 있으며 동적인 수의 작업을 생성할 수 있습니다.
- 작업은 **계층 구조로 정렬**됩니다. 작업 그룹의 각 작업에는 동일한 부모 작업을 가지고 있고 각 작업에는 자식 작업을 가질 수 있습니다.
- 작업과 작업 그룹 간의 명시적 관계 때문에 이런 접근 방식을 ***structured concurrency**(***구조적 동시성)**이라고 합니다.
- 정확성에 대한 책임 중 일부는 사용자가 지기는 하지만 작업 간의 명시적 부모-자식 관계를 통해 Swift는 취소 전파와 같은 일부 동작을 처리할 수 있고, 컴파일 시간에 일부 오류를 감지할 수 있습니다.
    
    ```swift
    await withTaskGroup(of: Data.self) { **taskGroup** in
        let photoNames = await listPhotos(inGallery: "Summer Vacation")
        for name in photoNames {
            **taskGroup**.addTask { await downloadPhoto(named: name) }
        }
    }
    ```
    
    - 작업 그룹에 대한 더 많은 정보 → [TaskGroup](https://developer.apple.com/documentation/swift/taskgroup)

### **Unstructured Concurrency**

- 이전 섹션에서 본 동시성에 대한 구조적 접근 방식 외에도 Swift는 **구조화되지 않은 동시성**도 지원합니다.
- 작업 그룹의 일부인 작업과 달리 구조화되지 않은 작업에는 부모 작업을 가지지 않습니다.
- 프로그램이 필요로 하는 방식으로 구조화되지 않은 작업을 관리할 수 있는 완전한 유연성이 있지만, 정확성에 대한 완전한 책임도 있습니다.
- 현재 `actor`에서 실행되는 비구조적 작업을 생성하려면 [`Task.init(priority:operation:)`](https://developer.apple.com/documentation/swift/task/3856790-init)
생성자를 호출하면 됩니다.
- 현재 `actor`의 일부가 아닌 비구조적 작업, 보다 구체적으로 분리된 작업을 생성하려면  [`Task.detached(priority:operation:)`](https://developer.apple.com/documentation/swift/task/3856786-detached) 클래스 메소드를 호출합니다.
- 이 두 작업 모두 작업과 상호작용할 수 있는 task handle을 반환합니다. 예를 들어 이러한 작업의 결과를 기다리거나 취소할 수 있습니다.
    
    ```swift
    let newPhoto = // ... some photo data ...
    let handle = Task {
        return await add(newPhoto, toGalleryNamed: "Spring Adventures")
    }
    let result = await handle.value
    ```
    
    - → [Task](https://developer.apple.com/documentation/swift/task)

### **Task Cancellation**

- 스위프트 동시성은 **cooperative cancellation model(협력 취소 모델)**을 사용합니다.
- 각 작업은 그것이 실행의 적절한 지점에서 취소되었는지에 대한 여부를 확인하고 적절한 방법으로 취소를 처리합니다.
- 수행 중인 작업에 따라 일반적으로 다음 중 하나를 의미합니다.
    - Throwing an error like `CancellationError`
    - Returning `nil` or an empty collection
    - Returning the partially completed work
- 취소를 확인하려면 작업이 취소된 경우 `CancellationError`를 발생시키는  [`Task.checkCancellation()`](https://developer.apple.com/documentation/swift/task/3814826-checkcancellation)을 호출하거나 [`Task.isCancelled`](https://developer.apple.com/documentation/swift/task/3814832-iscancelled)의 값을 확인하고 자체 코드에서 취소를 처리합니다.
- 예를 들어, 갤러리에서 다운로드 중인 사진에 대한 작업은 부분 다운로드를 삭제하고 네트워크 연결을 닫아야 할 수 있습니다.
- 이러한 취소를 수동으로 전파하려면 [`Task.cancel()`](https://developer.apple.com/documentation/swift/task/3851218-cancel) 을 호출하면 됩니다.

## **Actors**

- 클래스처럼 **`actor`**도 **참조 타입**입니다.
- 클래스와 달리 **`actor`**는 **한 번에 하나의 작업만 변경 가능한 상태에 접근할 수 있도록 허용**하므로 여러 작업의 코드가 **`actor`**의 동일한 인스턴스와 상호작용하는 것이 안전하게 됩니다.
- **`actor`** 키워드를 사용하여 정의합니다.
    
    ```swift
    actor 이름 {
        // 상세 코드
    }
    ```
    
    ```swift
    **actor** TemperatureLogger {
        let label: String
        var measurements: [Int]
        private(set) var max: Int
    
        init(label: String, measurement: Int) {
            self.label = label
            self.measurements = [measurement]
            self.max = measurement
        }
    }
    ```
    
- 구조체 및 클래스와 동일한 이니셜라이저 구문을 사용하여 **`actor`**의 인스턴스를 생성할 수 있습니다.
- **`actor`**의 프로퍼티나 메소드에 접근할 때 **`await`**를 사용하여 잠재적인 중단 지점을 표시합니다.
    
    ```swift
    let logger = TemperatureLogger(label: "Outdoors", measurement: 25)
    print(**await** logger.max)
    // Prints "25"
    ```
    
- **`actor`**는 한 번에 하나의 작업만 변경 가능한 상태에 접근할 수 있도록 허용하기 때문에 다른 작업 코드가 이미 logger에 접근 중인 경우에 이 코드는 프로퍼티 접근을 기다리는 동안 중단됩니다.
- 대조적으로 actor의 코드에서는 actor의 프로퍼티에 접근할 때 await를 작성하지 않습니다.
    
    ```swift
    extension TemperatureLogger {
        func update(with measurement: Int) {
            measurements.append(measurement)
            if measurement > max {
                max = measurement
            }
        }
    }
    ```
    
- `update(with:)` 메소드는 이미 actor에서 실행 중이므로 max와 같은 프로퍼티에 대한 접근을 await를 표시하지 않습니다.
- 이는 actor가 변경 가능한 상태와 상호작용하기 위해 한 번에 하나의 작업만 허용하는 이유 중 하나를 보여줍니다.
- actor의 상태에 대한 일부 업데이트는 일시적으로 불변성을 깨뜨립니다.
- TemperatureLogger actor는 온도 목록과 최고 온도를 추적하고 새 측정 값을 기록할 때 최고 온도를 업데이트 합니다.
- 업데이트 도중에 새 측정 값이 추가되면 이는 TemperatureLogger의 데이터가 일시적으로 일치하지 않게 되는 문제가 발생합니다.
- 따라서 여러 작업이 동시에 동일한 인스턴스와 상호작용하는 것을 방지하면 다음 이벤트 시퀀스와 같은 문제를 방지할 수 있습니다.
    1. Your code calls the `update(with:)` method. It updates the `measurements` array first.
    2. Before your code can update `max`, code elsewhere reads the maximum value and the array of temperatures.
    3. Your code finishes its update by changing `max`.
- 위와 같은 경우에 다른 곳에서 실행 중인 코드는 부정확한 정보에 접근하게 됩니다. 데이터가 일시적으로 유효하지 않은 동안 `update(with:)` 호출 중간에 actor에 대한 접근이 끼어있기 때문입니다.
- Swift actor를 사용하면 이러한 문제를 방지할 수 있습니다. actor는 한 시점에 하나의 작업만 허용하고 해당 코드는 await로 정지 가능한 지점을 표시하기 때문입니다.
- `update(with:)`는 중단 지점을 포함하지 않기 때문에 다른 코드는 업데이트 도중 데이터에 접근할 수 없습니다.
- 클래스의 인스턴스에서와 같이 actor 외부에서 이러한 프로퍼티에 접근하려고 하면 컴파일 오류가 발생합니다.
    
    ```swift
    print(logger.max)  // Error
    ```
    
- 위의 컴파일 오류의 이유는 actor의 프로퍼티는 actor의 독립 상태의 일부이기 때문입니다.
- 스위프트는 actor 내부의 코드만 actor의 로컬 상태에 접근할 수 있도록 보장합니다.
- 이러한 보장을 **actor isolation**이라고 부릅니다.