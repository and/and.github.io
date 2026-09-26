---
layout: default
title: Archive
permalink: /archive/
---

<h1 class="page-heading">Archive</h1>

<section class="posts-section">
{%- assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" -%}
{%- for year in posts_by_year -%}
  <h2 class="post-list-heading">{{ year.name }}</h2>
  <ul class="post-list">
    {%- for post in year.items -%}
    {% include post-row.html post=post %}
    {%- endfor -%}
  </ul>
{%- endfor -%}
</section>
