---
layout: post
title: Foo bar
category: posts             <!-- or code -->
tags: foo bar qux           <!-- need to have at least 1 tags -->
---
<!-- all of the above attributes are required! -->

## Section title

---
horizontal rule

  <hard line break>

`inline code block`

{% highlight [python] [linenos[=table]] %}
    highlighted
    code
    block
{% endhighlight %}

unordered list
- item
- item

ordered list
1.
2.
3.

<link with full url>
["alt text"](link "hover title")
!["alt text"](imglink "hover title")
{% include fig.html src="/assets/img/image.png" alt="alt text" title="hover text" caption="shown caption" %}

> blockquote paragraph
> continuing blockquote

_italic_, *italic*

__bold__, **bold**

~~ strike through ~~

table
1,1 | 1,2 | 1,3
---:|:---:|:---
align right | align center | align left
2,1 | 2,2 | 2,3

