*Welcome to this series of articles in which we'll examine the details of the new features in C# 6.0. You are currently reading part 1 of 11:*


1. <em><strong>Introduction</strong></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">Expression-Bodied Members</span></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">Auto-Implemented Property Initializers</span></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">Read-Only Auto-Implemented Properties</span></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">Null-Conditional Operator</span></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">NameOf Operator</span></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">String Interpolation</span></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">Await in Catch and Finally Blocks</span></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">Exception Filters</span></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">Index Initializers</span></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">Extension Add in Collection Initializers</span></em>
- <em><span title="Coming soon..." style="cursor:not-allowed;">Using Static Statement</span></em>

*In addition to these articles, I am also producing a series of free screencasts about C# 6.0 on my YouTube channel, [CodeCast](https://www.youtube.com/channel/UCcQsDUZiK1GWDcP7BpVO_kw?sub_confirmation=1). If you want to, you can check them out [here](https://www.youtube.com/playlist?list=PL5ze0DjYv5DYK391_xP1Y8Oait1-_o5x9).*



### Introduction

Conventionally, each new version of C# has one super-feature, and a handful of miniature ones. For example, in the previous version of C#, C# 5.0, the super-feature was [asynchronous methods (`async`)](https://msdn.microsoft.com/en-us/library/hh156513.aspx) and the miniature features were [caller information attributes](https://msdn.microsoft.com/en-us/library/hh534540.aspx) and [improved "closures"](https://stackoverflow.com/questions/12112881/has-foreachs-use-of-variables-been-changed-in-c-sharp-5). C# 6.0 is a bit different, in that there is no super-feature, just a bunch of miniature ones.

Instead of introducing a new super-feature, C# 6.0 introduces a bunch of small features that alleviate small pain points from previous versions of the language. Most of the new features in C# 6.0 will make your code simpler, leaner, more readable, and more pleasant to write, but they are unlikely to revolutionize the way you write C# like super-features (such as asynchronous methods) in earlier versions of the language have.

As you'll see throughout this series, most of the new features are essentially [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar) – in other words, most (but not all) of the features in C# 6.0 do not introduce new functionality, but instead enable you to do things you could already do in a more compact and readable manner.

There are 11 new features in total. Here's a quick rundown of the features (in the order we'll examine them), so you know what to expect:

