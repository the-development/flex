---
layout: post
permalink: /blog/index.html
cover: post_index.jpg
title: Blog
---

<div class="posts">
  <ul class="posts-list">
    {% for post in site.posts %}
      <a class="post-title" href="{{ post.url }}">
        <span class="post-date">{{ post.date | date: '%B %-d, %Y' }}</span>
        {{ post.title }}
      </a>
    {% endfor %}
  </ul>
</div>
