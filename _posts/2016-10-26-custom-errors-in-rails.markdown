---
layout: post
title: Custom Errors in Rails - A Cleaner Approach
description: "Often times I have seen programmers raise custom exceptions as, raise 'User is not authenticated', which is a very simple and readable approach but, does not have a well-defined structure to it. These are just strings littered all across your application and sometimes these strings get really really long. Ughh!"
share: true
comments: true
disqus: true
author: abhisheksarka
modified: 2016-10-26
blog: true
tags: [Ruby, Rails, Exception Handling, Custom Errors]

---

Often times I have seen programmers raise custom exceptions as `raise 'User is not authenticated'` which is a very simple and readable approach but, does not have a well-defined structure to it. These are just strings littered all across your application and sometimes these strings get really really long. Ughh!

Another way to raise custom exceptions is by first defining a sub-class inheriting `StandardError` and then raising it as shown below.

{% highlight ruby %}
class AuthenticationError < StandardError; end
class GatewayError < StandardError; end

raise AuthenticationError
{% endhighlight %}

Both the approaches are limited in scope as to what the error conveys. Also, forget about adding extra details in the errors like user-friendly messages, custom error codes, HTTP codes(if needed), etc.

But, then you might be wondering, why even need these details? Good thought ;) For a moment consider that the errors raised by your application **does respond to** `code`, `message` and `http_code`.

Since we know every error message contains these details we can capture it in a single place in our `application_controller.rb` and respond to it in a generic fashion. This kind of elegance is very useful in the long run since it promotes reusability and you don't have to handle errors in every controller. Just raise the custom exception and it will be handled automatically. Below is an example error handler in an example `api/application_controller.rb` file.

{% highlight ruby %}
# This is a generic API response handling
# mechanism. This is a basic example which does not
# include object serialization options but is enough
# to validate the point I am trying to make.

rescue_from ApplicationError do | ex |
  response_handler(ex)
end

def response_handler(res)
  if res.kind_of? StandardError
    error_handler(res)
  else
    success_handler(res)
  end
end

def error_handler(e)
  code = e.config[:http_code] || 500
  render status: code, json: { success: false, error: e.message, code: e.code }
end

def success_handler(e)
  render status: 200, json: { success: true, data: e }
end

{% endhighlight %}

Now, that we have an idea of why we need such a system in place let's get into how to do it.

Firstly, we **do not** want to create classes for every custom error and secondly, we want to make our errors declarative in a configuration file with the required details and it should be usable by the application.

The best solution(at least the one that I could come up with) is using some <a href="https://www.sitepoint.com/ruby-metaprogramming-part-i/">meta-programming</a> magic(specifically <a href="http://apidock.com/ruby/Module/const_missing">const_missing</a>) which should do the following

* Determine the exception our application is raising
* Find the details in the configuration file
* Create the error object

{% highlight ruby %}

# A plain old ruby class inheriting from StandardError
class ApplicationError < StandardError

  # responds to all the below attributes
  attr_accessor :config,
                :code,
                :message,
                :http_code

  # We pass config hash of the sort
  # {code: 100, message: 'Some Error Message', http_code: 501}              
  def initialize(config)
    @config = config
    @code = config[:code]
    @message = config[:message]
    @http_code = config[:http_code]
  end

  class << self
    # Whenever we call something like ApplicationError::SomeCustomError
    # this method is triggered. We then do a look up in an error configuration
    # file, discussed below, and then determine what error object to instantiate
    def const_missing(name)
      I18n.reload!
      err_hash = t(name)
      if err_hash.is_a? Hash
        err_hash[:name] = name
        return ApplicationError.new(err_hash)
      else
        super
      end
    end

    def t(error_name)
      I18n.t("error.#{error_name.to_s.underscore}")
    end
  end
end

{% endhighlight %}

The `ApplicationError` class now can be used to raise any kind of custom error as long as they have been configured in the `errors.en.yml` file inside of the `error` section. An example of an `errors.en.yml` file is given below.

{% highlight yaml %}
en:
  error:
    authentication_failure:
      http_code: 401
      code: 1050
      message: 'The user is not authenticated.'
    payment_gateway_timeout:
      code: 1100
      message: 'The payment gateway timed out.'
{% endhighlight %}

After the errors have been declared in the `errors.en.yml` file we can raise them as
{% highlight ruby %}
raise ApplicationError::AuthenticationFailure # ApplicationError: The user is not authenticated.
raise ApplicationError::PaymentGatewayTimeout # ApplicationError: The payment gateway timed out.
{% endhighlight %}

Looks much cleaner, doesn't it?  

<nav class="pagination" role="navigation">
    {% if page.previous %}
        <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
    {% endif %}
    {% if page.next %}
        <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
    {% endif %}
</nav><!-- /.pagination -->
