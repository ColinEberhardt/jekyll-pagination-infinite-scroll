---
author: ceberhardt
title: "Swift Sequences and Lazy Evaluation"
image: "ceberhardt/assets/featured/sequence.jpg"
categories: 
tags:
  - featured
  - featuredcolin
summary: "In this blog post I want to take a quick look at the Swift Sequence protocol, which forms the basis for the for-in loop, and see how this allows you to write code that performs sequence operations that are only evaluated on-demand."
layout: default_post
featured-overlay-inverted: true
---

I've had a lot of fun poking around the Swift APIs over the past few weeks. The lack of documentation means that there are a some real gems hidden among the more mundane API and language features.

In this blog post I want to take a quick look at the Swift `Sequence` protocol, which forms the basis for the `for-in` loop, and see how this allows you to write code that performs sequence operations that are only evaluated on-demand.

## Swift sequences

Swift has a couple of very elegant language features, the `for-in` loop and ranges, that when used in combination provide a very succinct syntax for simple loops:

{% highlight csharp %}
for i in 1...5 {
  println(i)
}
{% endhighlight %}

The above code iterates over the integers in the range 1 through to 5. 

The funky range operator (`...`) is a simple shorthand for creating `Range` instances. As a result, the following code is entirely equivalent:

{% highlight csharp %}
var range = Range(start: 1, end: 5)
for i in range {
  println(i)
}
{% endhighlight %}

But what is it about `Range` that allows the `for-in` loops to iterate over the numbers is generates?

The `for-in` loop operates on objects that adopt the `Sequence` protocol, which represents an ordered source of data. The `Sequence` protocol exposes a single method, `generate()`, which returns a `Generator`, which exposes a single method `next()` which allows the `for-in` loop to 'pull' a stream of items from the sequence.

This will probably all make a lot more sense by continuing to expand the current example to make explicit use of the sequence's generator:

{% highlight csharp %}
var sequence = Range(start: 1, end: 5)
var generator = sequence.generate()
while let i = generator.next() {
  println(i)
}
{% endhighlight %}

One interesting point is that the Swift `Range` is a finite sequence, in that it has a start and an end. The `Sequence` protocol does not mandate this, and can be used to represent infinite sequences (although you should probable avoid using infinite sequences with `for-in` loops!)

##Deferred Execution and Lazy Evaluation

Now that you have seen what a Swift Sequence is, let's look at how it supports the concept of lazy evaluation. First up, a couple of definitions:

**Deferred Execution**

Deferred execution means that the evaluation of some expression, or calculation, is deferred until the resultant value is required. With sequences you have the opportunity to defer execution until a value is requested from the generator.

**Lazy Evaluation**

Lazy evaluation is a topic that relates to deferred execution. In the context of Swift sequences, lazy evaluation can be implemented by ensuring that the work required to provide the next value via the generator is only performed when the next value is actually requested.

Both of the above probably sound like quite abstract concepts, so let's see how they can be used in practice.

##A Fibonacci sequence

Let's say you want to perform some calculations on a [Fibonacci sequence](http://en.wikipedia.org/wiki/Fibonacci_number). You could populate an array with the first 100 terms and use this for your calculation, but Swift sequences offer an elegant alternative.

Here is a simple implementation using a Swift sequence:

{% highlight csharp %}
class Fibonacci : Sequence {
  typealias GeneratorType = FibonacciGenerator
  
  func generate() -> FibonacciGenerator {
    return FibonacciGenerator()
  }
}

class FibonacciGenerator : Generator {
  var current = 0, nextValue = 1
  
  typealias Element = Int
  
  func next() -> Int? {
    let ret = current
    current = nextValue
    nextValue = nextValue + ret
    return ret
  }
}
{% endhighlight %}

The `Fibonacci` class implements the `Sequence` protocol (Note: `Sequence` is a generic protocol, hence the need for a `typealias` to specify the type parameter). The `FibonacciGenerator` is the generator implementation where the real work happens. Each time the `next()` method is invoked the next number in this never-ending sequence is generated.

To see the result, simply iterate over the first few numbers in the sequence:

{% highlight csharp %}
let fib = Fibonacci().generate()
for _ in 1..<10 {
  println(fib.next())
}
{% endhighlight %}

Which yields the following output:

<pre>
0
1
1
2
3
5
8
13
21
</pre>

There is actually a slightly simpler way to implement a sequence, that removes the need for a separate generator type. The following makes use of the `GeneratorOf`, where you pass a function (or closure) that implements the required functionality:

{% highlight csharp %}
class Fibonacci : Sequence {
  
  func generate() -> GeneratorOf<Int> {
    var current = 0, next = 1
    
    return GeneratorOf<Int> {
      var ret = current
      current = next
      next = next + ret
      return ret
    }
  }
}
{% endhighlight %}

Which is pretty neat!

In order to aid understanding the next set of examples, the following adds a simple `println` statement to the Fibonacci sequence so that you can see when the next item is requested from the generator:

{% highlight csharp %}
class Fibonacci : Sequence {
  
  let id:String
  init(_ id: String) {
    self.id = id;
  }
  
