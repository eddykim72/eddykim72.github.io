---
title: "임베디드 인터페이스 (Embedded Interfaces)"
permalink: /notes/embedded/
sidebar:
    nav: "menu_notes"
---
임베디드 인터페이스 연구

{% for post in site.notes %}
  {% if post.path contains 'embedded' %}
     {% include archive-single.html %}
  {% endif %}
{% endfor %}