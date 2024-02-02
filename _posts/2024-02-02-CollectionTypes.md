
---
layout: post
title: Collection Types - Array, Set, Dictionary
date: 2024-02-02
categories: Swift
tages:
  - [Swift, Collection Types, Array, Set, Dictionary]
---



# Collection Types - Array, Set, Dictionary

Array는 순서가 있는 값의 집합이고, 

Set은 순서가 없는 유일한(중복되지 않는) 값의 집합이고, 

Dictionary는 순서가 없는 key-value 형식의 집합입니다.

Collection Types은 값의 타입이 분명하기 때문에 잘못된 타입의 값을 삽입할 수 없습니다. 

이러한 타입들이 변수에 할당된다면 *mutable* 하지만, 상수에 할당된다면 *immutable* 하여 크기와 내용을 바꿀 수 없습니다. 

# Array

동일한 타입의 값을 순서가 있는 리스트에 저장합니다. 따라서 동일한 값을 저장할 수 있습니다. 

### Array의 선언

```swift
let someStrings1: Array<String>
let someStrings2: [String]
```

### 빈 Array 만들기

```swift
var someInts: [Int] = []
```

### 기본값을 가진 Array 만들기

```swift
var threeDoubles = Array(repeating: 0.0, count: 3)
// threeDoubles is of type [Double], and equals [0.0, 0.0, 0.0]
var anotherThreeDoubles = Array(repeating: 2.5, count: 3)
// anotherThreeDoubles is of type [Double], and equals [2.5, 2.5, 2.5]
```

### 두 배열을 더해서 배열 만들기

```swift
var sixDoubles = threeDoubles + anotherThreeDoubles
// sixDoubles is inferred as [Double], and equals [0.0, 0.0, 0.0, 2.5, 2.5, 2.5]
```

### Array Literal로 배열 만들기

```swift
var shoppingList: [String] = ["Eggs", "Milk"]
```

- 스위프트의 타입 추론으로 인해 초기화 시에 타입을 작성하지 않아도 됩니다.

```swift
var shoppingList = ["Eggs", "Milk"]
```

### 배열의 접근과 변경

- read-only `count` property  ⇒ 시간복잡도 O(1)

```swift
shoppingList.count
```

- Boolean `isEmpty` property ⇒ 시간복잡도 O(1)

```swift
if shoppingList.isEmpty {
    print("The shopping list is empty.")
} else {
    print("The shopping list isn't empty.")
}
```

- `append` method를 사용하여 배열의 끝에 새로운 값을 추가 ⇒ 평균 시간복잡도 O(1), 최악의 시간복잡도 O(N)

```swift
shoppingList.append("Flour")

// 여러 요소들을 추가할 수도 있음 ⇒ 평균 시간복잡도 O(M)
shoppingList.append(contentsOf: ["Chocolate Spread", "Cheese", "Butter"])

// 혹은 + operator를 사용할 수도 있음
shoppingList += ["Baking Powder"]
shoppingList += ["Chocolate Spread", "Cheese", "Butter"]
```

- 서브스크립트 문법을 사용하여 검색
    - 배열의 index는 0부터

```swift
var firstItem = shoppingList[0]
```

- 서브스크립트 문법을 사용하여 값 변경

```swift
shoppingList[0] = "Six eggs"
```

- 서브스크립트 문법을 사용할때는 타당한 인덱스를 사용해야 한다. 그렇지 않으면 runtime error가 발생한다.
    - EX) `shoppingList[shoppingList.count] = "Salt"` —> Runtime error
- range를 사용하여 값 변경
    - 변경할 값의 개수와 범위가 다르더라도 변경이 가능함

```swift
shoppingList[4...6] = ["Bananas", "Apples"]
shoppingList.replaceSubrange(4...6, with: ["Bananas", "Apples"])

// 삭제
shoppingList[0...1] = []
```

- 특정한 인덱스에 값 삽입 ⇒ 시간복잡도 O(1)

