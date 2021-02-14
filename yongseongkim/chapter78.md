## Reference
[Lecture 7](https://youtu.be/tmx-OwkBWxA)
[Lecture 8](https://youtu.be/mz-rNLWJ0bk)

## Colors and Images

```swift
// can be a Color specifier
.foregroundColor(Color.green)
// can also be a ShapeStyle 
.fill(Color.blue)
// could even be a View.
Color.white
```
UIColor comes from the old pre-SwiftUI world. Things that start with UI are from UIKIt. It was good at manipulating colors.

[Image in SwiftUI is primarily a View and it's a View that displays an image. It's not something that you would make a var of type Image and it holds an image. 
It will use `Image(_ name: String)` as the name of an image that it finds in your Assets.xcassets file. UIImage, that's the thing that if you had a var that was gonna hold an image in it, like a JPEG image, it would be of type UIImage. It was so good at handling images.](https://youtu.be/tmx-OwkBWxA?t=190)

## Multithreade Programming

### [Threads](https://youtu.be/tmx-OwkBWxA?t=454)

It just appears that you've got different pieces of code executing simultaneously. They're running at the exact same time somehow.

### [Queues](https://youtu.be/tmx-OwkBWxA?t=498)

A queue is nothing more that a bunch of blocks of code that are sitting in line just patiently waiting to get a thread of execution that will go and run them. 

[Anytime you want to do something in the UI, you have to use the main queue. 
There is some stuff like animation where all the calculations of the animation, the inter-frame, the animatableData that is going to happen in another queue off the main queue. But it's all gonna get coordinated back onto the main queue to do the drawing.  So the stuff all happens without smashing into each other.](https://youtu.be/tmx-OwkBWxA?t=592) 

[To do the non-UI stuff is in the pile of background queues. The system manages a bunch of threads to go pull blocks of code off of these background queues. They're running simultaneously with what's on the main queue. The main queue always gets higher priority.](https://youtu.be/tmx-OwkBWxA?t=653) 

[GCD's dispatching the code from the queues to be executed by the threads. One is getting a queue and two, plopping a block of code on the queue.](https://youtu.be/tmx-OwkBWxA?t=714) 

[`queue.sync` blocks and wait. It waits until it has taken this block of code off of its queue and executed it to completion. So we would never call queue.sync in UI code because it would block the UI.](https://youtu.be/tmx-OwkBWxA?t=881) 

```swift
// https://youtu.be/tmx-OwkBWxA?t=1055
DispatchQueue(global. userInitiated).async {
	// do something that might take a long time
	DispatchQueue.main.async {
		// UI code
	}
}
```

## Drag and Drop
<br>

| Drag Image from Browser | Drag Text from Palette|
| ------------- | ------------- |
| ![image_drag](./images/image_drag.png) | ![text_drag](./images/text_drag.png) |

```swift
struct EmojiArtDocumentView: View {
	var body: some View {
		VStack {
			ScrollView(.horizontal) {
				// https://youtu.be/tmx-OwkBWxA?t=4971
				HStack {
					ForEach(EmojiArtDocument.palette.map { String($0) }, id: \.self) {
						Text(emoji)
							// This function just needs to return the things to drag.
							.onDrag { NSItemProvider(object: emoji as NSString) }
					}
				}
			}
			GeometryReader { geometry in
				Rectangle().foreground(.white) // can replace it with `Color.white`
					// [Why didn't I just make a ZStack here?](https://youtu.be/tmx-OwkBWxA?t=3681)
					// This has to do with sizing.
					// We want DocumentView to be sized like it was a Rectangle.
					// Basically we wanted to use all the space offered to it. (Shape take all the space offered to them.)
					// We wouldn't want it to size it to the Image, because they size themselves to the size of the image.
					// (If you have a small image, we would have small little View here.)
					.overlay(Image(self.document.backgroiundImage!))
					.edgesIgnoringSafeArea([.horizontal, .bottom])
					// https://youtu.be/tmx-OwkBWxA?t=3290
					// The first argument is what kind of things do you want to drop.
					// `public.image` is what's called a URI.
					// It specifies kind of a public agreement of the type of things that are images.
					// But If you drag and drop an image, very likely the providewr of that image can also provide you its URL.
					// `isTargeted` argument is what's called a Binding.
					.onDrop(of: ["pulbic.image", "public.text"], isTargeted: nil) { providers, location in
						// The first argument `providers` are objects called NSItemProviders that provide the information that's being dropped.
						// The second argument `location` is the location of the drop.
						// It's currently in global coordinate systems, the coordinate system of the entire device.
						var location = geometry.convert(location, from: .global)
						location = CGPoint(x: locationx - geometry.sizw.width/2, y: location.y - geometry.size.height/2)
						// This function is supposed to return whether the drop succeeded. 
						return self.drop(providers: providers, at: location)
					}
				}
			}
	}

	private func drop(providers: [NSItemProdiver]) -> Bool {
		// It calls a function where it passes you the URL that it was able to find. 
		var found = providers.loadFirstObject(ofType: URL.self) { url in
			self.document.setBackground(url: url)
		}
		if !found {
			found = providers.loadObjects(ofType: String.self) { string in
				self.document.addEmoji(string, at: location, size: self.defaultEmojiSize)
			}
		}
		return found
	}
}
```

```swift
// [Trick - Using ViewBuilder](https://youtu.be/tmx-OwkBWxA?t=3915)
Color.white.overlay(
	Group {
		if self.document.backgroundImage != nil {
			Image(uiImage: self.document.backgroundImage!)
		}
	}
)
```

## UserDefaults

## Gestures
