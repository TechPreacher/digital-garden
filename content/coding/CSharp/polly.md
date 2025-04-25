---
title: Polly
draft: false
date: 2025-04-25
tags:
  - Coding
  - Learning
  - CSharp
---
source: [https://github.com/App-vNext/Polly](https://github.com/App-vNext/Polly)

## Using Polly for Exponential Backdown

Polly is a .NET resilience and transient-fault-handling library that allows developers to express policies such as Retry, Circuit Breaker, Timeout, Bulkhead Isolation, and Fallback in a fluent and thread-safe manner.

### Minimal Code Sample

```csharp
// Single exception type
Policy.Handle<HttpRequestException>(); // Retry once

Policy
	.Handle<SomeExceptionType>()
	.Retry(); // Retry, waiting a specified duration between each retry.

// (The wait is imposed on catching the failure, before making the next try.)

Policy
	.Handle<SomeExceptionType>()
	.WaitAndRetry(new[]
	{
		TimeSpan.FromSeconds(1),
		TimeSpan.FromSeconds(2),
		TimeSpan.FromSeconds(3)
	});   // Execute an action

var policy = Policy
	.Handle<SomeExceptionType>()
	.Retry();

policy.Execute(() => DoSomething());
```

