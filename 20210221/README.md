# SwiftUI

## @viewBuilder
some View를 리턴하는 모든 곳에는 해당 키워드@ViewBuilder를 넣을 수 있다.
Content는 뷰 리스트로 해석하고 하나로 결합한다.
여기서 결합되는 하나의 뷰는 (2개에서 10개의 뷰) TupleView일수도 있고 (if-else가 존재하는 경우) ConditionalContent View일수도 있고 (아무것도 없는 경우) EmptyView일수도 있다. 그리고 이것들의 조합이 될 수도 있다.

@ViewBuilder이 표시된 함수 또는 변수의 contents는 뷰의 리스트로 해석된다.
```
@ViewBuilder

func front(of card: Card) -> some View {
    RoundedRectangle(cornerRadius: 10)
    RoundedRectangle(cornerRadius: 10).stroke()
    Text(card.content)
}
```
위는 TupleView를 리턴한다.

아래와 같이 뷰를 리턴하는 경우 파라미터에도 @ViewBuilder을 붙일 수 있다.
```
struct GeometryReader<Content> where Content: View { 
    init(@ViewBuidler content: @escaping (GeometryProxy) -> Content) { ... }
}
```
여기서 content 파라미터는 뷰를 반환하는 함수이다.

## ViewModifier
> 뷰에 변화를 주고 새로운 View를 리턴해준다.
```
protocol ViewModifier {    
    func body(content: Content) -> some View { // Content는 dont care
        return some View that represents a modification of content
    }
}
```
content에 modification을 적용시킨 뷰 리턴함.

### Custom View Modifier

```
Text(“iOS”).modifier(Cardify(isFaceUp: true)) // eventually .cardify(isFaceUp: true)
struct Cardify: ViewModifier {
    var isFaceUp: Bool
    func body(content: Content) -> some View {
        ZStack {
            if isFaceUp {
                RoundedRectangle(cornerRadius: 10).fill(Color.white) 
                RoundedRectangle(cornerRadius: 10).stroke()
                content
            } else {
                RoundedRectangle(cornerRadius: 10)
            }
        }
    }
}
```
커스텀 ViewModifier는 .modifier()를 통해서 적용한다.  
ex) .modifier(Cardify( ~ ))  
여기서 content에 변화를 가하고 리턴한다. (ZStack안쪽)


아래와 같이 구현해주면 Text(“iOS”).cardify(isFaceUp: true)와 같이 호출할 수 있으며 어떤 뷰라도 사용할 수 있다.
```
extension View {
    func cardify(isFaceUp: Bool) -> some View {
        return self.modifier(Cardify(isFaceUp: isFaceUp))
    }
}
```

## @State

뷰 안에서 완전히 로컬라이즈 된 것.
얼럿 띄우기, 편집, 애니메이션과 같은 일시적인 상태에만 사용한다.

View 구조체는 (read - only) 읽기 전용이다.
예로 SwiftUI가 모든 뷰를 유지하는 데 사용하는 변수는 let! 이다.
뷰 생성 시 초기화되는 변수 외에는 변수가 있는 것이 소용이 없다.

뷰는 대부분 "stateless"이어야 하며 모델을 그리는 역할을 한다.
그래서 대부분 뷰는 어떤 상태가 필요하지 않기 때문에 읽기 전용이다. 
영구적인 상태는 모델에 속하고 일시적인 상태를 사용할 때 State를 사용한다.

```
@State private var somthingTemporary: SomeType 
```
SomeType은 아무 타입이나 사용 가능하다. 
@State var은 뷰의 변경이 필요할 때 사용된다. 
이를 사용하면 힙에 공간이 만들어지고 read-only뷰가 다시 만들어지면 새 버전이 해당 포인트를 가리키게 된다.
즉 뷰가 변경이 있어도 상태는 버려지지 않는다.
