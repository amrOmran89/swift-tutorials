---
title: "async await @concurrent with @MainActor in Swift 6.2"
date: 2025-09-16
categories: [Swift, Concurrency]
layout: post
---

In Swift 6.2, the `@concurrent` attribute is introduced to explicitly indicate that a function should execute concurrently on a background executor, overriding any default actor isolation such as `@MainActor`. This is particularly useful for offloading CPU-intensive tasks from the main thread to maintain a responsive user interface.

## Understanding `@concurrent` and `@MainActor`

By default, Swift 6.2 assumes that code runs on the `@MainActor`, meaning it executes on the main thread unless specified otherwise. This default behavior simplifies concurrency management for UI-related code.

However, for tasks that are computationally intensive and could block the main thread, you can use the `@concurrent` attribute to explicitly run these functions on a background thread. This ensures that such tasks do not interfere with the responsiveness of the UI.

### Using `@concurrent` in a `@MainActor` Context

When you define a class or struct with the `@MainActor` attribute, all its methods and properties are, by default, isolated to the main actor. To offload specific methods to a background thread, you can mark them with `@concurrent`. Here's an example:

{% highlight swift %}
@MainActor
class DataProcessor {
    // This method runs on the main thread
    func updateUI() {
        // UI update code
    }

    // This method runs concurrently on a background thread
    @concurrent
    func performHeavyComputation() async {
        // Intensive computation code
    }
}
{% endhighlight %}

In this example, `updateUI()` executes on the main thread, while `performHeavyComputation()` is offloaded to a background thread due to the `@concurrent` attribute.

### Key Points to Remember

- **Default Actor Isolation**: In Swift 6.2, code is assumed to run on the `@MainActor` by default, simplifying concurrency for UI-related tasks.

- **Explicit Concurrency with `@concurrent`**: Use the `@concurrent` attribute to explicitly mark functions that should run concurrently on a background thread, overriding the default `@MainActor` isolation.

- **Combining `@MainActor` and `@concurrent`**: Within a `@MainActor`-isolated context, you can use `@concurrent` to offload specific methods to background threads, ensuring that intensive tasks do not block the main thread.

By leveraging `@concurrent` alongside `@MainActor`, Swift 6.2 provides developers with fine-grained control over concurrency, enabling efficient and responsive applications.

## SwiftUI ViewModel with `.task {}`

{% highlight swift %}
import SwiftUI

// If your project uses default-isolation MainActor, you can omit @MainActor below.
@MainActor
final class ArticlesViewModel: ObservableObject {
    @Published private(set) var articles: [Article] = []
    @Published private(set) var isLoading: Bool = false

    func load() async {
        isLoading = true
        defer { isLoading = false }

        // 1) Fetch (typically fine off the main actor)
        let items = try? await API.fetchArticles()

        // 2) CPU-heavy post-processing off the main actor
        let processed: [Article] = await Task.detached(priority: .userInitiated) {
            (items ?? []).sorted(by: { $0.date > $1.date })
        }.value

        // 3) Publish results back on the main actor (we're already @MainActor here)
        articles = processed
    }
}

struct ArticlesView: View {
    @StateObject private var viewModel = ArticlesViewModel()

    var body: some View {
        List(viewModel.articles) { article in
            Text(article.title)
        }
        // Kicks off the load when the view appears
        .task {
            await viewModel.load()
        }
    }
}
{% endhighlight %}

### `.task(id:)` to rerun work when inputs change

{% highlight swift %}
struct SearchView: View {
    @StateObject private var viewModel = ArticlesViewModel()
    @State private var query: String = ""

    var body: some View {
        VStack {
            TextField("Search", text: $query)
            List(viewModel.articles) { article in
                Text(article.title)
            }
        }
        // Re-runs when `query` changes, cancelling the previous task
        .task(id: query) {
            await viewModel.search(query: query)
        }
    }
}

extension ArticlesViewModel {
    func search(query: String) async {
        guard !query.isEmpty else {
            articles = []
            return
        }

        let items = try? await API.searchArticles(query)
        let processed: [Article] = await Task.detached(priority: .userInitiated) {
            (items ?? []).filter { $0.title.localizedCaseInsensitiveContains(query) }
        }.value
        articles = processed
    }
}
{% endhighlight %}

## Default actor isolation in practice

- You can make the default actor `@MainActor` for a target or module, reducing the need to annotate every type or function.
- In Xcode, add the Swift compiler flag: `-default-isolation MainActor`.
- With Swift Package Manager, you can set it per target:

{% highlight swift %}
// swift-tools-version: 6.2
import PackageDescription

let package = Package(
    name: "App",
    targets: [
        .target(
            name: "App",
            swiftSettings: [
                .defaultIsolation(MainActor.self)
            ]
        )
    ]
)
{% endhighlight %}

When the default is `@MainActor`, your SwiftUI `View` models and UI helpers are main-actor isolated by default, and you can explicitly offload heavy work using background tasks (e.g., `Task.detached`) to keep the UI responsive.


Tip: Another option is to keep heavy work in `nonisolated` helper functions that do not touch UI state, then call them from main-actor code and only marshal results back to the UI on the main actor.
