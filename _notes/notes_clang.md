---
title: "C언어 연구"
excerpt: "C언어 제대로 알자"
permalink: /notes/clang/
sidebar:
    nav: "menu_notes"
---
뭐야 이건 C언어 제대로 알자 이잖아.

{% for post in site.notes %}
  {% if post.path contains 'clang' %}
     {% include archive-single.html %}
  {% endif %}
{% endfor %}