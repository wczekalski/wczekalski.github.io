# No title yet

This weekend I built a simple Mac app for myself. It consists of one table view with two types of cells. To give you an idea - it is so simple that I didn't get angry at 'AppKit' even once.

I used Swift 3 as the language of choice. I've been using 'js' (sigh, it's another story to tell) on daily basis for over 2 months now. I was glad to come back to something I had hoped to be normal and awesome. Yet, from the retrospect I don't think I was efficient. Why? How would I solve it? Read on! 

## How to Swift

A conversation of Swift and a developer:

D: Hi, how do I

S: Protocols

Ten minutes later:

D: Ok, so how do I use a protocol here?

S: This protocol has 'associatedtype' or 'Self' requirement!


I had two attempts to use protocols. Yet, I ended up not using them. At the beginning, I had technical difficulties[1]. Then, I realized that a protocol would be a redundant abstraction.

## Why do we create abstractions? 

We write all these blog posts and develop libraries to make software development cheaper (AND GET ALL THE INTERNET POINTS). We do it to spend less time on repetitive work. Software development is not a goal in itself. 

Framework or library developers' product are flexible abstractions.

It is different for end user software developers. Our goal is to deliver an application which meets needs of a target group. We (should) use or create abstractions which are suitable to our needs. Otherwise, we waste time and resources. 

Ideally you want to keep your code as domain specific as possible while maintaining flexibility. The cost of creating, removing and replacing an abstraction should be minimized. Developing general purpose code is costly.

For instance imagine you developed a library to process bitmaps. Suddenly your application switches to vector images. You need a solution to a problem which has a completely different domain. 

Now you have an awesome, tremendous piece of code you no longer need. It depends on a case whether it was worth it to develop it or not. Unfortunately, I developed many stupid libraries I never used again. 

Another, even more popular case, is developing an abstraction and when requirements change you create workarounds. I predicted all the scenarios EXCEPT FOR THIS ONE. You work around your own application specific code to develop the application. Sounds ridiculous, happens every time I develop something for a longer period.

[_A better, more in depth, smarter elaboration on this topic_ (YouTube)](https://www.youtube.com/watch?v=mVVNJKv9esE)

So this weekend, when I was writing the app I wanted to develop generic abstractions - protocols while writing very application specific code. The thing I just said not to do. I did it subconsciously. Swift makes it feel right[2].

In both cases I wanted to express that an instance might be of type 'A' or 'B'. I didn't need further polymorphism. 

Since the answer to everything in Swift is a protocol, I tried to implement one. It turned out to be impossible[1] but I realized how stupid it was. I spent a lot of energy trying to define a generic protocol I would never use again. I did it all, to express that an instance might be of one of two types.

This is a very common scenario. Swift is very precise and verbose when it comes to types. You can get a lot of power with protocols but then you lose expressiveness. I would even say that here, it promotes an "antipattern"[6].

It is always possible to do better. Except for usual complaining I would like to present a hypothetical solution to this particular problem.

# Union Types (or Sum Types, you name it)[4]

As part of the criticism, I would like to show how nice things can be. This construct is a concise and less abstract alternative to many usecases of protocols.

What are they?

'''
// Pseudo code is worth thousand words.
type Foo = String | Int | Double

func bar(foo: Foo) {
switch foo {
case Int: ...
case String: ...
case Double: ...
}
}
'''

You can think of them as lightweight enums with one significant difference.

'''
// You can compose them and they are always flattened

type Foo = String | Int | Double
type Bar = Date | Char
type FooBar = Foo | Bar // String | Int | Double | Date | Char
'''

The definition of 'bar' simply means that this instance might be either of type 'Date' or 'Char'.

'''
// You can also use them "in place"

let cell: UITableViewCell | UICollectionViewItem
'''

You could do crazy stuff with functions, making expressive more functional style possible.
'''
func compose<T, U, R&gt;(_ f: T -&gt; R, _ g: U -&gt; R) -&gt; (T | U) -&gt; R

func cell(string: String) -&gt; UITableViewCell
func cell(int: Int) -&gt; UITableViewCell

let createCell = compose(cell(string:), cell(int:))
'''

It is probably obvious at this point; this is not my idea. Union types exist in other languages, like 'Rocket', 'Haskell' or 'TypeScript'.[5]

## Right tool for a job

Union Types are not an abstraction. They serve completely different purpose than protocols. Those two concepts could/should be used together. Notable differences:

- Protocols: define an abstract interface for implementors to conform to
- Union Types: a way of saying that an instance is of one of a fixed amount of types

- P: Typically define a common interface to do something with conforming types
- UT: No conventions (AFAIK)

- P: perfectly suited for public library/framework interfaces (IMHO)
- UT: perfectly suited for ever changing application code (IMHO)

They are completely different.

## NO TITLE FOR THIS SECTION YET

I don't think Swift is a good tool for my job at the moment. In particular, I doubt it's possible to create one language to rule them all. Is it possible to compromise lightning fast performance with neat language features greedy developers like me want?

Swift is already complicated. When things evolve, they tend to grow. Looking at its trajectory it will just get more keywords and secret powers which are not necessarily something I want.

A possible solution would be some subsets of the language. They would be tailored for specific use cases. I guess we are nowhere near this point though.

When it comes to the language feature I described above I am not going to write a proposal for [these reasons][reasons]

You can talk to me on [Twitter](https://twitter.com/wokalski).

[1]: - I didn't know how to extend a generic type to conform to a protocol.
     - There are some things you cannot express[2] /* - An instance conforming to a protocol with 'associatedtype' couldn't be returned. It can only be used as a generic constraint and the compiler could not infer the generic type. This is a serious limitation now but I expect it to be solved in future versions. */

[2]: Have you ever heard "this looks (feels) good"? It's the elegant trap. "This fits together". Elegant trap. I say it (or have this thought) every single work day.
     Code often looks nice. It's especially true about abstractions. But then it's very easy to be bitten by your own cleverness.

[4]: There[ are many names for it](https://en.wikipedia.org/wiki/Tagged_union)
[6]: They are actually patterns in `SOLID`.

[reasons]:

Since some people might ask this very question let me tell you:
1. First and foremost. **Swift is too complex** and this would make it even more complex. Ideally languages encourage consistent style. Too many features make it impossible.
2. Swift is an extremely ambitious project. Writing a 'performant' *and* 'flexible' *and* 'multi-paradigm' *and* 'innovative' *and* 'easy to learn' *and* 'simple to understand' language is a Chris Lattner's kind of thing. There is a lot of stuff to consider. At the end of the day there might be a better future and I'm just holding it wrong.
3. I am not qualified to argue (and I don't feel like arguing in the internet)

THAT SAID, I am open to discuss it and contribute my own observations to a proposal of somebody else.

