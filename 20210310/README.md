# SwiftUI

### UIViewRepresentable 프로토콜
- SwiftUI에서 UIView를 추가할 때 사용
- SwiftUI에서 업데이트가 발생할 때 실행
- 시스템은 적절한 시간에 Representable 가능한 인스턴스의 메서드를 호출해 뷰를 만들고 업데이트 함.
- 시스템은 뷰 내에서 발생하는 변경 사항을 SwiftUI 다른 인터페이스의 다른 곳에 자동 전달하지 않음. (상호작용을 용이하게 하려면 Coordinate사용)
```
struct ContentView: View {
   var body: some View {
      VStack {
         Text("Global Sales")
         MyRepresentedCustomView()
      }
   }
}
```

### UIViewControllerRepresentable 프로토콜
- SwiftUI에서 UIViewController추가할 때 사용
- func makeUIViewController(context: Context) -> some UIViewController
- func updateUIViewController(_ uiViewController: some UIViewController, context: Context)
- 시스템은 뷰 컨트롤러 내에서 발생하는 변경 사항을 SwiftUI 다른 인터페이스의 다른 곳에 자동 전달하지 않음. (상호작용을 용이하게 하려면 Coordinate사용)

### Representables 5컴포넌트
- (view,controller를 만들때 사용하는 함수)
> func makeUIIVew{Controller}(context: Context) -> view/controller
- (적당한 때에 UIKit를 업데이트 시켜주는 함수 (bindings change등)
> func updateUIView{Controller}(view/controller, context: Context)
- (진행되는 델리게이트 활동을 처리하는 Coordinator 객체)  
> func makeCoordinator() -> Coordinator
- (Context 예를들어 Coordinator, SwiftUI의 environment, 애니메이션 트랜잭션)
> //passed into the methods above
- (view, controller가 사라질 때 정리해야하는 경우 - tear down)
> func dismantleUIVIew{Controller}(view/controller, coordinator: Coordinator)

