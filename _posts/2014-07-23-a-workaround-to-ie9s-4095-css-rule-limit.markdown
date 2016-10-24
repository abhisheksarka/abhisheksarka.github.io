---
layout: post
title: A workaround to IE 9's 4095 CSS rule limit
description: "As it turns out IE9 has a limitation of 4095 selectors per CSS file. Selectors beyond that will be simply ignored."
share: true
comments: true
modified: 2014-07-22
blog: true
tags: [AngularJS, IE9, Internet Explorer 9, Ruby on Rails]
image:
  feature: abstract-12.jpg
---

Handling browser differences is not exactly a walk in the park for developers. Though it is not as big an issue as it was 5 years ago(thanks to all the awesome JS/CSS frameworks out there) but as long as IE 9 is still in the picture it is not eliminated completely. New versions of IE are definitely catching up as we faced only one or two small issues in our recent projects as far as compatibility is concerned. Kudos to that. Happy days are near.

As it turns out IE9 has a limitation of 4095 selectors per CSS file. Selectors beyond that will be simply ignored. This is the reason our app was breaking in production because of the huge size of the compiled `application.css` file. The fix for this is simple.

* Split your `application.css` into separate files like `application_split1.css`(for bootstrap and jquery), `application_split2.css`(for other plugins) and `application.css`(for your application styles).
* Configure your environment files to precompile the new `application_split*.css` files.
* Lastly import these files in the application head.

For us that wasn't the only problem. Our app was breaking in local too which was unexpected because all `.css` files were imported separately and none exceeded the 4095 limit.
The reason for this is that you can only have a maximum of 31 CSS imports in a page. For us it was somewhere around 45. So all `<link>` tags after the 31st one was discarded by the browser. While that is definitely not a desired behaviour it gave us an opportunity to clean some of our CSS files. After bringing the count down to 31, the CSS seemed to work fine except for one not so small issue.
We are using [AngularJs](https://angularjs.org/) in the front end and the forms have error messages that show up conditionally using `ng-show`. It goes something like this.

{% highlight html %}
  <span data-ng-show="!formName.formField.$valid" class="text-danger">Some error message.</span>
{% endhighlight %}

These error messages should show up only when the value in the field is invalid but in IE9 these messages showed all the time no matter what. For a moment we thought it might be some AngularJs issue but in the docs it's clearly stated that AngularJs supports IE9.

So after a little inspection we found out that even `<style>` tags will be ignored alongwith `<link>` tags after 31 imports. The reason this breaks the `ng-show` behaviour is because AngularJs adds an internal stylesheet to the document, declaring the necessary classes it needs. One of these classes is `.ng-hide` which is added to the element when the expression in `ng-show` evaluates to `false`. Since IE9 ignored the style `ng-hide` directive was having no effect.

We furthur reduced the imports and brought it down to 25 and now the styles are working perfectly in IE9 in all the environments.

To sum it up:-

* In IE9 a maximum of 4095 selectors are allowed per CSS file.
* and a maximum of 31 style tags(internal `<style>` and external `<link>`) are allowed per page.


<nav class="pagination" role="navigation">
    {% if page.previous %}
        <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
    {% endif %}
    {% if page.next %}
        <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
    {% endif %}
</nav><!-- /.pagination -->