```swift
shoppingList.insert("Maple Syrup", at: 0)

// 여러 요소들을 삽입할 수도 있음
shoppingList.insert(contentsOf: ["Bananas", "Apples"], at: 2)
```

- 특정한 인덱스에 있는 값 삭제
    - 삭제된 값을 리턴함

```swift
let mapleSyrup = shoppingList.remove(at: 0)
```

- 배열의 마지막 값 삭제

```swift
let apples = shoppingList.removeLast() // ⇒ 시간복잡도 O(1)
shoppingList.remove(at: shoppingList.endIndex - 1) // ⇒ 시간복잡도 O(N)
```

- 그 외 삭제

```swift
// 범위 삭제
shoppingList.removeSubrange(0...2)

// 배열이 비어있는지 잘 확인해보고 삭제해야함 -> 그렇지 않으면 에러가 발생함 ⇒ 시간복잡도 O(N)
numsArray.removeFirst() // 맨 앞의 요소 삭제 -> 삭제 요소 리턴
numsArray.removeFirst(2) // 앞의 두 개 요소 삭제 -> 리턴 안함

// ⇒ 시간복잡도 O(1)
numsArray.removeLast()
numsArray.removeLast(2)

// 배열의 모든 요소 삭제
numsArray.removeAll()
numsArray.removeAll(keepingCapacity: true)  // ⇒ 시간복잡도 O(N)
// 저장공간을 일단은 보관해 줌 (안의 데이터만 날림) -> 메모리 공간 유지
// 메모리 공간을 유지하면 나중에 할당할때 빠르게 할당 가능
```

- 그 외
    - 특정한 값이 포함되어 있는지 아닌지 확인 ⇒ 시간복잡도 O(N)
    
    ```swift
    shoppingList.contains("Apples") // return (true || false)
    ```
    
    - 랜덤한 값 리턴 ⇒ 시간복잡도 O(1)
    
    ```swift
    shoppingList.randomElement() 
    ```
    
    - 특정한 인덱스의 두 값 교환 ⇒ 시간복잡도 O(1)
    
    ```swift
    shoppingList.swapAt(0, 1)
    ```
    
    - 배열의 첫번째 값 / 마지막 값 리턴 -> 옵셔널 (빈 배열이라면 nil 리턴 (값이 없을수도 있는 경우가 있기 때문에)) ⇒ 시간복잡도 O(1)
    
    ```swift
    shoppingList.first
    shoppingList.last
    ```
    

```swift
var numsArray = [1, 2, 3, 4, 5]

// 메모리의 주소를 의미
numsArray.startIndex // == 0
numsArray.endIndex // 배열로 저장되는 메모리 공간의 끝의 주소이기에 5가 나옴
// 따라서 endIndex를 얻기 위해서는 -1을 해줘야함
numsArray.endIndex.advanced(by: -1)

numsArray.index(before: numsArray.endIndex)
numsArray.index(1, offsetBy: 2) // 얼마만큼 떨어져 있는가

// 옵셔널 값 ⇒ 시간복잡도 O(N)
numsArray.firstIndex(of: 4) // 앞에서 부터 찾았을때 4는 몇번째인지
numsArray.lastIndex(of: 4) // 뒤에서 부터 찾았을때 4는 몇번째인지
// 사용 예시
if let index = numsArray.firstIndex(of: 3) {
    print(numsArray[index])
}

// 최솟값, 최댓값 ⇒ 시간복잡도 O(N)
numsArray.min()
numsArray.max()

// 배열을 직접 정렬 (오름차순)  ⇒ 시간복잡도 O(NlogN) Timsort
numsArray.sort()
numsArray // -> 정렬됨

// 정렬된 새로운 배열을 리턴  ⇒ 시간복잡도 O(NlogN) Timsort
numsArray.sorted()
let sortedNumsArray = numsArray.sorted()

// 역순으로 정렬 (내림차순) ⇒ 시간복잡도 O(N)
numsArray.reverse()
let reversedNumsArray = numsArray.reversed() // ⇒ 시간복잡도 O(1)

// 새로운 배열은 생성하지 않고, 배열의 메모리는 공유하면서 역순으로 열거할 수 있는 형식을 리턴
numsArray.sorted().reversed() 

// 배열을 랜덤으로 섞기 ⇒ 시간복잡도 O(N)
numsArray.shuffle()
numsArray.shuffled()

//  ⇒ 시간복잡도 O(N)
split(separator:maxSplits:omittingEmptySubsequences:)
split(maxSplits:omittingEmptySubsequences:whereSeparator:)

// 배열의 비교 -> 개별요소 비교, 저장순서도 비교
let a = ["A", "B", "C"]
let b = ["a", "b", "c"]

a == b // false
a != b // true
```