- **Expression-Bodied Members -** Allow you to associate a single expression with a member instead of a block (much like a lambda expression, but for members).
- **Auto-Implemented Property Initializers –** Allow you to initialize automatic properties with a default value, inline, much like a field initializer.
- **Read-Only Auto-Implemented Properties -** Allow you to omit the `set`ter from an automatic property, which makes it read-only.
- **Null-Conditional Operator –** A much more compact syntax for deep null checks.
- **NameOf Operator –** Allows you to refer to an identifier (such as an argument name) in a resilient way.
- **String Interpolation -** Language support for [*composite formatting*](https://msdn.microsoft.com/en-us/library/txafckwd.aspx).
- **Await in Catch and Finally Blocks –** It is now possible to use `await` in catch and finally blocks.
- **Exception Filters -** Allow you to conditionally enter exception handlers.
- **Index Initializers –** Allow you to elegantly add elements to an indexed collection via an index.
- **Extension Add in Collection Initializers –** Allow you to add elements to a collection based on an extension method called `Add` (previously, the `Add` method had to be part of the collection type.)
- **Using Static Statement –** Removes the need to fully qualify public static members.


While these features are unlikely to revolutionize the way you write C#, they _will_ change the way you write code in specific scenarios, due to the fact that they are so much more efficient, you'll likely forget there was another way to code them.

You might be wondering why there is no super-feature this time around. Presumably, the reason is that the managed languages team were busy completing a project they first announced around 6 years ago - the [_.NET Compiler Platform_](), better known by it's code name, _Roslyn_.

### Roslyn

Historically, the C# compiler was a [black box](https://en.wikipedia.org/wiki/Black_box) - you, the developer, input C# code and, if it was valid, the compiler would emit an assembly (a .exe or .dll), but the implementation of the compiler, however, was opaque:

![](https://i.imgur.com/HC1YO73.png)

We could never have known *exactly* what the compiler was doing internally – it was opaque - but it was safe to assume that it was doing normal compiler things: parsing source code, analysing the semantics, emitting IL, etc.:

![](https://github.com/dotnet/roslyn/wiki/images/compiler-pipeline.png)


As the source code traversed this pipeline, the compiler would develop a wealth of information about it - information about what elements are present in the code, how they relate to each other, etc. - this information, however, was private. This was unfortunate because, that information could have been of *tremendous* value to developers outside of the managed languages team. Think about tools like [SharpDevelop](), who require a deep understanding of source code - they had to rely on their own implementations to parse and analyse source code to power their features (such as syntax colorization and completion lists).  Even Visual Studio had three separate makeshift language services that lacked feature parity!

It would have been *so* much nicer if the compiler was 1/ open source and 2/ exposed a set of compiler APIs. As you may have guessed, now the compiler does both, through Roslyn.

Roslyn is a reimagination of what a compiler should be. In a nutshell, it is a [open source](), ground-up rewrite of the Visual Basic and C# compilers in those languages themselves. In addition to providing a set of familiar compilers (csc.exe and vbc.exe), it provides a set of compiler APIs that expose rich information about the source code:

![](https://github.com/dotnet/roslyn/wiki/images/compiler-pipeline-api.png)

As you can see in the above illustration (which I pinched from the [Roslyn documentation](https://github.com/dotnet/roslyn/wiki/Roslyn%20Overview)), each phase in the compiler pipeline has a corresponding compiler API.

This "openness" is obviously _very_ beneficial for tool developers - tools like SharpDevelop and Visual Studio can now use Roslyn to power their features instead of relying on their own makeshift implementations, which will make them more powerful.

The benefits of Roslyn are not limited to traditional tools like Visual Studio. Hot open-source tools like [Omnisharp](http://www.omnisharp.net/) and  [scriptcs](http://scriptcs.net/) are now powered by Roslyn. So is [dynamic development](http://weblogs.asp.net/scottgu/introducing-asp-net-5) in [ASP.NET 5](http://www.asp.net/vnext). Roslyn will also make it easier to embed C# in [domain-specific languages](), build [static analyzers](https://en.wikipedia.org/wiki/Static_program_analysis), and more! It really opens the door to a whole new world of [meta-programming](https://en.wikipedia.org/wiki/Metaprogramming) possibilities.

As you can probably appreciate, porting the current compilers' code to managed code was no small undertaking. *Presumably*, it is because the managed languages team was focused on completing Roslyn that there wasn't much time for language feature innovation this time around. That being said, Roslyn will make it easier to prototype and implement new features, so we'll likely see a quicker turn-around for language innovation going forward.

> #### Why the name Roslyn?
> A small piece of trivia is that the code name Roslyn was inspired by the name of a small town in Washington called [Roslyn](https://en.wikipedia.org/wiki/Roslyn,_Washington), which is about an hours drive from the Microsoft campus in Seattle.

I shan't belabour Roslyn in this article because, while Roslyn is a very interesting topic, understanding it will not particularly help your understanding of C# 6.0, which is the focus of these articles.

We're nearly ready to dive into C# 6.0, but before we do, I want to briefly explain a couple of technical details relating to the differences between the compiler, the runtime, and the framework libraries. If these details are familiar to you, I apologize - you should feel free to skip to the conclusion - but these topics have been the cause of a lot of confusion in the past.

### Dissecting the .NET Framework

It is important to understand that the C# compiler, while part of the .NET Framework, is versioned separately to the runtime and framework libraries.

![](https://upload.wikimedia.org/wikipedia/commons/a/af/Common_Language_Runtime_diagram.svg)

The *C# compiler* (as illustrated above in a diagram I pinched from Wikipedia) is a tool that translates C# code into bytecode known as [*MSIL*](https://en.wikipedia.org/wiki/MSIL).

The *runtime* manages the execution of bytecode. It recognizes a set of [IL instructions](https://en.wikipedia.org/wiki/List_of_CIL_instructions) and translates them into machine-specific assembly instructions on the fly.

The _framework libraries_ are essentially the libraries you reference from your projects to help you kick-start development:

![](https://i.imgur.com/DBM3PS8.png)

They encompass primitive types like `Int32` and `String` as well as complex types like `HttpClient` etc.

Understand that the new features in C# 6.0 stem from changes the compiler *only*. **You do *not* need to target the latest framework version to use C# 6.0 features.** This is because C# 6.0 is not dependent on new IL instructions, nor is it dependent on new types in the framework libraries (with one exception for *interpolated strings*, but we'll cross that bridge when we come to it.).

One more thing to note before we dive in is that, if you want to use C# 6.0 (and of course, you do), you’ll need to be using Visual Studio 2015. If you are yet to install Visual Studio 2015, you can head over to the [Visual Studio website](https://www.visualstudio.com/en-us/products/vs-2015-product-editions.aspx), and download the community edition for free.

### Conclusion

In this article I set the tone for C# 6.0. Now that you have a sound idea about what to expect, let's start by examining the first new feature: [expression-bodied members]().

**P.S. If you read this far, you might want to follow me on [Twitter](https://twitter.com/bookercodes) and [GitHub](https://github.com/alexbooker), or [subscribe](https://booker.codes/rss/) to my blog.**
