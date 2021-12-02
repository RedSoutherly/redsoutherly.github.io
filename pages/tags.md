---
layout: Post
title: By Tags
permalink: /tags/
content-type: eg
---

<br>
<div>
{% assign tags =  site.notes | map: 'tags' | join: ','  | split: ',' | uniq %}
{% for tag in tags %}
  <h2 id="{{ tag }}">{{ tag }}</h2>

  {% assign notes = site.notes | sort: "date" | reverese %}

  {% for note in notes %}
    {% if note.tags contains tag %}
      <li id="category-content" style="padding-bottom: 0.6em; list-style: none;"><a href="{{note.url}}">{{ note.title }}<small>{{ note.date | date: '%d-%B-%Y' }}</small></a></li>
    {% endif %}
  {% endfor %}
{% endfor %}
</div>
<br/>
<br/>