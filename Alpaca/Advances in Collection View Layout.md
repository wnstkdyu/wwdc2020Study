# Advances in Collction View Layout

컬렉션 뷰의 레이아웃을 잡는 새로운 API에 대해 소개한다. 먼저 현재 우리는 컬렉션 뷰의 레이아웃을 어떻게 잡고 있을까?

## Current state of the art

### Flow Layout

컬렉션 뷰는 iOS 6부터 등장하였으며 이 컬렉션 뷰의 레이아웃을 잡는 핵심 객체는 `UICollectionViewLayout`이다. 또한 애플에서는 추상 객체를 구현한 `UICollectionViewFlowLayout` 객체를 발표하였다.

`UICollectionViewFlowLayout` 객체는 **Line-based** 레이아웃으로 일반적인 디자인에 유용하게 쓰여 왔다. Line-based로 한 줄에 셀들을 채우고 다 채우면 다음 줄로 넘어가는 꽤 직관적인 레이아웃을 구성하기 이해하기도 쉽다.

하지만 지금의 앱에서는 어떨까?

기기가 점점 커지면서 화면이 표현할 수 있는 것들이 늘어나게 되고 앱 디자인은 점점 더 복잡한 레이아웃을 갖게 되었다. 이러한 레이아웃에는 Flow 레이아웃이 맞지 않으며 Custom 레이아웃을 택할 수 밖에 없을 것이다.

### Building Custom Layouts

커스텀 레이아웃을 만들 때는 다음의 것들을 다루거나 극복해야 한다.

- Boilerplace code
- Performance considerations
- Supplementary and decoration view challenges
- Self-sizing challenges

이러한 요소들을 고려하고 내놓은 새로운 구체 레이아웃 객체가 Compositional Layout이다.

## Compositional Layout

Compositional 레이아웃을 설계할 때의 기본적인 원칙은 다음의 세 가지이다.

- **Composable**: 작은 요소들을 합쳐 복잡한 레이아웃을 구성할 수 있다.
- **Flexible**: 어떤 레이아웃이라도 Compositional 레이아웃을 사용하여 만들 수 있다.
- **Fast**: 기본적으로 성능을 고려하여 프레임워크 내부에서 최적화를 했기 때문에 사용하는 입장에서 신경 쓸 필요가 없다.

Compositional 레이아웃은 *선언형* API로 설계되어 단지 무엇을 원하는 지 적음으로써 레이아웃을 만들 수 있다.

Compositional Layout의 핵심 객체들을 나타낸 것이다.

<img src="https://user-images.githubusercontent.com/22453984/92994716-9d57ba00-f537-11ea-865d-2cf2073d655e.png" alt="스크린샷 2020-09-12 오후 8 35 56" style="zoom:67%;" />

이렇게 핵심 객체들을 쭉 생성하는 것만으로도 간단한 리스트 레이아웃을 만들 수 있으며 중요하게 강조하는 것은 문제가 커져도 코드의 양이 선형적으로 커지지 않는다는 것이다. 

또한 눈여겨 볼 점은 객체들이 어떠한 체계를 가지고 행동한다는 것이다. Item은 모여서 Group을 이루고, Group은 모여 Section이 되며, 마지막으로 이 Section이 모여 Layout 객체가 된다. 이러한 패턴은 Compositional 레이아웃에서 계속해서 유지된다.

<img src="https://user-images.githubusercontent.com/22453984/92994798-6930c900-f538-11ea-88a9-8ad81da23a8b.png" alt="스크린샷 2020-09-12 오후 8 41 59" style="zoom:67%;" />

### Core Concepts

- `NSCollectionViewLayoutSize`
- `NSCollectionViewLayoutItem`
- `NSCollectionLayoutGroup`
- `NSCollectionLayoutSection`

#### NSCollectionViewLayoutSize

모든 요소는 명시적인 사이즈를 가지고 있으며 이 사이즈를 나타낸 객체로 너비와 높이를 추상화한 객체인 `NSCollectionLayoutDimension` 이 모여 만들어진다.

눈여겨 봐야 할 점은 너비와 높이 또한 단순 숫자가 아닌 추상화 객체를 통해 표현된다는 점이다.

