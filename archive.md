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
    <li>
      <a class="post-link" href="{{ post.url | relative_url }}">
        {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
        <div class="post-meta">{{ post.date | date: date_format }}</div>
        <h3 class="post-title">{{ post.title | escape }}</h3>
      </a>
    </li>
    {%- endfor -%}
  </ul>
{%- endfor -%}
</section>
