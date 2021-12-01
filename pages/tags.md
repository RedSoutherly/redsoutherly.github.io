---
layout: Post
title: By Tags
permalink: /tags/
content-type: eg
---


<br>
<div>
{% for tag in site.tags %}
  {%- assign conc = tag | first -%}
  {%- if conc != 'Favorite' -%}
    <h2 id="{{ conc }}">{{ conc }}</h2>
    {% for note in tag.last %} 
      <li id="category-content" style="padding-bottom: 0.6em; list-style: none;"><a href="{{note.url}}">{{ note.title }}</a></li>
    {% endfor %}
  {%- endif -%}
{% endfor %}
</div>
<br/>
<br/>
