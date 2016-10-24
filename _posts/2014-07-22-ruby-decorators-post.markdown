---
layout: post
title: Ruby Decorators
description: "While inheritance is one form of extension it is not always the best way to achieve flexibility in our designs. In certain cases you might want to add a number of different responsibilities to a base class."
share: true
comments: true
disqus: true
modified: 2014-07-22
tags: [Ruby, Rails, Decorator Pattern, Design Patterns]
blog: true
author: abhisheksarka
image:
  feature: abstract-10.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/

---

While inheritance is one form of extension it is not always the best way to achieve flexibility in our designs. In certain cases you might
want to add a number of different responsibilities to a base class. With the increase in the number of responsibilities you will have to write numerous classes each of them inheriting from the base class. This is where the decorator pattern comes in. It provides an alternative to subclassing. It allows you to add additional responsibilities to an object dynamically.

### An example scenario

Let us consider a use case where we have a class called 'Car'. The maximum speed of any car is 120kmph but by adding some enhancements the speed of the car can be increased to virtually any number(no speed limits imposed). Right now let us assume that there are two such enhancements available in the garage, "Alpha" and "Beta". "Alpha" and "Beta" can increase the speed of the car by 70kmph and 50kmph respectively. Also we should be able to add n number of these enhancements to reach incredibly high speeds.

### Solving using inheritance

{% highlight ruby %}
# The base class
class Car
  # has a default top speed of 120
  def top_speed
    120
  end
end
{% endhighlight %}

{% highlight ruby %}
class CarWithAlpha < Car
  def top_speed
    super + 70
  end
end
{% endhighlight %}

{% highlight ruby %}
class CarWithBeta < Car
  def top_speed
    super + 50
  end
end
{% endhighlight %}

{% highlight ruby %}
class CarWithAlphaAndBeta < Car
  def top_speed
    super + 70 + 50
  end
end
{% endhighlight %}

While the above implementation looks good enough but imagine what happens when the number of these garage enhancements increase. The combinations of various enhancements would also increase and hence in the end we would end up with a class explosion which would be nothing but a maintenance nightmare.


### Enter the decorator pattern

* A decorator is a wrapper object on the component class.
* Has the same interface as the component it is decorating so that it's presence is transparent to the clients.
* Delegates requests to components.
* Performs additional actions before or after delegating.

Now that we know what the pattern is lets write the classes

* First of all we need to identify the component class that we want to "Decorate". In our case it's "Car".
* Then create the decorator classes following the rules mentioned above.

The base class remains as is

{% highlight ruby %}
class Car
  def top_speed
    120
  end
end
{% endhighlight %}

Following the points mentioned above the decorator class will be as follows.

{% highlight ruby %}
 class Alpha
    # Wrapping the component class, Car object in this case
    def initialize(component)
      @component = component
    end
    # maintain the same public interface as component
    def top_speed
      # delegate to request to component and after delegation modify the value
      @component.top_speed + 70
    end
  end
{% endhighlight %}

In the same way let's write our second decorator

{% highlight ruby %}
 class Beta
    def initialize(component)
      @component = component
    end

    def top_speed
      @component.top_speed + 50
    end
  end
{% endhighlight %}


Now that our decorators/enhancers are ready let's create some superfast cars.

{% highlight ruby %}
  car_one = Alpha.new(Car.new) # one alpha
  car_one.top_speed # 190
  car_two = Alpha.new(Beta.new(Car.new)) # one alpha and one beta
  car_two.top_speed # 240
  car_three = Alpha.new(Alpha.new(Beta.new(Car.new))) # two alphas and one beta
  car_three.top_speed # 310
{% endhighlight %}

That's it. That's our hero "The decorator pattern" but use it with caution. Overusing it might end up complicating your design.  

<nav class="pagination" role="navigation">
    {% if page.previous %}
        <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
    {% endif %}
    {% if page.next %}
        <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
    {% endif %}
</nav><!-- /.pagination -->
