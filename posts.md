---
layout: page
title: Posts
permalink: /posts/
---

{% for category in site.categories %}
<h4 style="text-transform:capitalize;">{{ category | first }}</h4>
<ul>
  {% for posts in category %}
    {% for post in posts %}
      {% if post.url%}
        <h5>
        <li>
        <a href="{{ post.url }}">{{ post.title }}</a> -
        <span class="post-meta">({{ post.date | date: "%b %-d, %Y" }})</span>
        </li>
        </h5>
      {% endif %}
    {% endfor %}
  {% endfor %}
</ul>
{% endfor %}