##### NSCollectionLayoutDimension

높이와 너비가 이 타입으로 추상화되어 좀 더 풍부하고 선언적으로 레이아웃을 표현할 수 있다.

<img src="https://user-images.githubusercontent.com/22453984/92994892-64b8e000-f539-11ea-9331-376463fbb09d.png" alt="스크린샷 2020-09-12 오후 8 49 00" style="zoom:50%;" />

#### NSCollectionViewLayoutItem

Cell이나 Supplementary를 나타내는 객체.

#### NSCollectionLayoutGroup

Item 들이 모여 생기는 객체로 레이아웃의 기본 단위이며 Horizontal, Vertical, Custom의 세 형태를 가지고 있다.

#### NSCollectionLayoutSection

Group 들이 모여 생기는 객체로 Section의 레이아웃을 의미.

#### UICollectionViewCompositonalLayout

Section 들이 모여 생기는 레이아웃 객체로 Apple 플랫폼에 상관없이 동일한 API를 가진다. Section 마다 레이아웃을 다르게 할 수 있게 클로저 형태의 API를 제공한다.

<img src="https://user-images.githubusercontent.com/22453984/92995012-7fd81f80-f53a-11ea-8bb9-94410f49b3e7.png" alt="스크린샷 2020-09-12 오후 8 56 56" style="zoom:50%;" />



## Advanced Layouts

### NSCollectionLayoutSupplementaryItem

- Badges
- Headers
- Footers

이 세 가지 Supplementary 뷰들의 사용이 점점 더 많아지고 있기 때문에 새 API에서는 이 뷰들을 좀 더 쉽게 쓸 수 있게 만들었고 Item이나 Group에 달아(Anchored) 사용할 수 있게 하였다.

##### NSCollectionLayoutAnchor

이 Anchor 객체를 통해 Supplementary 뷰가 Item이나 Group의 어느 위치에 있어야 하는지 쉽게 선언하고 구현할 수 있다.

<img src="https://user-images.githubusercontent.com/22453984/92995626-19560000-f540-11ea-8e56-1ab780c3a78f.png" alt="스크린샷 2020-09-12 오후 9 36 57" style="zoom:50%;" />

<img src="https://user-images.githubusercontent.com/22453984/92995668-5c17d800-f540-11ea-91ba-25dce600dc46.png" alt="스크린샷 2020-09-12 오후 9 38 51" style="zoom:50%;" />

### Nested NSCollectionLayoutGroup

- NSCollectionLayoutGroup` is-a `NSCollectionLayoutItem`

- Nesting depth에 제한이 없다.

Group은 Item을 상속한 객체이기 때문에 Group 안에 다른 Group이 들어가는 복잡한 레이아웃 또한 쉽게 가능하다.

<img src="https://docs-assets.developer.apple.com/published/c16512182a/original-1585241227.png" alt="" style="zoom:67%;" />

<img src="https://user-images.githubusercontent.com/22453984/92995885-591de700-f542-11ea-9229-0658aff18845.png" alt="스크린샷 2020-09-12 오후 9 52 59" style="zoom:50%;" />

### Nested CollectionViews

CollectionView를 중첩하는 레이아웃은 지금의 앱에서는 매우 익숙한 형태이며 새로운 API를 이용하면 이러한 레이아웃을 쉽게 적용할 수 있다.

<img src="https://user-images.githubusercontent.com/22453984/92995961-e06b5a80-f542-11ea-9a7c-c583e3e05995.png" alt="스크린샷 2020-09-12 오후 9 56 53" style="zoom: 50%;" />

##### 샘플 앱 코드

더 이상 CollectionView를 중첩으로 사용하는 것이 아닌 Section을 중첩으로 사용함으로써 이전과 같은 레이아웃을 만들 수 있다.

```swift
let containerGroup = NSCollectionLayoutGroup.horizontal(
    layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(containerGroupFractionalWidth),
                                      heightDimension: .fractionalHeight(0.4)),
    subitems: [leadingItem, trailingGroup])
let section = NSCollectionLayoutSection(group: containerGroup)
section.orthogonalScrollingBehavior = sectionKind.orthogonalScrollingBehavior()
```

