---
title: Exercícios de Visão de IA do Azure
permalink: index.html
layout: home
---

# Exercícios de Visão de IA do Azure

Os exercícios a seguir foram projetados para dar suporte aos módulos no Microsoft Learn.


{% atribuir laboratórios = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %}

{% para atividade em laboratórios %} {% if activity.lab.title contains "Azure AI Custom Vision" %}  
    {% continuar %}  
  {% endif %} 
  - [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
