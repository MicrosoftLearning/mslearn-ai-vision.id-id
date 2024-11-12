---
title: Latihan Azure AI Visual
permalink: index.html
layout: home
---

# Latihan Azure AI Visual

Latihan berikut dirancang untuk mendukung modul di Microsoft Learn.


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %}

{% untuk aktivitas di lab %} 
  - [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
