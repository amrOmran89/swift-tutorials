---
title: "Async/Await in Swift"
date: 2025-09-15 12:00:00 +0200
layout: post
categories:
  - Swift
  - Concurrency
tags:
  - swift
  - concurrency
  - asyncawait
---

Swift 5.5 introduced **async/await** for structured concurrency.

{% highlight swift %}
func fetchUser() async throws -> User {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}
{% endhighlight %}