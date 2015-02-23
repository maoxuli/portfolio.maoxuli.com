---
layout: page
title: Tags
---

<div class='cloud comment'>// {{page.title}}<br>
{% for tag in site.tags %}
  <a href="#{{ tag[0] }}" title="{{ tag[0] }}" rel="{{ tag[1].size }}">{{ tag[0] }}({{ tag[1].size }})</a>
{% endfor %}
</div>

<div class="listing">
{% for tag in site.tags %}
  <p class="listing-seperator comment" id="{{ tag[0] }}">// {{ tag[0] }}</p>
{% for post in tag[1] %}
  <p class="listing-item">
    <span>
      <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
    </span>
    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  </p>
{% endfor %}
{% endfor %}
</div>