### 활용

- 특정 요소 삭제

```swift
var puppy = ["p", "u", "p", "p", "y"]

if let lastIndexOfP = puppy.lastIndex(of: "p") {
    puppy.remove(at: lastIndexOfP)
}
```

- 2차원 배열

```swift
var data = [[1, 2, 3],
            [4, 5, 6],
            [7, 8, 9]]
data[0][2]
```

### 반복문과의 결합

```swift
for item in shoppingList {
    print(item)
}
```

- 인덱스와 값이 필요할때 `enumerated()` method 사용  ⇒ 시간복잡도 O(N)

```swift
for (index, value) in shoppingList.enumerated() {
    print("Item \(index): \(value)")
}
```

```swift
// 열거된 것들을 Named 튜플 형태로 한개씩 전달
for tuple in numsArray.enumerated() {
    print(tuple)
    print("\(tuple.0) \(tuple.1)")
}
```

# Set

요소의 순서가 중요하지 않거나 요소가 확실하게 한 번만 나타날때 사용할 수 있습니다.

1. 집합의 수학적인 개념을 이용할 필요가 있을때, 
2. 검색 속도가 빨라야 할때, 
3. 배열을 사용하다가 값의 중복을 원하지 않을때

Set에 저장하기 위한 타입은 `Hashable` 해야합니다. 

해시 값은 Int이며, a = b 이면 해시값은 동일합니다. 

스위프트의 기본 타입들은 기본적으로 Hashable하며 Set의 값 혹은 딕셔너리의 key 값에 사용이 가능합니다. 

연관값이 없는 열거형 case 값 또한 기본적으로 Hashable 합니다. 

커스텀 타입을 Set의 값 타입 혹은 딕셔너리의 key 타입으로 사용하기 위해서는 스위프트 표준 라이브러리의 Hashable 프로토콜을 준수하도록 만들어야 합니다.

### Set의 선언

- Set에는 배열과 비슷하기 때문에 반드시 Set 타입인 것을 표시해줘야합니다.

```swift
let emptySet: Set<Int>
```

### 빈 Set의 초기화

```swift
var letters = Set<Character>()
// 타입이 지정되었다면 array literal을 이용하여 빈 set을 만들 수 있습니다.
letters = []
```

### Array literal을 이용하여 Set 만들기

```swift
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]
var favoriteGenres: Set = ["Rock", "Classical", "Hip hop"]
```

### Set에 접근과 변경

- read-only `count` property  ⇒ 시간복잡도 O(1)

```swift
favoriteGenres.count
```

- Boolean `isEmpty` property

```swift
if favoriteGenres.isEmpty {
    print("As far as music goes, I'm not picky.")
} else {
    print("I have particular music preferences.")
}
```

- 새로운 값 삽입
    - Set은 append 함수를 제공하지 않음

```swift
favoriteGenres.insert("Jazz")
```

- 삭제 후 삭제된 값을 리턴

```swift
if let removedGenre = favoriteGenres.remove("Rock") {
    print("\(removedGenre)? I'm over it.")
} else {
    print("I never much cared for that.")
}
```

```swift
// 삭제하기
set.remove(2) // 옵셔널 -> 삭제한 요소 리턴

// 존재하지 않는 요소 삭제 -> nil (에러 아님)

set.removeAll()
set.removeAll(keepingCapacity: true)
```

