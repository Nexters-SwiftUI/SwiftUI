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
		startAngle: ANgle.degrees(0-90),
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
				...
				content
			} else {
				...
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
