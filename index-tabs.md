---
title: "Lud's Dev Blog"
layout: default
---

# Lud's Dev Blog

<ul id="lang-switch" class="nav nav-tabs">
  <li class="active"><a id="all-lang" href="#">All Posts</a></li>
  <li><a class="lang-switch" data-lang="en" href="#">In English</a></li>
  <li><a class="lang-switch" data-lang="fr" href="#">En Français</a></li>
</ul>

<br/>

<ul class="posts all">
  {% for post in site.posts %}
    <li lang="{{ post.lang }}"><span>{{ post.date | date_to_string }}</span>
        &mdash;
        <a href="{{ post.url }}">
            {% if 1 == 0 %}<img src="{{ site.url}}/img/{{ post.lang }}.png" alt="post language flag"/>{% endif %}
            {{ post.title }}
        </a>
    </li>
  {% endfor %}
</ul>

<script type="text/javascript">
    $(function(){
        $('#all-lang').on('click',function(){
            $('ul.posts').addClass('all');
            $('ul.posts li').show();
        });
        $('.lang-switch').on('click',function(){
            $('ul.posts').removeClass('all');
            $('ul.posts li').hide();
            $('ul.posts li[lang="'+$(this).attr('data-lang')+'"]').show();
        });
        var $langswitchs = $('#lang-switch li a');
        $langswitchs.on('click',function(){
            $langswitchs.parent().removeClass('active');
            $(this).parent('li').addClass('active');
        });
    })
</script>