- 특정한 값을 포함하고 있는지 아닌지 확인  ⇒ 시간복잡도 O(1)

```swift
if favoriteGenres.contains("Funk") {
    print("I get up on the good foot.")
} else {
    print("It's too funky in here.")
}

// contains(where:)  ⇒ 시간복잡도 O(N)
```

- 그 외

```swift
set.randomElement()

// 업데이트(update)
set.update(with: 1) // 옵셔널 -> 새로운 요소가 추가되면 리턴 nil
```

### 반복문과의 결합

- 정렬되지 않은 컬렉션이기 때문에, 실행할 때마다 순서가 달라짐

```swift
for genre in favoriteGenres {
    print("\(genre)")
}
```

- Set은 순서가 없기 때문에 순서대로 값을 보고 싶다면 sorted() 메서드를 사용하여 배열로 값을 리턴합니다.

```swift
for genre in favoriteGenres.sorted() {
    print("\(genre)")
}
```

### Fundamental Set Operations

```swift
let oddDigits: Set = [1, 3, 5, 7, 9]
let evenDigits: Set = [0, 2, 4, 6, 8]
let singleDigitPrimeNumbers: Set = [2, 3, 5, 7]

oddDigits.union(evenDigits).sorted()
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
oddDigits.intersection(evenDigits).sorted()
// []
oddDigits.subtracting(singleDigitPrimeNumbers).sorted()
// [1, 9]
oddDigits.symmetricDifference(singleDigitPrimeNumbers).sorted()
// [1, 2, 9]
```

```swift
// 합집합
var unionSet = bSet.union(aSet)
bSet.formUnion(aSet) // 원본변경

// 교집합
var interSet = aSet.intersection(bSet)
aSet.formIntersection(bSet) // 원본변경

// 차집합
var subSet = aSet.subtracting(bSet)
aSet.subtract(bSet) // 원본변경

// 대칭차집합
var symmetricSet = aSet.symmetricDifference(bSet)
aSet.formSymmetricDifference(bSet) // 원본변경
```

### **Set Membership and Equality**

```swift
let houseAnimals: Set = ["🐶", "🐱"]
let farmAnimals: Set = ["🐮", "🐔", "🐑", "🐶", "🐱"]
let cityAnimals: Set = ["🐦", "🐭"]

houseAnimals.isSubset(of: farmAnimals) // 부분집합 여부
// true 
farmAnimals.isSuperset(of: houseAnimals) // 상위집합 여부
// true
farmAnimals.isDisjoint(with: cityAnimals) // 서로소 여부
// true
```

```swift
var aSet: Set = [1, 2, 3, 4]
var bSet: Set = [2, 5, 3]

bSet.isSubset(of: aSet) // 부분집한 여부
bSet.isStrictSubset(of: aSet) // 진부분집합 여부

aSet.isSuperset(of: bSet) // 상위집합 여부
aSet.isStrictSuperset(of: bSet) // 진상위집합 여부

aSet.isDisjoint(with: bSet) // 서로소 여부
```

# Dictionary

순서가 없는 key-value 쌍을 저장합니다. 

key는 유일하며 value의 식별자 역할을 합니다. 

딕셔너리의 key 타입은 Hashable 프로토콜로 구성되어 있습니다.

### Dictionary 선언

```swift
var words1: Dictionary<Int, String>
var words: [String: String]
```

### 빈 Dictionary 만들기

```swift
var namesOfIntegers: [Int: String] = [:]
// 타입 정보가 있을때 dictionary literal을 이용
namesOfIntegers = [:]
```

### Dictionary Literal를 이용하여 만들기

```swift
var airports: [String: String] = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]
var airports = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]
```

### 딕셔너리 관련 함수

- read-only `count` property  ⇒ 시간복잡도 O(1)

```swift
airports.count
```

- Boolean `isEmpty` property

