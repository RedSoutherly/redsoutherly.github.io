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
    <h2 id="{{ conc }}">{{ conc }}</h2>
    {% for post in tag.last %} 
      <li id="category-content" style="padding-bottom: 0.6em; list-style: none;"><a href="{{post.url}}">{{ post.title }}</a></li>
    {% endfor %}
{% endfor %}
</div>
<div>
{% assign tags =  site.notes | map: 'tags' | join: ','  | split: ',' | uniq %}
{% for tag in tags %}
  <h2 id="{{ tag }}">{{ tag }}</h2>
  {% for note in site.notes %} 
    {$- if note.tags contains tag -$}
      <li id="category-content" style="padding-bottom: 0.6em; list-style: none;"><a href="{{note.url}}">{{ note.title }}</a></li>
    {%- endif -%}
  {% endfor %}
{% endfor %}
</div>
<br/>
<br/>