---
title: "Processing Pipelines (Dataflow) - Introduction"
date: 2018-09-09T16:33:49-03:00
draft: false
tags: ["dataflow", "pipeline"]
---

Hello Guys, recently I changed a job going to a new challenge into [Take](https://take.net/). From this, I started to study some parts of the stack that I was not familiar. One of these is Dataflow. This is the start of my summary mostly provided by the study of the documentation and some coding.

# Processing Pipelines (Dataflow) - Introduction

The library Dataflow provides several functions to intensive processing, helping the developers with concurrency-enabled applications. When you develop some logic using the concept proposed by the dataflow called "pipelines", basically you are creating steps of processing called "blocks".

The library Dataflow is available like one [package NUGET](https://www.nuget.org/packages/System.Threading.Tasks.Dataflow/). It abstracts away most of the work needed when creating asynchronous and parallel processing code.

## Types of blocks on Dataflow

The library Dataflow is structured in one concept called Blocks, which are data structures that buffer and process data. There are three types of a block on Dataflow: 

1- `Source Blocks` - Block that is the source of data and can be read by others components

2- `Target Blocks` - Block that is a receiver of data and can be written by other components

3- `Propagator Blocks` - Block that is both `source` and `target`. 

This blocks can be connected forming one ``pipeline``, which is a form of network, in that way data can be processed through the pipe propagating synchronously or asynchronously to targets as that data becomes available. The method `LinkTo` presented on the source blocks and propagator, links the blocks, with this, you can connect to one or more targets, and provide a filter for each link (optional). This is useful for example when you want just the items who has value to continue on the pipeline.

These types are divided into three categories: *buffering blocks*, *execution blocks* and *grouping blocks*. 

### Buffering Blocks

Can be used to hold data. There are three blocks of this category: [BufferBlock<T>](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.bufferblock-1),  [BroadcastBlock<T>](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.broadcastblock-1) and [WriteOnceBlock<T>](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.writeonceblock-1). 

#### BufferBlock<T>

This block store a queue(FIFO) of messages that can be written to by multiple sources and read by multiple targets. When a target receives a message from this block, that message is removed from the queue. In that way, each message only can be read by a single target. The use case for the BufferBlock is when you want to pass multiple messages to another part of the software and it must receive each message.

#### BroadcastBlock<T>

The use case for the BroadcastBlock is when you want to pass multiple messages to another part of the software, but it needs only the most recent value and when you want to broadcast a message to multiple components.

#### WriteOnceBlock<T>

This block becomes immutable after it receives a value, and when one target receives a message from this block, that message is not removed from that object, sending a message to multiple targets. Therefore after the block receives a message, it discards subsequent messages. The use case for the WriteOnceBlock is when you want to propagate only the first of multiple messages.

### Configuring the Buffering Blocks

Providing a [DataflowBlockOptions](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.dataflowblockoptions) to the constructor of the blocks can configure some behavior on your pipeline. 

- BoundedCapacity - The maximum number of messages that can be buffered by the block
- EnsureOrdered - No need to explain
- MaxMessagesPerTask - The maximum number of messages that can be processed per task
- NameFormat - The format string to use when a block is queried for its name
- TaskScheduler - [TaskScheduler](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.taskscheduler?view=netcore-2.1) to use for scheduling tasks. 
- CancellationToken - monitor cancellation requests

### Execution Blocks

Can be used to execute a delegate user-provided for each received data. In all execution blocks, he receives a delegate and the processing of each input element is considered completed when the delegate returns. There are three blocks of this category: [ActionBlock<T>](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.actionblock-1), [TransformBlock<T>](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.transformblock-2) and [TransformManyBlock<T>](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.transformmanyblock-2)

#### ActionBlock<T>

The ActionBlock is a target block that executes a delegate when it receives data. This can be used for synchronous and asynchronous processing of each message. After this, you wait until the block processing is completed. Basically, he will start a buffer of items, start the task to sequentially or parallelly process (can be configured) and complete the task when the buffer is empty.
The use case for the ActionBlock is when you want to process messages.

#### TransformBlock<TInput, TOutput>

This block resembles the ActionBlock, the difference is that he acts as both source and target (propagator block). You will pass one delegate that will receive the input `TInput` and return the `TOutput`. So, this block basically will receive one input `TInput` and will pass the `TOutput` to the next block of the pipeline.

#### TransformManyBlock<TInput, TOutput>

This block resembles the TransformBlock, the only is change is that he produces zero or more output values for each input.

### Configuring the Execution Blocks

Providing an [ExecutionDataflowBlockOptions](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.executiondataflowblockoptions?view=netcore-2.1) to the constructor of the execution blocks who inherits DataflowBlockOptions. Providing two more options for configuration:

- MaxDegreeOfParallelism - The maximum number of messages that may be processed per task. Per default is 1, when the value is higher than 1 the messages will be processed concurrently.
- SingleProducerConstrained    - Get if the code is constrained to one producer at a time

### Grouping Blocks

There are three blocks of this category:  [BatchBlock<T>](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.batchblock-1), [JoinBlock<T1,T2>](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.joinblock-2), and [BatchedJoinBlock<T1,T2>](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.batchedjoinblock-2). There is two modes for grouping blocks, ``Greedy`` and ``NonGreedy``, the greedy mode is default when he receives the specified number of elements he return the array, the non greedy mode postpones messages until enough sources offer messages to the block to form a batch.

#### BatchBlock<T>

This block combines sets of input data, known as batches, into arrays of output data. When received one specific number of elements, it propagates an array that contains those elements. 

#### JoinBlock<T1, T2...>

This block collects input element and propagates Tuple with the types defined `Tuple<T1, T2...>`. 

#### BatchedJoinBlock<T1, T2...>

This block collect batched of input elements and propagate Tuples with the lists of the types defined `Tuple<IList<T1>, IList<T2>...>`.

### Configuring the Grouping Blocks

Providing a [GroupingDataflowBlockOptions](https://docs.microsoft.com/pt-br/dotnet/api/system.threading.tasks.dataflow.groupingdataflowblockoptions?view=netcore-2.1) to the constructor of the execution blocks who inherits DataflowBlockOptions. Providing two more options for configuration:

- Greedy - Boolean value to use to determine whether to greedily consume offered messages.
- MaxNumberOfGroups - The maximum number of groups that should be generated by the block.