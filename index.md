---
title: "Lud's Dev Blog"
layout: default
---

# Lud's Dev Blog

<ul class="posts">
  {% for post in site.posts limit: 5 %}
    <li><span>{{ post.date | date_to_string }}</span>
        &mdash;
        <a href="{{ post.url }}">
            {% if post.lang %}<img src="{{ site.url}}/img/{{ post.lang }}.png" alt="post language flag"/>{% endif %}
            {{ post.title }}
        </a>
    </li>
  {% endfor %}
</ul>
