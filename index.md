---
layout: page
title: SWE Notes
---

<div class="card-grid">
{% assign sorted_pages = site.pages | sort: "title" %}
{% for page in sorted_pages %}
{% if page.description and page.url != "/" %}
  <a href="{{ page.url | relative_url }}" class="card">
    <h3>{{ page.title }}</h3>
    <p>{{ page.description }}</p>
  </a>
{% endif %}
{% endfor %}
</div>
