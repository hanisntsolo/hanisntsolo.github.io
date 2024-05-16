---
layout: page
title: DALL-E Generated Images
permalink: /dall-e-images/
---

## DALL-E Generated Images

{% assign images = site.static_files | where_exp: 'item', 'item.path contains "assets/img/dall-e"' %}
{% for image in images %}
  ![DALL-E Image]({{ image.path }})
{% endfor %}
