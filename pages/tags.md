---
layout: Post
title: By Tags
permalink: /tags/
content-type: eg
---


<br>
<div>
{% assign pages = site.notes | merge(site.posts) %}
{% assign tags =  pages | map: 'tags' | join: ','  | split: ',' | uniq %}
{% for tag in tags %}
  {%- assign conc = tag | first -%}
  {%- if conc != 'Favorite' -%}
    <h2 id="{{ conc }}">{{ conc }}</h2>
    {% for post in tag.last %} 
      <li id="category-content" style="padding-bottom: 0.6em; list-style: none;"><a href="{{post.url}}">{{ post.title }}</a></li>
    {% endfor %}
    {% for notes in tag.last %} 
      <li id="category-content" style="padding-bottom: 0.6em; list-style: none;"><a href="{{notes.url}}">{{ notes.title }}</a></li>
    {% endfor %}
  {%- endif -%}
{% endfor %}
</div>
<br/>
<br/>