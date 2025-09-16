---
title: "SwiftUI Basics"
date: 2023-02-01 10:00:00 +0200
categories: [Swift, SwiftUI]
tags: [swift, swiftui, ios]
layout: post
---

This is a **dummy post** demonstrating SwiftUI basics.

{% highlight swift %}
struct ContentView: View {
    var body: some View {
        Text("Hello SwiftUI!")
            .font(.largeTitle)
            .padding()
    }
}
{% endhighlight %}

### Common modifiers

- **font**: Typography for `Text`.
- **foregroundColor / foregroundStyle**: Text and symbol color/style.
- **padding**: Inner spacing around a view.
- **background / overlay**: Layers behind/above a view.
- **cornerRadius / clipShape**: Rounding or clipping.
- **frame**: Propose width/height and alignment.
- **opacity / shadow**: Visual effects.

{% highlight swift %}
Text("Badge")
    .font(.headline)
    .foregroundStyle(.white)
    .padding(.horizontal, 12)
    .padding(.vertical, 6)
    .background(.blue)
    .cornerRadius(12)
{% endhighlight %}

### Chaining and modifier order

Modifiers return new views, so you can chain them. Order often matters: layout-affecting modifiers (`frame`, `padding`) earlier vs. visual effects (`background`, `overlay`) later can produce different results.

{% highlight swift %}
// Example: order changes the background area
Text("Hello")
    .padding()
    .background(.yellow)   // background includes the padding
    .cornerRadius(8)

Text("Hello")
    .background(.yellow)   // background only behind the text
    .padding()
    .cornerRadius(8)
{% endhighlight %}

### Conditional modifiers

Apply modifiers based on state or conditions.

{% highlight swift %}
struct ToggleableText: View {
    @State private var isOn = false

    var body: some View {
        VStack(spacing: 16) {
            Text(isOn ? "On" : "Off")
                .font(.title2)
                .foregroundStyle(isOn ? .green : .red)
                .opacity(isOn ? 1 : 0.6)

            Button(isOn ? "Turn Off" : "Turn On") {
                withAnimation(.easeInOut) { isOn.toggle() }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
{% endhighlight %}

A common pattern is extracting conditional logic into helper modifiers to keep chains readable.

### Custom ViewModifier

Create reusable styling via `ViewModifier` and a `View` extension.

{% highlight swift %}
struct PrimaryButtonModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .font(.headline)
            .foregroundStyle(.white)
            .padding(.vertical, 10)
            .padding(.horizontal, 16)
            .background(.blue)
            .cornerRadius(10)
            .shadow(radius: 2)
    }
}

extension View {
    func primaryButtonStyle() -> some View { modifier(PrimaryButtonModifier()) }
}

// Usage
Button("Continue") {}
    .primaryButtonStyle()
{% endhighlight %}

### Environment vs. local modifiers

Modifiers applied to container views (like `VStack`) can act as environment values for children (e.g., `font`, `tint`). Local modifiers on a child override environment ones.

{% highlight swift %}
VStack(alignment: .leading, spacing: 8) {
    Text("Title")           // gets .title3 from the VStack
    Text("Detail").font(.caption) // overrides to .caption
}
.font(.title3)
.tint(.purple)
{% endhighlight %}

### Layout-related modifiers

- **frame(width:height:alignment:)**: Propose size and align content inside the frame.
- **fixedSize()**: Prevent text/image from being compressed or truncated.
- **layoutPriority(_:)**: Control which views get space first.
- **alignmentGuide(_:computeValue:)**: Custom alignment in stacks.
- **offset / position**: Visual displacement vs. absolute position in parent.

{% highlight swift %}
HStack(alignment: .firstTextBaseline, spacing: 12) {
    Text("$").font(.title)
    Text("199").font(.largeTitle).layoutPriority(1)
    Text(".99").font(.title3)
}
.padding()
{% endhighlight %}

### Text and Image specifics

- `Text`: `lineLimit`, `multilineTextAlignment`, `kerning`, `baselineOffset`.
- `Image`: `resizable`, `scaledToFit/Fill`, `clipShape`, `renderingMode`.

{% highlight swift %}
Image("screenshot1")
    .resizable()
    .scaledToFit()
    .cornerRadius(8)
    .shadow(radius: 4)
{% endhighlight %}

### Accessibility and interaction

- **accessibilityLabel / Hint / Value**: Describe UI for VoiceOver.
- **accessibilityHidden**: Hide decorative elements.
- **onTapGesture / gesture**: Add interactions.
- **animation / withAnimation**: Animate state changes.

{% highlight swift %}
Text("Read more")
    .accessibilityLabel("Read more about SwiftUI basics")
    .onTapGesture { /* handle tap */ }
{% endhighlight %}