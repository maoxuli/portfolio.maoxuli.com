---
layout: page
title: Categories
---

<p class="listing-seperator comment">// {{page.title}}</p>
<div class='cloud'>
{% for cat in site.categories %}
  <a href="#{{ cat[0] }}" title="{{ cat[0] }}" rel="{{ cat[1].size }}">{{ cat[0] }}({{ cat[1].size }})</a>
{% endfor %}
</div>

<div class="listing">
{% for cat in site.categories %}
  <p class="listing-seperator comment" id="{{ cat[0] }}">// {{ cat[0] }}</p>
{% for post in cat[1] %}
{% if post.close == null %}
  <p class="listing-item">
    <span>
	  {{ post.date | date:"%m/%Y" }} - present
    </span>
    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  </p>
{% endif %}
{% endfor %}

{% for post in cat[1] %}
{% if post.close %}
  <p class="listing-item">
    <span>
	  {{ post.date | date:"%m/%Y" }} - {{ post.close | date:"%m/%Y" }}
    </span>
    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  </p>
{% endif %}
{% endfor %}
{% endfor %}
</div>

