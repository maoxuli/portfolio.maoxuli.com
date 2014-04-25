---
layout: page
title: Tags
---

<div class='cloud comment'>// {{page.title}}<br>
{% for tag in site.tags %}
  <a href="#{{ tag[0] }}" title="{{ tag[0] }}" rel="{{ tag[1].size }}">{{ tag[0] }}({{ tag[1].size }})</a>
{% endfor %}
</div>

<ul class="listing">
{% for tag in site.tags %}
  <li class="listing-seperator comment" id="{{ tag[0] }}">// {{ tag[0] }}</li>
{% for post in tag[1] %}
  <li class="listing-item">
    <span>
      <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
    </span>
    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  </li>
{% endfor %}
{% endfor %}
</ul>
