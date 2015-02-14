---
layout: page
title: Portfolio
---

<ul class="listing">
{% for post in site.posts %}
  {% if post.close != null %}
  {% capture y %}{{post.close | date:"%Y"}}{% endcapture %}
  {% if year != y %}
    {% assign year = y %}
    <li class="listing-seperator comment" id="{{ y }}">// {{ y }}</li>
  {% endif %}
  <li class="listing-item">
    <span>
      <time datetime="{{ post.close | date:"%Y-%m-%d" }}">{{ post.close | date:"%Y-%m-%d" }}</time>
    </span>
    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
    {% if site.truncate %}
    <p>{{ post.content | strip_html | truncate: site.truncate }}</p>
    {% endif%}
  </li>
  {% endif %}
{% endfor %}
</ul>
