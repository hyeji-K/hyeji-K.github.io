---
layout: post
# title: Clean Architecture: 영화 앱 구조 뜯어보며 이해하기
date: 2025-03-02
categories: geultto
tags: [geultto, refactoring]
---

그동안 미뤄 왔던 클린 아키텍처에 대해 공부하고자 로버트 C. 마틴(Robert C. Martin)의 *Clean Architecture*를 읽었다. 이 책은 소프트웨어 설계 원칙과 컴포넌트 간의 관계를 체계적으로 정리하며, 유지보수성과 확장성을 고려한 구조적 접근 방식을 강조한다. 다만 책의 내용이 머릿속에서 잘 정리가 되지 않았기 때문에, 이 글에서는 [**iOS-Clean-Architecture-MVVM**](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM)의 코드를 뜯어보면서 지피티와의 대화를 통해 스스로 이해한 내용을 정리해보고자 한다. 

## 아키텍처의 목표

로버트 마틴은 아키텍처의 목표가 "변경 용이성"에 있다고 강조한다. 좋은 소프트웨어 아키텍처는 비즈니스 요구사항이 변경되더라도 최소한의 영향으로 대응할 수 있어야 한다. 이를 위해 **관심사의 분리(Separation of Concerns)**와 **의존성 역전(Dependency Inversion)**을 기반으로 구조를 설계해야 한다.

## **클린 아키텍처(Clean Architecture)**

책에서는 **클린 아키텍처(Clean Architecture)**의 핵심 개념을 도식화한 원형 구조를 제시한다. 이 구조는 크게 네 개의 계층으로 나뉜다.

- **Entity (엔티티)**는 비즈니스 규칙을 포함하는 핵심 계층
- **Use Case (유스케이스)**는 애플리케이션의 동작을 정의하는 계층
- **Interface Adapter (인터페이스 어댑터)**는 데이터 변환 및 UI와의 연결 역할
- **Frameworks & Drivers (프레임워크 및 드라이버)**는 데이터베이스, 웹 프레임워크 등 외부 요소

이 계층 구조의 가장 중요한 특징은 "의존성 규칙(Dependency Rule)"이다. **안쪽 계층이 바깥쪽 계층에 대해 알지 못해야 하며, 의존성은 항상 안쪽으로 향해야 한다.**

## iOS-Clean-Architecture-MVVM의 코드를 보며 이해하기
## 도메인 계층(Entity와 Use Case)

```swift
struct Movie: Equatable, Identifiable {
    typealias Identifier = String
    enum Genre {
        case adventure
        case scienceFiction
    }
    let id: Identifier
    let title: String?
    let genre: Genre?
    let posterPath: String?
    let overview: String?
    let releaseDate: Date?
}
```

처음에 코드를 봤을때 MVVM의 Model과 구조가 비슷하다고 생각되어, 클린 아키텍처에서 Model을 Entity로 부르는 줄 알았다. 하지만 서버에서 받아오는 데이터와 Entity의 구조가 다르다는 것을 알게 되었고, MVVM의 Model과 Entity는 목적과 역할에 차이가 있다는 걸 알게 되었다. 

- MVVM의 Model은 네트워크 요청을 통해 불러온 데이터의 구조를 정의하고, ViewModel에서 데이터를 변환하거나 가공할 때 사용한다.
- Entity는 앱의 **핵심이 되는 업무 규칙**들이 속하는 계층으로, 비즈니스 로직을 포함할 수 있으며, UI나 네트워크, 데이터베이스 등의 세부 구현과 독립적이고, 어디서든 재사용될 수 있다. 또한 Use Case에서 비즈니스 규칙을 처리할 때 주로 사용된다.

### 🙋?  그럼 클린 아키텍처에서 Model은 없는 것일까?

Clean Architecture에서는 Model 대신 **Entity와 Data Model(DTO)로 나눠서 사용한다.**

위에서 설명했듯이 **엔티티는 비즈니스 로직을 포함할 수 있는 핵심 도메인 객체**이며, **DTO는 네트워크, 데이터베이스에서 데이터를 변환하는 역할**이다. 클린 아키텍처에서는 Entity와 Data Model을 분리하여 외부 의존성을 줄이려는 아키텍처이다. 

- 데이터 흐름 예시
    - API에서 데이터를 가져옴 (MovieDTO)
    - Repository에서 Entity로 변환 (Movie)
    - Use Case에서 비즈니스 로직 수행
    - UI에서 Entity를 활용하여 표시

### 🙋?  유스케이스(Use Case)에서 비즈니스 로직을 수행한다는데, 그럼 대체 유스케이스란 뭘까?

