---
layout: post
title: Fizz Buzz with Java 8 streams
---

The famous FizzBuzz interview question is used to filter out most of the programmers on a job interview.<br/>
It goes like this:
<blockquote>
<p>
Write a program that prints the numbers from 1 to 100.<br/>
But for multiples of three print “fizz” instead of the number and for the multiples of five print “buzz”.<br/>
For numbers which are multiples of both three and five print “fizzbuzz”."
</p>
</blockquote>

Here is my simple java 8 solution to demonstrate the power of the streams:

```java
import java.io.PrintStream;
import static java.util.stream.IntStream.rangeClosed;

public class FizzBuzzWithStreams {
    public static void main(String... arguments) {
      new FizzBuzzWithStreams().fizzBuzz(1, 100, System.out);
    }

    public void fizzBuzz(int from, int to, PrintStream out){
        rangeClosed(from,to)
                .mapToObj(FizzBuzzWithStreams::transformNr)
                .forEach(out::println);
    }

    public static String transformNr(int nr) {
        if (nr % 15 == 0) return "fizzbuzz";
        if (nr % 3 == 0) return "fizz";
        if (nr % 5 == 0) return "buzz";
        return Long.toString(nr);
    }
}
```

Despite the simplicity of this coding task it is not only about the basic algorithm of the division
but also to see how one structures and tests the code she writes.

It is also becomes more interesting when you spice it up with new requirements:<br/>
- If the number contains a three you must output the text 'lucky'.<br/>
- Produce a statistic at the end of the program showing how many times the following were output: fizz, buzz, fizzbuzz, lucky, an integer.

Check out my <a href="https://github.com/tamaslang/codingtest-fizzbuzz">Github repository</a> for my approach on this.

A good friend of mine also came across this problem recently which <a href="http://benedekfazekas.github.io/2015/02/06/java8-fizzbuzzed/">made him write a really good comparison</a> between java 8 and clojure solutions.

Finally if you haven't already solved this problem, try yourself c|;-)

