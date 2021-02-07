## Reference
- [Lecture5](https://youtu.be/SIYdYpPXil4)
- [Lecture6](https://youtu.be/eHEeWzFP6O4)

### ViewBuilder
@ViewBuilder, this keyword can be tagged onto any function that returns some View.
The compiler will interpret what's in the curly braces of that function to be a View(list of Views → View, might be a TupleView = It can contains 10 views.).
_ConditionalContent View is created when there's an if-else in ViewBuilder.

```swift
// It would return a TupleView<RoundedRectangle, RoundedRectangle, Text>
@ViewBuilder
func front(of card: Card) -> some View {
	RoundedRectange(cornerRadius: 10)
	RoundedRectange(cornerRadius: 10)
	Text(card.text)
}

// GeometryReader allow to use ViewBuilder syntax
struct GeometryReader<Content> where Content: View {
	init(@ViewBuilder: content: @escaping (GeometryProxy) -> Content)
}

@ViewBuilder
func body(for size: CGSize) -> some View {
	if card.isFaceUp || !card.isMatched {
		ZStack { ... }
	}
}
```

### Why didn't we do this(ViewBuilder) in Grid?
Grid is asking you to provide a View for a certain item and we passed a function.
ViewBuilder's still kind of private.
Its implementation for example has no way to extract the Views from a TupleView, ConditionalContent(it's not public API). It's just you can't get them out of there.

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

### [ViewModifier](https://youtu.be/oDKDGCRdSHc?t=2514)
View are animated through their modifiers. The ViewModifier protocol has one function in it

```swift
protocol ViewModifier {
	// Protocol doesn't care what type Content is.
	associatedtype Content

	func body(content: Content) -> some View {
		return View // some View that represents a modification fo content
	}
}
```

```swift
// .modifier() returns a View that displays the body result.
Text("").modifier(Cardify(isFaceUp: true))

struct Cardify: ViewModifier {
	var isFaceUp: Bool
	func body(content: Content) -> some View {
		ZStack {
			if isFaceUp { 
				Pie()
				...
				content
			} else { ... }
		}
	}
}
```