유스케이스는 **애플리케이션에서 수행해야 하는 특정한 동작이나 기능을 정의한 개념**으로, 사용자의 행동과 시스템이 어떻게 상호작용하는 지를 설명한다. 즉, 사용자가 제공해야 하는 입력, 사용자에게 보여줄 출력, 해당 출력을 생성하기 위한 처리 단계를 기술한다.

유스케이스는 엔티티(Entity)와 인터페이스 어댑터(Interface ****Adapter) 계층 사이에서 **비즈니스 로직을 담당**하는데, UI가 직접 데이터베이스나 네트워크 요청을 하지 않고, 유즈케이스를 통해서만 데이터를 가져오기 때문에 유지보수가 쉽다. 또한 **의존성 역전 원칙(DIP)을 적용**하여 외부 요소(UI, DB 등)와 분리한다.

### 🙋?  **의존성 역전 원칙(DIP; Dependency Inversion Principle)이란?**

**"고수준 모듈(비즈니스 로직)이 저수준 모듈(데이터, 네트워크)에 의존하지 말고, 둘 다 추상화(인터페이스)에 의존해야 한다."**는 것으로, 

- **비즈니스 로직(Use Case)은 데이터 계층(Repository의 구현체)와 직접적으로 연결되지 않아야 한다.**
- **Repository는 인터페이스를 사용하여 추상화해야 한다.**
- **Use Case는 인터페이스에 의존하고, 실제 구현체(네트워크, 캐시)는 바뀌어도 영향이 없어야 한다.**

### 🙋?  그럼 여기서 ‘의존한다’라는 건 무엇일까?

**의존이란 한 객체가 다른 객체를 필요로 하여, 해당 객체가 변경될 때 함께 영향을 받는 관계**다.

- 의존의 대표적인 방식
    - 변수(속성)로 다른 객체를 참조하는 경우 (`let repository: MoviesRepository`)
    - 함수의 매개변수로 객체를 받는 경우 (`func fetchMovies(using repository: MoviesRepository)`)
    - 상속(Inheritance)을 통해 부모 클래스에 의존하는 경우 (`class MoviesRepository: BaseRepository`)
    - 인터페이스(프로토콜)에 의존하는 경우 (`protocol MoviesRepository {}` → `class DefaultMoviesRepository: MoviesRepository {}`)

### 🙋?  그렇다면 유스케이스는 외부 요소와 분리하기 위해 어떻게 의존성 역전 원칙을 적용할까?

유스케이스는 구체적인 구현체가 아니라 추상화된 **인터페이스(프로토콜)**에 의존하도록 설계하여 데이터 계층에 직접 의존하지 않도록 설계해야 한다. 이렇게 **의존성 주입(Dependency Injection)을 활용하여 유연성을 높이고, 유지보수성을 개선할 수 있다.**

- DIP를 적용한 의존 관계 예시

```swift
// 인터페이스 
protocol MoviesRepository {
    func fetchMoviesList(...) -> [Movie]
}

// 기본 구현체
final class DefaultMoviesRepository: MoviesRepository {
    func fetchMoviesList(...) -> [Movie] { /* ... */ }
}

// 유스케이스
class FetchMoviesUseCase {
    private let repository: MoviesRepository  // ✅ 인터페이스에 의존

    init(repository: MoviesRepository) {
        self.repository = repository
    }
}
```

### 🙋?  인터페이스와 유스케이스의 관계는 어떻게 되는 거야?

Clean Architecture에서 인터페이스(Interface)와 유즈케이스(Use Case)는 **서로 독립적이지만 긴밀한 관계**를 맺고 있으며, 인터페이스는 유즈케이스가 직접 네트워크, 데이터베이스 등의 구체적인 구현체에 의존하지 않도록 **추상화된 접근 방법을 제공하여 유스케이스와 외부 세계를 연결하는 역할을 한다.**

### 🙋?  그러면 이제 도메인 계층을 다 이해한 것 같아! 근데 엔티티와 유스케이스 중에 뭘 먼저 정의해?

Clean Architecture를 적용할 때 엔티티(Entity)와 유즈케이스(Use Case) 중 어떤 것을 먼저 정의해야 하는지는 프로젝트의 성격과 도메인의 복잡성에 따라 다를 수 있지만, 일반적으로 **엔티티(Entity)를 먼저 정의한 후, 이를 기반으로 유즈케이스(Use Case)를 설계하는 것이 권장**된다.

