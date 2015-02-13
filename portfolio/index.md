---
layout: page
title: Portfolio
---

<ul class="listing">
{% for post in paginator.posts %}
  {% capture y %}{{post.date | date:"%Y"}}{% endcapture %}
  {% if year != y %}
    {% assign year = y %}
    <li class="listing-seperator comment" id="{{ y }}">// {{ y }}</li>
  {% endif %}
  <li class="listing-item">
    <span>
      <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
    </span>
    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
    {% if site.truncate %}
    <p>{{ post.content | strip_html | truncate: site.truncate }}</p>
    {% endif%}
  </li>
{% endfor %}
</ul>

<div class="page-nav">
  {% if paginator.previous_page %}
    {% if paginator.previous_page == 1 %}
      <a href="/">&lt;PREV</a>
    {% else %}
      <a href="/page{{paginator.previous_page}}">&lt;PREV</a>
    {% endif %}
  {% else %}
    <span>&lt;PREV</span>
  {% endif %}

  {% if paginator.page == 1 %}
    <span class="current-page">1</span>
  {% else %}
    <a href="/">1</a>
  {% endif %}

  {% for count in (2..paginator.total_pages) %}
    {% if count == paginator.page %}
      <span class="current-page">{{count}}</span>
    {% else %}
      <a href="/page{{count}}">{{count}}</a>
    {% endif %}
  {% endfor %}

  {% if paginator.next_page %}
    <a class="next" href="/page{{paginator.next_page}}">NEXT&gt;</a>
  {% else %}
    <span>NEXT&gt;</span>
  {% endif %}
  
  (Total {{ paginator.total_posts }} posts)
</div>