```swift
if airports.isEmpty {
    print("The airports dictionary is empty.")
} else {
    print("The airports dictionary isn't empty.")
}
```

- 서브스크립트 문법을 사용하여 새로운 값 추가
    - 딕셔너리는 append 함수를 제공하지 않음

```swift
airports["LHR"] = "London"
```

- 서브스크립트 문법을 사용하여 키 값과 연관된 값을 수정

```swift
airports["LHR"] = "London Heathrow"
```

- 키가 이미 존재하면 값을 덮어쓰고, 존재하지 않는다면 값을 추가
    - 업데이트를 수행하면 oldValue를 리턴하는데 옵셔널 값이며, nil이 리턴될 수도 있음
    - 이를 통해 값의 업데이트 여부를 알 수 있습니다.

```swift
if let oldValue = airports.updateValue("Dublin Airport", forKey: "DUB") {
    print("The old value for DUB was \(oldValue).")
}
```

- 서브스크립트 문법을 사용하여 값 검색
    - 키에 대한 값이 존재하지 않을 수도 있기 때문에 옵셔널 값을 리턴
    - 값이 존재하지 않으면 nil 리턴
    - 딕셔너리는 값만 따로 검색하는 방법은 제공하지 않음

```swift
if let airportName = airports["DUB"] {
    print("The name of the airport is \(airportName).")
} else {
    print("That airport isn't in the airports dictionary.")
}
```

```swift
// 키값에 해당하는 요소가 없으면 기본값을 리턴하는 문법 -> nil이 발생할 확률이 없음
airports["DUB", default: "Empty"] 
```

- 삭제  ⇒ 시간복잡도 O(N)
    - 키에 대한 값을 nil로 설정함으로써 삭제 가능
    - 존재하지 않는 키/값을 삭제 -> 아무일도 일어나지 않음 (에러 아님)

```swift
airports["APL"] = nil
```

- 값이 존재한다면 삭제하면서 삭제된 값 리턴, 값이 존재하지 않는다면 nil 리턴

```swift
if let removedValue = airports.removeValue(forKey: "DUB") {
    print("The removed airport's name is \(removedValue).")
} else {
    print("The airports dictionary doesn't contain a value for DUB.")
}
```

- 전체 삭제

```swift
dic.removeAll()
dic.removeAll(keepingCapacity: true)
```

- 그 외

```swift
dic.randomElement() // Named Tuple 형태로 리턴 (옵셔널)

// 딕셔너리 비교
// 원래 순서가 없기 때문에, 순서는 상관없음
// 순서 상관없이 비교하면 무조건 true가 나옴 - Hashable하기 때문에, 순서 상관없이 비교 가능

// 활용
var dic1 = [String: [String]]()
dic1["arr1"] = ["A", "B"]

var dic2 = [String: [String: Int]]()
```

```swift
//  ⇒ 시간복잡도 O(N)
dic.contains(where: <#T##((key: String, value: String)) throws -> Bool#>)
```

### 반복문과의 결합

- 딕셔너리의 각각의 아이템은 (key, value) 튜플을 리턴

```swift
for (airportCode, airportName) in airports {
    print("\(airportCode): \(airportName)")
}
```

```swift
for airportCode in airports.keys {
    print("Airport code: \(airportCode)")
}

for airportName in airports.values {
    print("Airport name: \(airportName)")
}
```

- 딕셔너리의 key 값들의 집합 혹은 value 값들의 집합을 사용할 필요가 있다면 Array 인스턴스를 사용할 수 있습니다.

```swift
let airportCodes = [String](airports.keys)
// airportCodes is ["LHR", "YYZ"]

let airportNames = [String](airports.values)
// airportNames is ["London Heathrow", "Toronto Pearson"]
```

- 딕셔너리는 순서가 없기 때문에 특정 순서로 정렬하려면 키 또는 값에 `sorted()` 메서드를 사용할 수 있습니다.

```swift
dic.keys.sorted()
dic.keys.sorted()

for key in dic.keys.sorted() {
    print(key)
}
```
