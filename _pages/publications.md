---
layout: page
permalink: /publications/
title: publications
description: my area of interest is Deep Learning, Database Optimization, and Algorithms
years: [2024, 2022]
nav: true
nav_order: 1
---
<!-- _pages/publications.md -->
<div class="publications">

{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f papers -q @*[year={{y}}]* %}
{% endfor %}

</div>
