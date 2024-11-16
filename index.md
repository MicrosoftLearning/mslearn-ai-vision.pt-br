---
title: Exercícios de Visão de IA do Azure
permalink: index.html
layout: home
---

# Exercícios de Visão de IA do Azure

Os exercícios a seguir foram projetados para dar suporte aos módulos no Microsoft Learn.


{% atribuir laboratórios = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %}

{% for activity in labs  %} 
  - [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
