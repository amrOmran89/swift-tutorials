---
title: "Understand some keyword in SwiftUI"
date: 2024-03-10
layout: post
categories: [Swift, Generics, Protocols]
---

Protocols with associated types (PATs) are powerful but historically awkward to use directly as return types or stored properties. The `some` keyword (opaque result types) fixes many of these pain points by letting you return “a concrete type that conforms to a protocol” while keeping the concrete type hidden from the caller.

## The problem: existentials don’t work with PATs

When a protocol has `Self` or `associatedtype` requirements, you cannot use it directly as a type unless you add more information:

{% highlight swift %}
protocol Repository {
    associatedtype Model
    func getAll() async throws -> [Model]
}

// Won't compile:
// error: protocol 'Repository' can only be used as a generic constraint because it has Self or associated type requirements
func makeRepo() -> Repository { /* ... */ }
{% endhighlight %}

The compiler needs to know which concrete `Model` is used. Before opaque types, you’d be forced into type erasure or complicated generics.

## The fix: return an opaque type with `some`

With `some`, you promise to return one specific concrete type that conforms to the protocol, but you don’t reveal the type name to the caller.

{% highlight swift %}
struct User { let id: Int }

struct UsersRepo: Repository {
    func getAll() async throws -> [User] { /* fetch */ [] }
}

// Works: the concrete return type is UsersRepo, but callers only see "some Repository".
func makeRepo() -> some Repository { UsersRepo() }

// Callers know that whatever comes back is a Repository with a stable, single concrete type.
let repo = makeRepo()
let users = try await repo.getAll()
{% endhighlight %}

Key properties of `some`:

- **Opaque, not dynamic**: The concrete type is fixed per function declaration. Different code paths in the same function must return the same underlying type.
- **Type identity preserved**: The compiler can still specialize and optimize as if it knew the concrete type.
- **Great with PATs**: You don’t have to expose the associated types to callers.

### One type per function

All return paths must resolve to the same concrete type when you use `some`:

{% highlight swift %}
func makeNumberRepo(useCache: Bool) -> some Repository {
    if useCache { return UsersRepo() }
    // error: return types 'UsersRepo' and 'OrdersRepo' do not match
    // return OrdersRepo()
    return UsersRepo()
}
{% endhighlight %}

## SwiftUI made this mainstream

SwiftUI’s `View` is a protocol with associated types. The famous signature uses `some`:

{% highlight swift %}
struct ContentView: View {
    var body: some View {
        Text("Hello")
    }
}
{% endhighlight %}

`some View` lets you return a concrete view tree without exposing its exact type.

## When you need heterogeneity: `any` or type erasure

Opaque types solve “return a single hidden concrete type.” If you need to store or pass around values of **different** conforming types in one container, use `any Protocol` (an existential) or type erasure.

### Using `any` with primary associated types

Primary associated types allow you to constrain existentials more ergonomically:

{% highlight swift %}
protocol CollectionLike<Element> { // imagine a simplified Collection
    associatedtype Element
    var count: Int { get }
}

// With primary associated types, you can write:
func logCounts(_ collections: [any CollectionLike<Int>]) {
    for c in collections { print(c.count) }
}
{% endhighlight %}

### Type-erased wrappers

If a protocol can’t be easily used as an existential (due to multiple associated types or complex constraints), a type-erased box still helps:

{% highlight swift %}
struct AnyRepository<Model>: Repository {
    private let _getAll: () async throws -> [Model]
    init<R: Repository>(_ repo: R) where R.Model == Model {
        _getAll = repo.getAll
    }
    func getAll() async throws -> [Model] { try await _getAll() }
}

let mixed: [AnyRepository<User>] = [AnyRepository(UsersRepo())]
{% endhighlight %}

## `some` vs `any`: quick guidance

- **Use `some Protocol`** when you return or expose exactly one hidden concrete type per declaration and want compile-time specialization.
- **Use `any Protocol`** when you need to store or pass around heterogeneous values conforming to a protocol.
- If neither fits due to complex associated types, consider **type erasure**.

## Bonus: opaque parameters

You can also use `some` in parameter position (opaque parameters) to express “any type that conforms to P, chosen by the caller, but the callee treats it generically.”

{% highlight swift %}
protocol Renderer { associatedtype Output; func render() -> Output }

func display(_ r: some Renderer) {
    // `r` has a concrete Renderer type picked by the caller.
    _ = r.render()
}
{% endhighlight %}

By combining `some`, `any`, primary associated types, and (where needed) type erasure, Swift makes protocols with associated types practical and ergonomic in everyday code.
