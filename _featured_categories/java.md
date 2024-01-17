---
layout: list
title: Java
slug: java
sidebar: true
description: >
  Java 공부를 기록하는 카테고리입니다.
type: category
menu : true
submenu: true
order: 4
no_groups: false
---

<style>
  ul {
    /*list-style-type: none;*/
    margin: 0;
    padding: 0;
  }

  li {
    font-size: 1.1em;
    font-weight: bold;
    color: #e50c0c; /* 원하는 글자색으로 변경 */
    margin-bottom: 8px;
  }
</style>

{% assign posts_with_tags = site.categories.java | where_exp:"post", "post.tags.size > 0" %}
{% assign grouped_posts = posts_with_tags | group_by_exp:"post", "post.tags[0]" %}

{% for group in grouped_posts %}
<h2 onclick="toggleGroup('{{ group.name }}')" style="cursor: pointer;">
{{ group.name }}
</h2>
  <ul id="{{ group.name }}">
    {% for post in group.items %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}

<script>
  function toggleGroup(groupName) {
    var group = document.getElementById(groupName);
    group.style.display = (group.style.display === 'none' || group.style.display === '') ? 'block' : 'none';
  }

  function toggleTags() {
    var tags = document.querySelectorAll('h2[id]');
    tags.forEach(function(tag) {
      tag.style.display = (tag.style.display === 'none' || tag.style.display === '') ? 'block' : 'none';
    });
  }
</script>