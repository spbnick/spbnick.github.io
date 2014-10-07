---
layout: post
title:  "Liquid: truncate without dots"
date:   2014-10-07 19:18:00
---
While building [the new DIGImend website][digimend] I needed a way to truncate
strings in [Liquid][shopify-liquid], which is the template language
[Jekyll][jekyll] uses. Specifically, I needed to compare parts of page URLs
when iterating over sub-pages of a page to generate menus and page lists.

To my surprise I couldn't find any. The version used by Jekyll didn't seem to
support [`slice`][liquid-slice], and the only alternative,
[`truncate`][liquid-truncate], seemed to always replace the end of the
truncated string with three dots ("...", an ellipsis).

However, diving into Liquid source in search of an alternative, I found that
[`truncate`][liquid-truncate] accepts optional second argument: a string to
replace that ellipsis with. So I could write this:

{% highlight liquid %}
{% raw %}
{% assign page_base = page_dir | truncate: parent_dir_len, "" %}
{% endraw %}
{% endhighlight %}

When writing this post I found that the Liquid's GitHub project wiki
[mentions][liquid-wiki-filters] this argument, but being otherwise quite low
on details it wasn't the first place to look to me.

[digimend]:             http://digimend.github.io
[shopify-liquid]:       http://docs.shopify.com/themes/liquid-documentation/basics
[jekyll]:               http://jekyllrb.com
[liquid-slice]:         http://docs.shopify.com/themes/liquid-documentation/filters/string-filters#slice
[liquid-truncate]:      http://docs.shopify.com/themes/liquid-documentation/filters/string-filters#truncate
[liquid-wiki-filters]:  https://github.com/Shopify/liquid/wiki/Liquid-for-Designers#standard-filters
