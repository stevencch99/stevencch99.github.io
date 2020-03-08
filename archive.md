---
bg: "john-wang-yyxyxA-oCLI-unsplash.jpg"
layout: page
permalink: /posts/
title: "Archive"
crawlertitle: "All articles"
summary: ""
active: archive
---
{% capture tags %}
  {% for tag in site.tags %}
    {{ tag[0] }}
  {% endfor %}
{% endcapture%}

{% assign sort_tags = tags | split: ' ' | sort %}

<div class="tag-wrapper">
{% for tag in sort_tags %}
  <a class="post-tags" href="{{ site.baseurl }}/posts/#{{ tag | downcase }}">{{ tag }}</a>
{% endfor %}
</div>

{% for tag in sort_tags %}
  <h2 class="category-key" id="{{ tag | downcase }}">{{ tag }}</h2>

  <ul class="year">
    {% for post in site.tags[tag] %}
      <li>
        {%- assign date_format = site.setting.date_format | default: "%b %-d, %Y" -%}
        {% if post.lastmod %}
          <a href="{{ post.url | relative_url}}">{{ post.title }}</a>
          <span class="date">{{ post.lastmod | date: date_format }}</span>
        {% else %}
          <a href="{{ post.url | relative_url}}">{{ post.title }}</a>
          <span class="date">{{ post.date | date: date_format }}</span>
        {% endif %}
      </li>
    {% endfor %}
  </ul>
{% endfor %}
