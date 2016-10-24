---
layout: post
title: Ruby Concurrency and Parallelism - With a Practical Example
description: "Concurrency and Parallelism are not the same thing. Understand how they differ and how it affects multi-threading in Ruby, across interpreters."
share: true
comments: true
disqus: true
author: abhisheksarka
modified: 2016-10-19
blog: true
tags: [Ruby, Rails, RubyThreading, Concurrency, Parallelism]

---

To understand the difference between **concurrency** and **parallelism** let's begin with a straight forward example. Consider the below method `count`. It iterates over **n** integers decrementing **n** with every iteration till it reaches 0.

We will consider a sufficiently large **n** at **200000000**(200 million) and run it using <a href="https://en.wikipedia.org/wiki/Ruby_MRI" target="_blank">Ruby MRI/CRuby</a> and <a href="https://en.wikipedia.org/wiki/JRuby" target="_blank">JRuby</a>.

{% highlight ruby %}
def count(n)
  while n > 0 do
    n -= 1
  end
end
{% endhighlight %}

Below we have two versions of execution. **Single-Threaded** and **Muti-Threaded**


### Single-Threaded

{% highlight ruby %}
require 'benchmark'

puts Benchmark.measure {
  count(200000000) # n
}
{% endhighlight %}


### Multi-Threaded

{% highlight ruby %}
require 'benchmark'

puts Benchmark.measure {
  t1 = Thread.new { count(100000000) # n/2 }
  t2 = Thread.new { count(100000000) # n/2 }
  t1.join
  t2.join
}
{% endhighlight %}

### Results on a single processor system with *2 cores*

|                   | MRI Ruby(seconds)   | JRuby(seconds)   |
|:-----------------:|:----------:|:-------:|
| Single Threaded   | 3.96       | 27      |
| Multi Threaded    | 3.96       | 16      |

<br>
Okay, so the results seem a bit off and, inconsistent across interpreters. TBH, it kind of raises more questions than it answers. Now, before we go down the rabbit hole, keep in mind that this is not a <a href="https://en.wikipedia.org/wiki/Ruby_MRI" target="_blank">Ruby MRI/CRuby</a> vs <a href="https://en.wikipedia.org/wiki/JRuby" target="_blank">JRuby</a> blog. Instead, we will focus on how the execution time differs on per interpreter basis. The question you should be asking at this point is ...

**Why did muti-threading reduce the execution time by almost 41% in JRuby but, had no effect in MRI Ruby?**   

The reason is MRI Ruby uses a <a href="https://en.wikipedia.org/wiki/Global_interpreter_lock">GIL</a>(Global Interpreter Lock) to ensure that only one thread runs at a time. This prevents it from utilizing both the available cores in our system. So, both the threads most likely executed one after the other or the interpreter might have context-switched the threads. This is **concurrency**

In case of JRuby there is no GIL hence, the interpreter utilizes both the cores to execute the threads simultaneously and thereby, reduce the time of execution. This is **parallelism**

To get an idea of how the flow of execution might be, below is a very simple pictorial representation
<br><br>

#### Concurrent
<strong>
\|&nbsp;&nbsp;T1&nbsp;&nbsp;\|
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\|&nbsp;&nbsp;T2&nbsp;&nbsp;\|
<br>
\|&nbsp;&nbsp;T1&nbsp;&nbsp;\|
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\|&nbsp;&nbsp;T2&nbsp;&nbsp;\|
</strong>
<br><br>

#### Parallel

<strong>
\|&nbsp;&nbsp;T1&nbsp;&nbsp;\| \|&nbsp;&nbsp;T2&nbsp;&nbsp;\|
<br>
\|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\| \|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\|
<br>
\|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\| \|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\|
<br>
\|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\| \|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\|
</strong>
<br><br>

Curious readers might now wonder, if we did not achieve any benefit of multi-threading in MRI Ruby what is the point of using threads at all?

The answer to this lies in the nature of the task. Notice the `count` method written above is **non-blocking** in nature which means it does not wait for any system resources, IO operation, etc. Let us make a simple change and create a **blocking** version of it.

{% highlight ruby %}
def count(n)
  while n > 0 do
    sleep(0.5) # wait for half a second
    n -= 1
  end
end
{% endhighlight %}

Executing the same Single-Threaded and Multi-Threaded version of the code with **n=30** we get the following

|                   | MRI Ruby(seconds)   |
|:-----------------:|:-------------------:|
| Single Threaded   | 15.01               |
| Multi Threaded    | 7.55                |

<br>

It is vital to understand the difference between concurrency and parallelism. Using mutl-threading without knowing how it works might not yield the results you expect. Hopefully, this tutorial has provided a useful introduction regarding the same.

<nav class="pagination" role="navigation">
    {% if page.previous %}
        <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
    {% endif %}
    {% if page.next %}
        <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
    {% endif %}
</nav><!-- /.pagination -->
