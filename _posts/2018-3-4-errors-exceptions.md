---
layout: post
title: Errors, Exceptions, and whatnot
categories: [FSharp, functional programming, domain modeling]
fullview: true
comments: true
---
Functional programming and domain modeling are some of my favorite subjects when it comes to software development.
In the last few years, I've been combining them to design and write code that is easy to understand and one of the aspects that can be tricky is how to communicate errors and failures.

The way I like to think about error handling is based on [Type Driven Design](https://www.tikalk.com/posts/2011/11/29/type-driven-design/) although without most of its power and formalism when applied to common OO languages.

>The thoughtful use of types can make a design more transparent and improve correctness at the same time. - Scott Wlaschin

As an example, I want to define one function that saves one payment identifier object.

### Naive(typeless) contract 
```csharp
void Save(PaymentIdentifier value);
```
Analyzing the type signature the only thing I can tell is that I must give an instance of PaymentIdentifier.

Cristal clear right? well, not really.

The first thing I would question before calling this function is if *this function call fails and which errors must be caught*.
This information could be somewhere else in comments, tests or more usually I would need to dive into the function and its dependencies to be sure that my invocation is safe.

### Result type
```csharp
Result Save(PaymentIdentifier value);
```
Now we have one return for this function invocation, let's check a dummy implementation for this result type.
```csharp
    public class Result<T> where T : class
    {
        public Error Error { get; }
        public bool Successful { get; }
        public T Value { get; }

        private Result(T value, bool successful = true)
        {
            Value = value;
            Successful = successful;
        }
        private Result(Error errorCode)
        {
            Error = errorCode;
            Successful = false;
        }
        public static Result<T> Success(T value) => new Result<T>(value);

        public static Result<T> Failure(Error error) => new Result<T>(error);
        public static Result<T> Failure(ErrorCode errorCode, [CallerMemberName] string callerName = "", [CallerFilePath] string filePath = null,
            [CallerLineNumber] int lineNumber = 0) => new Result<T>(new Error(errorCode, callerName: callerName, filePath: filePath, lineNumber: lineNumber));

    }
```
The important bits of the previous code is the representation of error and success. **the type system is informing result checking is required.

#### Advantages
* Clear identification of functions that can fail
* Control over failures that can be expected(IO, side effects)
* The Error can carry more information, therefore, improve logging and troubleshooting

### What about the exceptions, they are designed for it, right?
Yes, but they are supposed to _happen_ sometimes and not be part of my design. When exceptions are the selected mechanism for error one needs to know every invoked function in the stack in order to catch it properly.
What it smells like?
>By keeping data private and providing public well-defined service methods the role of the object becomes clear to other objects. This increases usability. Other objects are aware of the format to send messages into the object providing the public service. This is essentially a contract between the two objects. The invoker is agreeing to send the message in the specific format, including passing any of the required parameter information. The invoked object is agreeing to process the message and if necessary, return a value in the pre-specified format.

Yes, lack of encapsulation! The function has a clear definition *only * the happy path.

### Distributed software failures are part of the flow, they should be considered in the design
Failures will happen and it must be taken into account. The use of Result-like types helps to think about possible/common failures and also provides a typed way to take decisions when they happen.

### Result-like patterns are getting more popular
When writing this post I discovered that [Rust has a native support for Result type](https://doc.rust-lang.org/std/result) and encourage its use.
F# always had Option types but [_now_](https://blogs.msdn.microsoft.com/dotnet/2017/03/07/announcing-f-4-1-and-the-visual-f-tools-for-visual-studio-2017-2/) has the Result type for better semantics.

### How to give it a try?
If your language doesn't support [option type](https://en.wikipedia.org/wiki/Option_type) you can implement a simple result type or look for packages.
In C#, for example, there is the [https://github.com/louthy/language-ext](https://github.com/louthy/language-ext).