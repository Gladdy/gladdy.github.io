---
layout: page
title: Projects
permalink: /projects/
---

<div class="row">

  {% for project in site.projects %}
    {% for i in (1..4) %}
      <div class="col-md-6 col-sm-12">
        <img src="http://static.guim.co.uk/sys-images/Guardian/Pix/pictures/2014/4/11/1397210130748/Spring-Lamb.-Image-shot-2-011.jpg" alt="Youtube Thumbnail Image">
        <p> {{ project.title }} </p>
      </div>
    {% endfor %}
  {% endfor %}


</div>
