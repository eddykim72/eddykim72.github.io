---
title: "C언어 연구"
permalink: /notes/clang/
share: false
sidebar:
    nav: "menu_notes"
---
뭐야 이건 C언어 제대로 알자 이잖아.

{% assign notes = site.notes | sort: "date" | reverse %}

{% for post in notes %}
  {% if post.path contains 'clang' %}
     {% include archive-single.html %}
  {% endif %}
{% endfor %}