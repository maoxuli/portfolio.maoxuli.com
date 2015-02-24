---
layout: page
title: Categories
---

<div class='cloud comment'>// {{page.title}}<br>
{% for cat in site.categories %}
  <a href="#{{ cat[0] }}" title="{{ cat[0] }}" rel="{{ cat[1].size }}">{{ cat[0] }}({{ cat[1].size }})</a>
{% endfor %}
</div>

<div class="listing">
{% for cat in site.categories %}
  <p class="listing-seperator comment" id="{{ cat[0] }}">// {{ cat[0] }}</p>
{% for post in cat[1] %}
  <p class="listing-item">
    <span>
    {% if post.close == null %}
	  {{ post.date | date:"%m/%Y" }} - present
	{% else %}
	  {{ post.date | date:"%m/%Y" }} - {{ post.close | date:"%m/%Y" }}
	{% endif %}
    </span>
    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  </p>
{% endfor %}
{% endfor %}
</div>

