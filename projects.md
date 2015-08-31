---
layout: page
title: Projects
permalink: /projects/
---

<div class="row">

  {% for project in site.projects %}
    <div class="col-md-6 col-sm-12">
      <img src="{{ project.imgurl }}" alt="Youtube Thumbnail Image">
      <p> {{ project.title }} </p>
    </div>
  {% endfor %}


</div>
