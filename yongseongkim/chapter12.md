## Reference
- [Lecture1](https://youtu.be/jbtqIBpUG7g)
- [Lecture2](https://youtu.be/4GjXq2Sr55Q)

```swift
// It's not object-oriented programming like the superclass. This is functional programming.
// Something like a ContentView functions like a View or it behaves like a View.
// ContentView behaves like a View. A View is jus t a rectangle area on screen. 
struct ContentView: View {
  var body: some View {
    HStack {
      ForEach(0..<4) { idx in
        CardView()
      }
    }
  }
}

struct CardView: View {
  var isFaceUp: Bool
  var body: some View {
    ZStack {
      if isFaceUp {
        RoundedRectangle(cornerRadius: 10.0).fill(Color.white)
        RoundedRectangle(cornerRadius: 10.0).stroke(lineWidth: 3)
        Text("")
      } else {
        RoundedRectangle(cornerRadius: 10.0).fill()
      }
    }
    // Views inside of ZStack use foregroundColor orange.
    // It gets passed down into the environment for all the views on the inside of ZStack.
    .foregroundColor(Color.orange)
  }
}
```

## MVVM(Model - View - ViewModel)
The Model is UI independent. doesn't import SwiftUI. encapsulate the data and the logic. 
All the state about the game is in the Model. The View itself doesn't need to have any statea.
### [Why is the imperative model kind of bad for UI?](https://youtu.be/4GjXq2Sr55Q?t=256)
The main reason has to do with the time.
If you want to understand how your UI is built, you need this other dimension of time to know when this function can be called and what function call depends on some other call happening first and then once your UI is up and running, someone could call a function to change your UI at any time so you have to kind of be always on guard and ready for that.
[When it changes, you get notified.](https://youtu.be/4GjXq2Sr55Q?t=576)
