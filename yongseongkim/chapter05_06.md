## Reference

- [Lecture5](https://youtu.be/oDKDGCRdSHc)
- [Lecture6](https://youtu.be/eHEeWzFP6O4)


## [ViewBuilder](https://youtu.be/oDKDGCRdSHc?t=723)

@ViewBuilder, this keyword can be tagged onto any function that returns some View.
The compiler will interpret what's in the curly braces of that function to be a list of Views.
It builds that list of Views into a single View.

```swift
// https://www.swiftbysundell.com/tips/adding-swiftui-viewbuilder-to-functions/
private extension SongRow {
  // we need to use AnyView to perform type erasure in order to give both of our code branches the same return type.
  // Opaque result type inferred to be AnyView.
  func makeButtonLabel() -> some View {
    if isPlaying {
      return AnyView(PauseIcon())
    } else {
      return AnyView(PlayIcon())
    }
  }
}

private extension SongRow {
  @ViewBuilder func makeButtonLabel() -> some View {
    if isPlaying {
      PauseIcon()
    } else {
      PlayIcon()
    }
  }
}
```
- for 2 to 10 Views, TupleView(limited to 10)
```swift
// It would return a TupleView<RoundedRectangle, RoundedRectangle, Text>
@ViewBuilder
func front(of card: Card) -> some View {
  RoundedRectange(cornerRadius: 10)
  RoundedRectange(cornerRadius: 10)
  Text(card.text)
}
```
- When there's an if-else, ConditionalContent
```swift
@ViewBuilder
func body(for size: CGSize) -> some View {
  if card.isFaceUp || !card.isMatched {
    ZStack { ... }
  }
}
```
- If there's nothing at all, EmptyView
```swift
@ViewBuilder
func builder() -> some View {
}
```
- any combination of the above
```swift
// GeometryReader allow to use ViewBuilder syntax
// ZStack, HStack, VStack, ForEach, Group all do this same thing.
struct GeometryReader<Content> where Content: View {
  init(@ViewBuilder: content: @escaping (GeometryProxy) -> Content)
}
```
- [Why didn't we do this(ViewBuilder) in Grid?](https://youtu.be/oDKDGCRdSHc?t=1131)
  - Gird is asking you to provide a View for a certain item and we passed a function.
  - ViewBuilder's still kind of private.
  - Its implementation for example has no way to extract the Views from a TupleView, ConditionalContent(it's not public API). It's just you can't get them out of there.

## [Shape](https://youtu.be/oDKDGCRdSHc?t=1223)
A protocol inherits from View drawing itself by filling with the current foreground color.
This can be change with `stroke()` and `fill()`.
The Shape protocol implements View's body var for you. You are required to implement `path` function.

![shape_pie_example](./images/shape_pie_example.png)

```swift
// Draw a pie like Pac-Man.
struct Pie: Shape {
  var startAngle: Angle
  var endAngle: Angle
  var clockwise: Bool = false
  // That is the rect in which we're supposed to fit our Shape.
  // add lines, arcs, besizer curves, etc.
  func path(in rect: CGRect) -> Path {
    let center = CGPoint(x: rect.midX, y: rect.midY)
    let radius = min(rect.width, rect.height) / 2
    let start = CGPoint(
      x: center.x + radius * cos(CGFloat(startAngle.radians)),
      y: center.y + radius * sin(CGFloat(startAngle.radians))
    )
    var p = Path()
    p.move(to: center)
    p.addLine(to: start)
    p.addArc(
      center: center,
      radius: radius,
      startAngle: startAngle,
      endAngle: endAngle,
      clockwise: clockwise
    )
    p.addLine(to: center)
    return p
  }
}

var body: some View {
  // Why we substract 90 degrees? https://youtu.be/oDKDGCRdSHc?t=2384
  // The first thing to understand in iOS is that Angle Zero is out to the right
  // To be in the kind of degrees Where zero is straight up, we're gonna have to subtract 90 degrees.
  Pie(
    startAngle: Angle.degrees(0-90),
    endAngle: Angle.degrees(110-90),
    // Why we use clockwise? https://youtu.be/oDKDGCRdSHc?t=2430
    // In iOS System, the drawing coordinate system that you're drawing has (0,0) is the upper left and is upside down.
    // Starting up here at (0,0), Y is getting bigger as we get down.
    // So since this whole thing is upside down, clockwise and counterclockwise are going the other way as well.
    clockwise: true
  )
}
```

### [ViewModifier](https://youtu.be/oDKDGCRdSHc?t=2514)(aspectRatio, padding, font, foregroundColor …)

```swift
Text("")
	.aspectRatio(2/3) // = .modifier(AspectModifier(2/3))

// The ViewModifier protocol has one function in it
protocol ViewModifier {
  // Protocol doesn't care what type Content is.
  associatedtype Content

  func body(content: Content) -> some View {
    // return some View that represents a modification fo content
    return content
  }
}
```

```swift
// .modifier() returns a View that displays the body result.
Text("")
  // .aspectRatio(2/3) = .modifier(AspectModifier(2/3))
  .modifier(Cardify(isFaceUp: true))

struct Cardify: ViewModifier {
  var isFaceUp: Bool
  func body(content: Content) -> some View {
    ZStack {
      if isFaceUp { 
        Pie()
        /* ... */
        content
      } else {
        /* ... */
      }
    }
  }
}
```

### ViewBuilder and Modifier
```swift
// ViewBuilder
extension View {
  @ViewBuilder func visibleOrInvisible(_ isVisible: Bool) -> some View {
    if isVisible {
      self
    } else {
      // hidden: they do remain in the view hierarchy and affect layout.
      self.hidden()
    }
  }

  @ViewBuilder func visibleOrGone(_ isVisible: Bool) -> some View {
    if isVisible {
      self
    }
  }
}

struct ContentView: Vie {
  var isVisible: Bool

  var body: some View {
    Text("")
      .visibleOrGone(isVisible)
  }
}

// ViewModifier
struct VisibleOrInvisible: ViewModifier {
  var isVisible: Bool
    
  func body(content: Content) -> some View {
    Group {
      if isVisible {
        content
      } else {
        content.hidden()
      }
    }
  }
}

struct ContentView: View {
  var isVisible: Bool
    
  var body: some View {
    Text("")
      .modifier(VisibleOrInvisible(isVisible: isVisible))
  }
}
```

## [@State](https://youtu.be/3krC2c56ceQ?t=109)

There are a few rare times when a View needs some state like editing mode. 
We can create storage in our read-only Views and we do it by marking a var that stores the information we want with @State.
If I change @State variables, View might get redrawn. It changes in a way that makes my body draw differently, it'll get redrawn.

The space for @State var is gonna be allocated in the heap.
It's basically making a pointer so View has a pointer in it and it points in to the heap.
When your View gets rebuilt, cause your View's getting rebuilt all the time when isFaceUp changes in your CardView, you gotta make a new CardView.
When that happens, this pointer gets moved to the new version.

## [Animations](https://youtu.be/3krC2c56ceQ?t=498)

Animation only works for Views that are in a container that is already on screen.

### How do we make an animation go?

- Implicit Animation
  - One is an implicit animation where we're going to just mark a View.
  - Whenever one of the modifiers on this View changes, we're going to animate that change.
  - We don't usually put implicit animations on container Views, because it just ends up giving that animation to each of ths things inside. (Mostly, when you have a kind of a group of Views, they wanna work together to animate.)
```swift
// We don't usually put `.animation` on container Views
// A container just propagates the `.animation` modifier to all the Views it contains.
// It works like `.font`
Text("...")
  .opacity(scary ? 1 : 0)
  .rotationEffect(Angle.degrees(upsideDown ? 180 : 0))
  .animation(Animation.easeInOut)
```
- Explicit Animation
  - The second one is explicitly, where we are going to call some code that is going to result in some changes to ViewModifiers, Shapes or Views are gonna be coming and going. We are gonna wrap that code by calling the function withAnimation. Inside the curly braces there, we're gonna put the code and that's going to cause all the things that would change all those ViewModifier arguments that change, all the Views come and go. They're all gonna happen together in one concurrent animation. Explicit animation is where we cause an animation to happen all with the same duration and curve for a whole bunch of Views.
```swift
// Do not override an implicit animation.
// Even if there's an explicit animation going on at the same time, It's gonna have no effect on them.
// Implicitly animations always win.
withAnimation(.linear(duration: 2) {
  // do something that will cause ViewModifier/Shape arguments to change some.
  self.viewModel.resetGame()
}
```

The animation curve controls the rate at which the animation plays out. (linear, easeInOut, spring)
Mostly, we don't use them implicit animations that much because when you have a kind of a group of Views, they wanna work together to animate.
That's why putting `.animation` on a container like a ZStack.

### [Transitions](https://youtu.be/3krC2c56ceQ?t=1249)

Transitions specify how to animate the arrival and departure of Views.
A transition is only a pair of ViewModifiers.
One of the ViewModifiers is modifying the View for what is supposed to look like when it's there and the other on is modifying the View for what it's supposed to look like when it's not there.
And system is gonna animate between those two ViewModifiers to make that thing appear or disappear.
So a transition is really just a version of changes to the arguments of ViewModifier.
Unlike animation, transition does not get redistributed to a container's content Views.

```swift
// When you put `transition` on a ZStack or a VStack,
// it is talking about the transition when the entire ZStack comes on screen or goes off screen.
// It does not distribute it off into its things inside.
ZStack {
  if isFaceUp {
    RoundedRectangle() // default transition is .opacity
    Text().transition(.scale)
  } else {
    RoundedRectangle(cornerRadius: 10).transition(.identity)
  }
}
```
Group and ForEach, they do distribute `.transition` to their content View.
(Group and ForEach are container Views but since they don't put anything on screen anywhere, they're just essentially grouping or creating Views from a list of Identifiables.)
Transitions only happen when an explicit animation is going on.
Transitions do not really work with implicit animations.
That's the only time a transition animation will happen is when you are animating it.

### AnyTransition

`AnyTransition` is a type-erased transition.
`AnyTransition` is just a struct that has an initializer that takes which is a transition we had modifiers and all that stuff.
It just knows how to do the transition thing with it. 
AnyTansition is just a struct. It has some static vars on it for the built in transitions like `.opacity`, `scale`.

### onAppear

When a View appears on screen, then it calls `.onAppear {}`. 

### [How do Shapes and ViewModifiers participate in whole animation system?](https://youtu.be/3krC2c56ceQ?t=2185)

The animation system divides up the duration of the animation into little tiny pieces, depending on the curve.
And then it just asks all the shapes and ViewModifiers that are Animatable.
And piecing it together into like a little movie which is the animation.
The communication between the animation system and ViewModifiers and Shape is just `animatableData` variable in protocol `Animatable`.
```swift
struct Cardify: ViewModifier, Animatable {
  var rotation: Double
  var isFaceUp: Bool {
    rotation < 90
  }

  // Just renamed rotation to be animatableData.
  // animatableData is the name that the animation systems is going to look for.
  var animatableData: Double {
    get { return rotation }
    set { rotation = newValue }
  }

  func body(content: Content) -> some View {
    // Flip animation.
    ZStack {
      if isFaceUp {
        RoundedRectangle().fill(Color.white)
        content
      } else {
        RoundedRectangle().fill(Color.orange)
      }
      .rotation3DEffect(Angle.degrees(rotation), axis: (0, 1, 0))
    }
  }
}
```

### AnimatableModifier

`AnimatableModifier` is just ViewModifier and Animatable.
`AnimatableModifier` also signals to the system, I'm gonna be ViewModifier that wants to participate in the animation system.
