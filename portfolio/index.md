---
layout: page
title: Portfolio
---

<ul class="listing">
{% for post in site.posts %}
  <li class="listing-seperator comment" id="present">// Present</li>
  {% if is_null({{ post.close }}) %}
    <li class="listing-item">
	  <span>
		<time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
	  </span>
	  {{ post.title }}
	</li>
  {% else %}
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
  {% endif %}
{% endfor %}
</ul>