- 엔티티(Entity)를 먼저 정의하는 이유
    1. **도메인의 핵심 개념을 먼저 정리해야 유즈케이스를 설계할 수 있음**
        - 엔티티는 **비즈니스 규칙을 포함하는 핵심 계층**이며, 엔티티를 정의하지 않고 유스케이스를 설계하면, 유즈케이스에서 다뤄야 할 데이터 구조가 불명확해진다.
    2. **유즈케이스는 엔티티를 기반으로 동작**
        - 유즈케이스는 도메인 객체(Entity)를 조작하는 역할을 하므로, 엔티티가 먼저 정의되어야 한다.
        - 예를 들어, 엔티티가 있어야 유스케이스에서 반환할 데이터 구조를 정할 수 있음.
    3. **비즈니스 로직의 일관성을 유지할 수 있음**
        - 엔티티를 먼저 정의하면, 도메인 로직(비즈니스 규칙)이 유즈케이스에 의해 변경되지 않도록 보호할 수 있다.

## 데이터 계층(Data Layer)

Clean Architecture에서는 데이터 계층에는 데이터 모델이 있고, 프레젠테이션 계층(Presentation Layer)에는 뷰(View)와 뷰모델(ViewModel)이 존재한다. 

- 데이터 계층의 역할
    - 데이터 저장 및 가져오기 (API, 데이터베이스, 캐시)
    - Data Model(DTO)을 정의하여 데이터 변환 수행
    - Repository 패턴을 사용해 데이터 접근을 추상화
 

### 🙋?  "ViewModel이 비즈니스 로직을 모른다"라는 의미 무엇일까?

이는 ViewModel이 비즈니스 로직이 어떻게 구현되는지 알 필요가 없다는 것을 뜻한다. 

MVVM에서 ViewModel이 비즈니스 로직을 직접 다룬다면, API 호출, 데이터 변환, 캐싱 등의 로직이 ViewModel 안에 들어가게 된다. 그러나 Clean Architecture를 적용하면, ViewModel이 비즈니스 로직을 실행하긴 하지만, 그 내부 구현을 몰라도 된다. 이는 ViewModel이 비즈니스 로직을 직접 수행하는 것이 아니라, **비즈니스 로직을 Use Case가 담당하고, ViewModel은 단순히 Use Case를 실행하는 역할을 한다는 것을 말한다.** 

### 🙋?  **그렇다면 데이터 변환 처리는?**  (API → DTO → Entity)

API를 통해 받아온 데이터 변환을 Use Case에서 하지 않고, **Repository 또는 Interface Adapter 계층에서 처리**하는 것이 일반적이다. API에서 받아온 데이터는 일반적으로 Data Model (DTO)형태로 전달되며, 이를 Entity로 변환하는 과정이 필요하다. 

```swift
extension MoviesResponseDTO.MovieDTO {
    func toDomain() -> Movie {
        return .init(id: Movie.Identifier(id),
                     title: title,
                     genre: genre?.toDomain(),
                     posterPath: posterPath,
                     overview: overview,
                     releaseDate: dateFormatter.date(from: releaseDate ?? ""))
    }
}
```

## 프레젠테이션 계층(Presentation Layer)

- UI 업데이트 (View)
- Use Case를 호출하여 데이터를 가져옴 (ViewModel)

## 클린 아키텍처를 읽고나서…

프로젝트를 할 때 iOS 개발자가 나 혼자이거나, 팀원이 있어도 대규모의 앱이 아니었기 때문에 이제까지 MVC 패턴으로만 작업을 했고 아키텍처의 필요성을 느끼지 못했었다. 그래서 아키텍처가 중요하다고 해도 언젠간 읽어봐야지 생각만 하고 계속 미루던 일을 글또의 글 작성의 강제성을 빌려 드디어 읽어보게 되었다. 처음 책장을 펼쳤을 때 솔직히 너무 지루했다. 그리고 책을 다 읽을 때까지도 여전히 지루했고, 3주나 걸려서 다 읽을 수 있었다. 읽고 난 이후에 [**iOS-Clean-Architecture-MVVM**](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM)의 코드를 살펴보면서 그제서야 전반적인 내용을 이해할 수 있었다. 

앞으로 코드를 쳐보면서 연습을 해야하겠지만, 이로써 알고 있어야 할 한 가지를 끝냈다는 기분이 들어 마음이 조금 가벼워졌다! 그리고 읽기 전엔 굉장히 어렵게 느껴졌었는데, 읽어보고 나니 생각보다 어려운 내용이 아니었다는 것을 알게 되었다. 이제 그 다음은 Combine을 익혀보자!