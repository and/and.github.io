---
layout: home
title: "and.log"
---

Notes on what I'm learning.

{%- assign topics = site.categories | sort -%}
{%- if topics.size > 0 %}
<nav class="topic-index" aria-label="Topics">
  {%- for t in topics %}
  <a href="{{ '/topics/' | relative_url }}#{{ t[0] | slugify }}">{{ t[0] }} <span>{{ t[1].size }}</span></a>
  {%- endfor %}
</nav>
{%- endif %}
