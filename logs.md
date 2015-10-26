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

<div id="disqus_thread" style="margin-top:50px;"></div>
  <script type="text/javascript">
    /* * * CONFIGURATION VARIABLES * * */
    var disqus_shortname = 'gladdygithubio';
    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
    var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
    dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
    (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
  </script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
