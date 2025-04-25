---
title: Polly Bulkhead
draft: false
date: 2025-04-25
tags:
  - Coding
  - Learning
  - CSharp
---
source: [https://github.com/App-vNext/Polly](https://github.com/App-vNext/Polly)

## Using Polly and Bulkhead to manage number of parallel Threads

This sample will start multiple threads in parallel but only allow a set number (2) to run in parallel and a fixed number (10) of queue slots for newly created threads to wait until one in the pool becomes available.

### Minimal Sample

```csharp
namespace Polly_Bulkhead_Test
{
	using System.Threading.Tasks;
	using Polly;
	using Polly.Bulkhead;
	using System;
	using System.Collections.Generic;

	public class Program
	{
		static async Task Main()
		{
			var pollyBulkheadRunner = new PollyBulkheadRunner();
			await pollyBulkheadRunner.RunTask();
		}
	}

	public class PollyBulkheadRunner
	{
		private readonly AsyncBulkheadPolicy _bulkhead = Policy.BulkheadAsync(2, 10); public async Task RunTask()
		{
			var iterations = 10;
			var tasks = new List<Task>(); for (var i = 0; i < iterations; i++)
			{
				var task = _bulkhead.ExecuteAsync(async () => {
					await DoSomethingAsync(_bulkhead.BulkheadAvailableCount, _bulkhead.QueueAvailableCount);
				});
				tasks.Add(task);
			}
			await Task.WhenAll(tasks);
		}
		
		private async Task DoSomethingAsync(int bulkheadTasksAvailable, int bulkheadQueueAvailable)
		{
			await Task.Delay(2000);
			Console.WriteLine($"Task ended at {DateTime.Now}.\t Available Tasks: {bulkheadTasksAvailable}.\t Available Queue: {bulkheadQueueAvailable}.");
		}
	}
}
```
