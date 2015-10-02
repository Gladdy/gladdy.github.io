---
layout: page
title: Cambridge logs
permalink: /logs/
---


<ul>
  {% for log in site.logs %}
  {% if log.url%}
  <h3>
    <li>
      <span class="post-meta">({{ log.date | date: "%b %-d, %Y" }})</span> -
      <a href="{{ log.url }}">{{ log.title }}</a>
    </li>
  </h3>
  {% endif %}
  {% endfor %}
</ul>
