# Loom

## Loom的简介

Project Loom 是一个在 Java 虚拟机 (JVM) 上构建的，用于协作式并发的新特性。它旨在提供轻量级的线程（VirtualThreads）和轻量级的进程（Fibers），以简化异步编程。

## Loom的历史

Loom 项目始于 2017 年，最初由 Ron Pressler 和 Alan Bateman 提出，并在 OpenJDK 社区中受到广泛讨论。Loom 是 OpenJDK 社区中的一个开源项目，由 Oracle 公司开发和维护。

是的，Loom 是在 JDK 14 中引入的实验性特性。JDK 14 于 2020 年 3 月发布，其中引入了 Project Loom 的初始版本，包括 Continuations 和 Virtual Threads 的实现。这些特性都是以实验性质的形式出现，需要在启用了特定的命令行选项时才能使用。

## Loom的实现方式

### Continuations

#### Continuations的简介

Continuation 是一种协作式的控制流程。在传统的调用-返回模型中，函数的控制权是在函数被调用时传递给函数体的。而在 Continuation 中，控制权可以在函数调用之间传递。这意味着一个 Continuation 可以挂起执行，并在稍后恢复它的执行。

#### Continuations的原理

在 Java 中，Continuation 通过 Java 14 引入的协程实现。协程是一种轻量级的线程，它在执行上下文之间切换，而不是像传统的线程那样在操作系统级别上切换。协程使用 Continuation 实现异步操作，以避免线程上下文切换的开销。

补充：

首先，Continuation 保存了当前线程的堆栈信息，包括程序计数器和本地变量等，使得在恢复 Continuation 时可以从上一次挂起的位置继续执行程序。同时，Continuation 还保存了一个 continuation prompt，它可以用来控制程序的执行流程，例如在需要进行异常处理或跳出当前循环时，可以使用 continuation prompt 来控制程序的执行。

其次，Continuation 可以嵌套使用，即可以在一个 Continuation 中保存另一个 Continuation，从而形成一个 Continuation 树。这种嵌套的 Continuation 树可以用来保存整个程序的执行状态，从而实现程序的协程化。

最后，Continuation 的实现方式是基于栈的，每个 Continuation 都包含了一份完整的栈信息，从而使得在 Continuation 之间切换时不需要进行额外的栈操作，从而提高了 Continuation 的执行效率。

总之，Continuation 是一种可以保存程序执行状态的机制，是 Loom 中实现 Fiber 和 VirtualThread 的基础。在使用 Continuation 时需要注意控制程序的执行流程，并且可以利用 Continuation 的嵌套和栈实现协程化的程序设计。

### VirtualThreads

#### VirtualThreads的简介

VirtualThreads 是一种轻量级线程，它可以在单个操作系统线程上执行。在传统的线程模型中，每个线程都有一个操作系统线程与之关联，这会导致线程的创建和销毁开销很大。VirtualThreads 可以在不增加线程数量的情况下创建和销毁，从而减少了线程管理的开销。

#### VirtualThreads的原理

VirtualThreads 的实现方式是基于 Continuation 实现的。在一个 VirtualThread 内部，多个 Continuation 可以在同一个线程上执行，而不会产生线程切换的开销。

#### VirtualThreads和系统级线程的区别

VirtualThreads 和传统的系统级线程有很大的区别。传统的线程是由操作系统管理的，它们需要占用操作系统的资源，并且线程的创建和销毁都需要操作系统的介入。而 VirtualThreads 是由 JVM 管理的，它们不需要占用操作系统的资源，并且可以在不增加线程数量的情况下创建和销毁。

### Fibers

#### Fibers的简介

Fiber 是一种轻量级的协程，它在执行上下文之间切换，而不是在操作系统级别上切换。它可以用于实现异步操作，并且可以在不增加线程数量的情况下创建和销毁。

#### Fibers的实现原理

Fiber 的实现方式是基于 Continuation 实现的。一个 Fiber 包含多个 Continuation，它们可以在同一个线程上执行。Fiber 可以挂起自己的执行，等待其他 Fiber 完成操作，然后再恢复执行。

## Loom和其他语言的类似协程概念的对比

### 和GoRoutine的区别

Go 语言中的协程（GoRoutine）是由 Go 运行时环境（Go runtime）管理的。GoRoutine 可以使用关键字 go 创建，并且可以与操作系统级线程一起工作。GoRoutine 是一种非常轻量级的线程，可以在 Go 程序中创建数千个协程。

和 GoRoutine 相比，Loom 中的 VirtualThreads 和 Fibers 是更轻量级的线程，它们不需要操作系统级别的线程支持，并且可以在不增加线程数量的情况下创建和销毁。

### 和CoRoutine的区别

Coroutine 是一种由 Lua 语言引入的协程实现。Coroutine 与 Loom 中的 Fiber 有一些相似之处，它们都是基于 Continuation 实现的协程，可以在一个线程中执行多个协程。

和 Coroutine 相比，Loom 中的 VirtualThreads 更加灵活，因为它们可以在单个操作系统线程上执行，而不需要创建额外的线程。而 Coroutine 则需要在多个操作系统线程之间切换执行。

## 总结

Loom 是 Java 语言中一个重要的新特性，它通过 Continuation 实现了一种轻量级的线程模型。Loom 中的 VirtualThreads 和 Fibers 可以在不增加线程数量的情况下创建和销毁，这使得 Java 程序可以更高效地利用计算机资源。

相比于 GoRoutine 和 Coroutine，Loom 中的 VirtualThreads 和 Fibers 更加轻量级，可以在单个操作系统线程上执行，从而减少了线程切换的开销，并且更加灵活。Loom 的推出将进一步提升 Java 语言在异步编程方面的能力，并且有望成为 Java 程序员的一个必备技能。

