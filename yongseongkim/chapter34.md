## Reference
- [Lecture3](https://youtu.be/SIYdYpPXil4)
- [Lecture4](https://youtu.be/eHEeWzFP6O4)

## Make Reactive View
```swift
// The way we make the ViewModel participate in reactive thing is using a constrains and gains thing called ObservableObject.
class EmojiMemoryGame: ObservableObject {
	// Broadcasts every time something changes, we just need to @Published all of our vars that we care whether they change.
	// So it's publishing every time the Model changes.
	@Publishsed var model: MemoryGame 
}

struct EmojiMemoryGameView: View {
	// When it sees this ViewModel Publishing, it redraws.
	// Redraws every time it sees this thing, say objectWillChange.send.
	// This var has an ObservableObject in it. 
	@ObservedObject var viewModel: EmojiMemoryGame
	// SwiftUI is smart about seeing whether something actually changed. 
}
```

## [Layout: How do we decide whether all our Views go on Screen?](https://youtu.be/SIYdYpPXil4?t=2996)
1. Container Views "offer" space to the Views inside them.
2. View then choose what size they want to be.
3. Container Views then position the Views inside of them.

### Container Views
The layout process of stacks like HStack and VStack have layout steps like below.
1. The stacks divide up the space that they're offered equally.
2. Then they offer it to the least flexible Views first.
   - Least flexible means like fixed size `Image`
   - Slightly flexible is `Text`. Text always wants to size itself to fit the text inside of it.
   - Very flexible is all the Shape like `RoundedRectangle`.
3. As the next step, the stack deducts the size of the first child from available space and devide it in to equal portions again.
4. Repeats this process.

- Spacer always takes all the space offered to it and draws nothing.
- `Divider` only uses enough space to draw that line. It takes the minimum space needed to fit the line.
- `layoutPriority` trumps "least flexible". The default value is 0.
- `ForEach` defers to its container to lay out the Views inside of it. 

### GeometryReader: How does a View know what space was offered to it?
`GeometryReader` wraps around whatever thing that you want to adapt to the size.
`GeometryProxy` itself always accepts all the space offered to it.
`edgesIgnoringSafeArea`: The safe area is best visualized by thinking of the notch on the iPhone X.
```swift
struct EmojiGameView: View {
	var body: some View {
		HStack {
			ForEach(cards) { card in
				CardView(card: card)
			}
		}
	}
}

struct CardView: View {
	var body: some View {
		GeometryReader { geometry in
            // The `size` var is the amount of space that is being "offered" to us by our container.
			ZStack {
				Text(self.card.content)
			}
			.font(Font.system(size: min(geometry.size.width, geometry.size.height)))
		}
	}
}
```

### Group
Group is like a ZStack or any of these other things in that its function argument here is a View builder.
It does not lay the children out or try to position them in any way.
It's good for grouping things
```swift
struct EmojiMemoryGameView: View {
	var body: some View {
		Grid(items: viewModel.cards) { card in
			CardView(card)
		}
	}
}

struct Grid<Item, ItemView>: View where Item: Identifiable, ItemView: View {
	var items: [Item]
	var viewForItem: (Item) -> ItemView
	
	// This function is going to escape from this initializer.
	// Whatever code is in the function has to be able to be called later.
	// The functions that can be called later live in the heap and they have pointers to them.
	// All the things inside the function have be kept in the heap.
	// If anything in the function points back, we've got a situation where two things in the heap are pointing to each other.
	// They're never gonna be able to go away. (memory cycle)
	init(_ items: [Item], viewForItem: @escaping (Item) -> ItemView) {
	}

	var body: some View {
		GeometryReader { geometry in
			self.body(for: GridLayout(itemCount: items.count, in: geometry.size))
		}
	}

	func body(for layout: GridLayout) -> some View {
		ForEach(items) { item in
			self.body(for: item, in: layout)
		}
	}

	func body(for item: Item, in layout: GridLayout) -> some View {
		let index = items.firstIndex(matching: item)
		return Group {
			// What does Group do if index is nil?
			// It's gonna return a group that has some sort of empty content.
			if index != nil {
				viewForItem(item)
					.frame(width: layout.itemSize.width, height: layout.itemSize.height)
					.position(layout.location(ofItemAt: index))
			}
		}
	}
}

struct GridLayout {
	var size: CGSize
	var rowCount: Int = 0
	var columnCount: Int = 0
	var itemSize: CGSize {}

	func location(ofItemAt index: Int) -> CGPoint {}
}
```

### Modifier
Modifiers essentially "contain" the View they modify.
```swift
// It's going to reduce, take 10 points off the edges of it 
// and pass that space that's left onto the next View
// which is the View returned by the foregroundColor modifier(actually HStack).
HStack {
	ForEach(viewModel.cards) { card in
		// Each aspectRatio View is going to set its width to be its share of the HStack's width.
		// and then pick a height that matches the aspect ratio.
		// Or if the height is limited here, it might be the other way around
		// where the aspectRatio View takes all of the height it's offered
		// and instead chooses a width that's less to fit.
		// It depends on whichever is gonna fit best in the space that is offered.
		CardView(card: card).aspectRatio(2/3, contentMode: .fit)
	}
}
.foregroundColor(Color.orange)
.padding(10)
```
