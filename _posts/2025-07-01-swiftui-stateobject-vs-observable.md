---
title: "Understanding @StateObject vs @State with @Observable in SwiftUI"
date: 2025-07-01
categories: [SwiftUI, iOS]
tags: [swift, swiftui, ios]
layout: post
---

SwiftUI provides different ways to manage state in your views, and it can be a bit confusing to understand when to use `@StateObject` with `ObservableObject` versus `@State` with the new `@Observable` macro. In this post, we'll break down the differences and best use cases for each.

---

## @StateObject with ObservableObject

Before Swift 6.2, the main way to manage reference-type observable state was using a class conforming to `ObservableObject`.

{% highlight swift %}
 import SwiftUI

 class Counter: ObservableObject {
    @Published var value: Int = 0
 }

 struct CounterView: View {
    @StateObject private var counter = Counter()
    
    var body: some View {
        VStack {
            Text("Value: \(counter.value)")
            Button("Increment") {
                counter.value += 1
            }
        }
    }
 }
{% endhighlight %}

**Key points:**

* `@StateObject` creates and owns the instance of a reference type.
* Properties marked with `@Published` notify SwiftUI of changes, triggering view updates.
* Use `@StateObject` when the view **owns the object** and manages its lifecycle.
* For passing the object down the view hierarchy, use `@ObservedObject` or `@EnvironmentObject`.

---

## @State with @Observable macro

Swift 6.2 introduced the `@Observable` macro, which works with structs and allows automatic observation of properties.

{% highlight swift %}
import SwiftUI

@Observable
struct Counter {
    var value: Int = 0
}

struct CounterView: View {
    @State private var counter = Counter()
    
    var body: some View {
        VStack {
            Text("Value: \(counter.value)")
            Button("Increment") {
                counter.value += 1
            }
        }
    }
}
{% endhighlight %}

**Key points:**

* `@Observable` automatically makes struct properties observable.
* Store the struct in a view with `@State`.
* Works with **value types**, making state management safer and simpler.
* No need for `@Published`; all properties are observed automatically.


## When to Use Each

* **`@StateObject + ObservableObject`**: Use when you need reference semantics or shared state between multiple views.
* **`@State + @Observable`**: Use for lightweight, Swift-native value types that automatically notify SwiftUI of changes.

---


This shows both approaches in the same view for comparison. The **class** version uses `@StateObject` and `@Published`, while the **struct** version uses `@Observable` and `@State`.

---

## Conclusion

SwiftUI is evolving toward value-type observables with the `@Observable` macro, making state management safer and simpler. However, `ObservableObject` and `@StateObject` are still essential when **reference semantics** or **shared state across multiple views** are required.
