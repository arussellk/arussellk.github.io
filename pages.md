---
layout: default
title: Pages
---

This is a collection of intuitive or illustrative explanations of various topics mostly in CS or math. These explanations aren't necessarily exhaustive or formal proofs, instead they serve to help me really understand the what and why of these ideas.

<ul>
  {% assign pages = site.pages | sort: 'title' %}
  {% for page in pages %}
  <li><a href="{{ page.url }}">{{ page.title }}</a></li>
  {% endfor %}
<ul>
