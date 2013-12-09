---
title: Hello, World !
layout: default
---

# Hello World qzd


Proin eleifend libero accumsan felis luctus nec consectetur purus commodo. Phasellus sodales est nec massa imperdiet commodo. Maecenas risus nulla, placerat vel vestibulum vel, dapibus quis libero [shop](/A-shoplist-with-d3js/).

<ul>
  {% for post in site.posts limit: 5 %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

Donec libero libero, bibendum non condimentum ac, ullamcorper at sapien. Duis feugiat urna vel justo cursus facilisis. Vivamus ligula dui, convallis a varius vitae, facilisis eget magna.

{% highlight erlang %}
foo(bar) -> baz;
foo(_) -> bee.

foo2(X) ->
    case X
        of bar -> baz
         ; _ -> bee
    end.

toString(X) -> toString(X,false).

toString(bar,false) -> "bar";
toString(bar,true) -> <<"bar">>;
toString(X,ToBin) ->
    case ToBin
        of true -> <<"Other binary">>
         ; false -> "Some list"
    end.

{% endhighlight %}

