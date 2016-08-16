---
layout: post
title: "Protocol Oriented Overengineering (Swift)"
date: 2016-08-16 00:00:00
draft: false
---

Last weekend I built a simple [Mac app](https://github.com/wokalski/Hourglass) for myself. It consists of one table view with two types of cells. To give you an idea - it is so simple that I didn't get angry at `AppKit` even once.

I used Swift 3 as the language of choice. I've been using `JavaScript` (sigh, it's another story to tell) on daily basis for over 2 months now. I was glad to come back to something I had hoped to be normal and awesome. Yet, from the retrospect I don't think I was efficient. Why? How would I solve it? Read on!

## How to Swift

A conversation of a developer and Swift:

>Developer: "Hi, how do I"
>
>Swift: "Protocols!"
>

Ten minutes later...

>Developer: "Ok, so how do I use a protocol here?"
>
>Swift: "This protocol has `associatedtype` or `Self` requirement, you dummy!"


I had two attempts to use protocols. Yet, I ended up not using them. At the beginning, I had technical difficulties[^1]. Then, I realized that a protocol would be a redundant abstraction.

## Why do we create abstractions?

All decisions during software development are about saving time. The meaning of time differs. Sometimes you want to put a prototype together. Other times you maintain a mature product. 

The same applies to abstractions. We create them when we think, we can save time.

### Framework vs end-user product

Frameworks and libraries are abstractions. Generally, their public interface is as reusable as possible.

An application is the opposite. It performs very domain specific tasks. The code is as domain specifc as possible too. It's because, when functionality changes, the underlying code has to change too.

Developing generic abstractions is costly. In end-user products we develop them when we think it will save time in a longer period.

For instance, imagine you developed a library to process bitmaps. Suddenly your application switches to vector images. You need a solution to a problem which has a completely different domain. 

Now you have an awesome, tremendous piece of code you no longer need. It depends on a case whether it was worth it to develop it or not. Unfortunately, I had developed many stupid libraries I never used again. 

Another, even more popular case, is developing an abstraction and when requirements change you create workarounds. "_I predicted all the scenarios EXCEPT FOR THIS ONE_". You work around your own application specific code to make progress. Sounds ridiculous, happens every time I develop something for a longer period.

[_A better, more in depth, smarter elaboration on this topic_ (YouTube)](https://www.youtube.com/watch?v=mVVNJKv9esE)

So this weekend, when I was writing the app I wanted to develop generic abstractions - protocols while writing very application specific code. The thing I just said not to do. I did it subconsciously. Swift makes it feel right.

### The elegant trap

Have you ever heard the phrase "this looks (feels) good"? It's the elegant trap. "This fits together". Elegant trap. I say it (or have this thought) every single work day. 

Code _often_ looks nice. It's especially true about abstractions. But then it's very easy to be bitten by your own cleverness. The elegant trap is why I wanted to create a protocol.

### The least powerful tool

In both cases I wanted to express that an instance might be of type 'A' or 'B'. I didn't need further polymorphism. 

Since the answer to everything in Swift is a protocol, I tried to implement one. It turned out to be impossible but I realized how stupid it was. I spent a lot of energy trying to define a generic protocol I would never use again. I did it all, to express that an instance might be of one of two types.

This is a very common scenario. Swift is very precise and verbose when it comes to types. You can get a lot of power with protocols but then you lose expressiveness. In this case, a protocol was redundant overhead.

It is always possible to do better. Except for usual complaining I would like to present a hypothetical solution to this particular problem.

## Union Types (or Sum Types, [you name it](https://en.wikipedia.org/wiki/Tagged_union))

As part of the criticism, I would like to show how nice things can be. This construct is a concise and less abstract alternative to many usecases of protocols.

What are they?

``` swift
// Pseudo code is worth a thousand words.
type Foo = String | Int | Double

func bar(foo: Foo) {
    switch foo {
    case Int: ...
    case String: ...
    case Double: ...
    }
}
```

You can think of them as lightweight enums with one significant difference.

``` swift
// You can compose them and they are always flattened

type Foo = String | Int | Double
type Bar = Date | Char
type FooBar = Foo | Bar // String | Int | Double | Date | Char
```

The definition of 'bar' simply means that this instance might be either of type 'Date' or 'Char'.

``` swift
// You can also use them "in place"

let cell: UITableViewCell | UICollectionViewCell
```

You could do crazy stuff with functions, making expressive more functional style possible.

``` swift
func combine<T, U, R>(_ f: T -> R, _ g: U -> R) -> (T | U) -> R

func cell(string: String): UITableViewCell
func cell(int: Int): UITableViewCell

let createCell = combine(cell(string:), cell(int:))

// createCell takes a String or Int and returns a UITableViewCell.
```

It is probably obvious at this point; this is not my idea. Union types exist in other languages, like `Racket`, `Haskell`, `TypeScript`, and [others](https://en.wikipedia.org/wiki/Tagged_union).

## Right tool for a job

Union Types are not an abstraction. These two concepts serve different purpose and should be used together.

| **Protocols**                                                       | **Union Types**                                                      |
|---------------------------------------------------------------------|----------------------------------------------------------------------|
| Define a generic interface and behaviors for conforming instances   | A way of saying that instance is of one of a _fixed_ amount of types |
| Best for public library/framework reusable abstractions | Most powerful in ever changing, application specific code        |


## Conclusion

I don't think Swift is a good tool for my job at the moment. In particular, I doubt it's possible to create one language to rule them all. Is it possible to compromise lightning fast performance with neat language features greedy developers like me want?

Swift is already complicated. When things evolve, they tend to grow. Looking at its trajectory it will just get more keywords and secret powers which are not necessarily something I want.

A possible solution would be some subsets of the language. They would be tailored for specific use cases. I guess we are nowhere near this point though.

You can talk to me on [Twitter](https://twitter.com/wokalski).

_Thanks to Maarten Schumacher and [Arek Holko](http://holko.pl) for the feedback on drafts._

PS: I am not going to write a proposal for the feature. [^6]

---

[^1]: - I didn't know how to extend a generic type to make it conform to a protocol.
      - An instance conforming to a protocol with `associatedtype` couldn't be returned. It can only be used as a generic constraint and the compiler could not infer the generic type. This is a serious limitation now but I expect it to be solved in future versions.

[^6]: Since some people might ask this very question let me tell you:
      
      1. First and foremost. **Swift is too complex** and this would make it even more complex. Ideally languages encourage consistent style. Too many features make it impossible.

      2. Swift is an extremely ambitious project. Writing a 'performant' *and* 'flexible' *and* 'multi-paradigm' *and* 'innovative' *and* 'easy to learn' *and* 'simple to understand' language is a Chris Lattner's kind of thing. There is a lot of stuff to consider. At the end of the day there might be a better future and I'm just holding it wrong.

      3. I am not qualified to argue (and I don't feel like arguing in the internet)

      THAT SAID, I am open to discuss it and contribute my own observations to a proposal of somebody else.

