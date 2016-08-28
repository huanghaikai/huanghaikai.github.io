---
layout: default
title: Categories
scripts: [search.js]
permalink: /categories
---
<div class="archive-header">
    <h2>Categories</h2>
    <span class="icon toggle-search">{% include search.svg %} Search</span>
</div>
<div class="search">
    <div class="wrapper">
        <span class="icon toggle-search">{% include close.svg %}</span>
        <input type="text" class="search-input" id="search-input" placeholder="Search...">
        <ul id="results-container"></ul>
    </div>
</div>
<div class="tags-expo">
  <div class="tags-expo-list">
    {% for tag in site.categories %}
    <a href="#{{ tag[0] | slugify }}" class="post-tag">{{ tag[0] }}</a>
    {% endfor %}
  </div>
  <hr/>
  <div class="tags-expo-section">
    {% for tag in site.categories %}
    <h2 id="{{ tag[0] | slugify }}">{{ tag[0] }}</h2>
    <ul class="tags-expo-posts">
      {% for post in tag[1] %}
      <li>
          <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
          <small class="post-date">{{ post.date | date_to_string }}</small>
      </li>
      {% endfor %}
    </ul>
    {% endfor %}
  </div>
</div>
