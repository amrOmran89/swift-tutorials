---
title: "Bilingual iOS: LTR and RTL (English and Arabic)"
date: 2024-08-27 12:30:00 +0200
categories: [iOS, SwiftUI, UIKit, Localization]
tags: [ios, swiftui, uikit, localization, rtl, ltr, arabic, english]
layout: post
---

Designing for both English (Left‑To‑Right) and Arabic (Right‑To‑Left) requires thinking in terms of direction‑aware layout, text, and images. This post shows how to localize strings and build UI that naturally mirrors between LTR and RTL in SwiftUI and UIKit.

### 1) Project localization setup (en + ar)

1. In Xcode, select the project → Info → Localizations → add English and Arabic.
2. Add `Localizable.strings` for each language.
3. Use direction‑aware constraints (leading/trailing) and semantic text alignment.

`Localizable.strings (English)`
{% highlight text %}
/* Localizable.strings (en) */
"app_title" = "My Awesome App";
"welcome_title" = "Welcome";
"greeting_name" = "Hello, %@!";
"next" = "Next";
"previous" = "Previous";
"lorem" = "This is a long, multi-line paragraph demonstrating text wrapping and alignment in LTR.";
{% endhighlight %}

`Localizable.strings (Arabic)`
{% highlight text %}
/* Localizable.strings (ar) */
"app_title" = "تطبيقي الرائع";
"welcome_title" = "أهلاً وسهلاً";
"greeting_name" = "مرحباً، ‏%@!";
"next" = "التالي";
"previous" = "السابق";
"lorem" = "هذه فقرة طويلة متعددة الأسطر توضح التفاف النص والمحاذاة في اتجاه من اليمين إلى اليسار.";
{% endhighlight %}

Tip: Plurals and formatting can use `.stringsdict` if needed.

### 2) SwiftUI: Localized text and direction‑aware layout

- Use localization keys directly in `Text`.
- Prefer leading/trailing over left/right for paddings and alignments.
- Use direction‑aware SF Symbols like `chevron.forward` instead of `chevron.right`.

{% highlight swift %}
import SwiftUI

struct GreetingView: View {
    let name: String

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // Localized keys
            Text("welcome_title")
                .font(.title2)
                .bold()

            // With interpolation via String(format:)
            Text(String(format: NSLocalizedString("greeting_name", comment: "greeting with name"), name))

            // Paragraph text wraps and aligns naturally to leading edge (LTR or RTL)
            Text("lorem")
                .multilineTextAlignment(.leading)
                .padding(.top, 4)

            // Direction-aware icon + label
            HStack(spacing: 8) {
                Image(systemName: "chevron.forward")
                Text("next")
            }
            .frame(maxWidth: .infinity, alignment: .leading)
            .padding(.vertical, 8)
        }
        .padding(.horizontal, 16)
    }
}

#Preview("English – LTR") {
    GreetingView(name: "Sarah")
        .environment(\.locale, Locale(identifier: "en"))
        .environment(\.layoutDirection, .leftToRight)
}

#Preview("Arabic – RTL") {
    GreetingView(name: "سارة")
        .environment(\.locale, Locale(identifier: "ar"))
        .environment(\.layoutDirection, .rightToLeft)
}
{% endhighlight %}

Notes:
- `.multilineTextAlignment(.leading)` respects locale direction.
- Use `Spacer()` and `.frame(maxWidth: .infinity, alignment: .leading)` for layouts that should mirror.
- Prefer direction‑aware symbols (`.forward`, `.backward`) where applicable.

### 3) Testing language and direction at runtime

To test quickly without changing the device language, you can force layout direction at the app root. Use for development/testing only.

SwiftUI entry point example:
{% highlight swift %}
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            RootView()
                // Try different combinations while testing
                .environment(\.locale, Locale(identifier: "ar"))
                .environment(\.layoutDirection, .rightToLeft)
        }
    }
}
{% endhighlight %}

### 4) UIKit tips for RTL/LTR

- Use Auto Layout with `leadingAnchor`/`trailingAnchor` and avoid absolute left/right.
- Set `textAlignment = .natural` for labels/text views.
- Let views be `semanticContentAttribute = .unspecified` so they mirror automatically.
- Force direction for a subtree if needed (testing or exceptions):

{% highlight swift %}
let container = UIView()
container.semanticContentAttribute = .forceRightToLeft // or .forceLeftToRight
{% endhighlight %}

- Flip images for RTL when meaning implies direction:

{% highlight swift %}
let base = UIImage(named: "arrowIcon")!
let directionAware = base.imageFlippedForRightToLeftLayoutDirection()
imageView.image = directionAware
{% endhighlight %}

- Prefer system symbols that mirror automatically, e.g., `chevron.forward`, `arrowshape.turn.up.forward`, `play.fill` with `.semanticContentAttribute = .playback` for media controls if necessary.

### 5) SwiftUI images and mirroring

- SF Symbols (direction‑aware variants) will mirror automatically in RTL.
- For custom raster/PDF assets that must mirror, load via `UIImage` and wrap in `Image(uiImage:)`.

{% highlight swift %}
struct MirroredImage: View {
    var body: some View {
        let uiImage = UIImage(named: "chatBubble")!
            .imageFlippedForRightToLeftLayoutDirection()
        Image(uiImage: uiImage)
            .renderingMode(.original)
            .padding(.leading, 12)
    }
}
{% endhighlight %}

### 6) English and Arabic text examples

You can also mix explicit English and Arabic text when exploring typography and baseline behavior:

{% highlight swift %}
VStack(alignment: .leading, spacing: 8) {
    Text("Hello, world!")
    Text("مرحبا بالعالم")
    HStack(spacing: 12) {
        Text("USD 199")
        Text("١٩٩ دولار") // Eastern Arabic numerals
    }
}
.padding()
{% endhighlight %}

### 7) Bonus: localized numbers, dates, and lists

Use `FormatStyle` to get locale‑aware output automatically:

{% highlight swift %}
VStack(alignment: .leading) {
    Text(Date.now, format: .dateTime.year().month().day())
    Text(12345, format: .number)
    Text(["Tea", "Coffee", "Juice"], format: .list(type: .and))
}
// When locale is Arabic (ar), these automatically localize.
{% endhighlight %}

### Checklist and best practices

- Use `leading`/`trailing` instead of `left`/`right` paddings and constraints.
- Keep text alignment `.natural` or `.leading` to respect direction.
- Prefer direction‑aware symbols (`.forward`, `.backward`).
- Test with `.environment(\.locale, .init(identifier: "ar"))` and `.environment(\.layoutDirection, .rightToLeft)`.
- Mirror custom images when the visual meaning is directional.

With these patterns, your app should feel native in both English and Arabic, with correct mirroring for text, layout, and icons.