  func generate() -> GeneratorOf<Int> {
    var current = 0, next = 1
    
    return GeneratorOf<Int> {
      var ret = current
      current = next
      next = next + ret      
      println("\(self.id) - \(ret)")
      return ret
    }
  }
}
{% endhighlight %}

Updating the code that iterates over the sequence to use this implementation that logs the generation:

{% highlight csharp %}
let fib = Fibonacci("Fib#1").generate()
for _ in 1..5 {
  println(fib.next())
}
{% endhighlight %}

Yields the following output

<pre>
Fib#1 - 0
0
Fib#1 - 1
1
Fib#1 - 1
1
Fib#1 - 2
2
</pre>

This clearly demonstrates that the next item is being requested from the generator on each iteration and the Fibonacci sequence is being generated lazily.

##Manipulating sequences

The lazy evaluation of sequence is quite cool, but things get even more interesting when you apply operation to the sequence, transforming its output. The Swift APIs have a handful of operations you can apply to sequences.

Let's say you are only interested in the even numbers within the Fibonacci sequence. This can be achieved using the `filter` function:

{% highlight csharp %}
let fibonacci = Fibonacci("Fib#1")
let evenNumbers = filter(fibonacci) { $0 % 2 == 0}
{% endhighlight %}

The `filter` method returns a new sequence which contains the items within the source sequence for which the closure returns true. As a result `evenNumbers` is also a sequence.

As a result of deferred execution you can perform operations on sequences, but nothing will be generated until something starts 'pulling' data from one of the sequences.

To see this in action, iterate over the first few items in the sequence: 

{% highlight csharp %}
var generator = evenNumbers.generate()
for _ in 1..5 {
  println(generator.next())
}
{% endhighlight %}

This yields the following output:

<pre>
Fib#1 - 0
0
Fib#1 - 1
Fib#1 - 1
Fib#1 - 2
2
Fib#1 - 3
Fib#1 - 5
Fib#1 - 8
8
Fib#1 - 13
Fib#1 - 21
Fib#1 - 34
34
</pre>

As you can see, the source sequence is generating multiple values for each value that passes through the filter operation. Again, all lazily evaluated.

Another interesting operation can be applied by using the `Zip2` struct, which combines a pair of sequences, generating tuples which combine the generated output of each sequence.

For example, the following 'zips' together the odd and even Fibonacci number:

{% highlight csharp %}
let evenNumbers = filter(Fibonacci()) { $0 % 2 == 0}
let oddNumbers = filter(Fibonacci()) { $0 % 2 == 1}
var zipped = Zip2(oddNumbers, evenNumbers).generate()
for _ in 1..5 {
  println(zipped.next())
}
{% endhighlight %}

Which gives the following output:

<pre>
(1, 0)
(1, 2)
(3, 8)
(5, 34)
</pre>

##Creating your own sequence operations

Unfortunately `filter` and `Zip2` are pretty much the only sequence operations I could find within the Swift APIs. So how do you go about adding your own operations?

Personally I am not keen on global functions, and prefer to add functionality via class extensions.

Unfortunately `Sequence` is a protocol, so cannot be extended :-(

However, all is not lost, the framework provides `SequenceOf` struct which wraps adapts `Sequence`, without changing its behaviour, and in this context can be used as a type for providing extensions.

When creating sequence operations you have to be mindful of the fact that they should evaluate lazily, just like the Fibonacci sequence example above.

Here's a very simple 'skip' operation, that skips the first 'n' items in a sequence:

{% highlight csharp %}
extension SequenceOf {
  func skip (n:Int) -> SequenceOf<T> {
    var generator =  self.generate();
    for _ in 0..n {
      println("skipped")
      generator.next()
    }
    return SequenceOf(generator)
  }
}
{% endhighlight %}

Adding a skip to the zip:

{% highlight csharp %}
let evenNumbers = filter(Fibonacci("#1")) { $0 % 2 == 0}
let oddNumbers = filter(Fibonacci("#2")) { $0 % 2 == 1}
var zipped = SequenceOf(Zip2(oddNumbers, evenNumbers))
               .skip(2).generate()
for _ in 1...2 {
  println(zipped.next())
}
{% endhighlight %}

Yields the following output:

<pre>
skipped
#2 - 0
#2 - 1
#1 - 0
skipped
#2 - 1
#1 - 1
#1 - 1
#1 - 2
#2 - 2
#2 - 3
#1 - 3
#1 - 5
#1 - 8
(3, 8)
#2 - 5
#1 - 13
#1 - 21
#1 - 34
(5, 34)
</pre>

You can of course chain multiple operations `skip(2).skip(3)`, however, with just a simple skip operation this isn't much fun!

Because I think sequences show a lot of potential, I plan to contribute a suite of operations to the already rather excellent [ExSwift](https://github.com/pNre/ExSwift) library.

## Conclusions

Swift sequences are really cool and show a lot of potential. The examples in this blog post have been a little contrived, it is unlikely you will find yourself performing operations on the Fibonacci sequence! However, I could certainly see myself operating on sequences generated from streams, or perhaps paging network requests. 

Regards, Colin E.

