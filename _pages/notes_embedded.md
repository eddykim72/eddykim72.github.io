---
title: "임베디드 인터페이스 (Embedded Interfaces)"
permalink: /notes/embedded/
share: false
comments: false
sidebar:
    nav: "menu_notes"
---
임베디드 인터페이스 연구

{% assign notes = site.notes | sort: "date" | reverse %}

{% for post in notes %}
  {% if post.path contains 'embedded' %}
     {% include archive-single.html %}
  {% endif %}
{% endfor %}