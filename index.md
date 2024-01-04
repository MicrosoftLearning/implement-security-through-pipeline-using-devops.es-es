---
title: Ejercicios de implementación de la seguridad a través de una canalización mediante Azure DevOps
permalink: index.html
layout: home
---

# Ejercicios de implementación de la seguridad a través de una canalización mediante Azure DevOps

Los ejercicios siguientes están diseñados para admitir los módulos de [Implementación de la seguridad a través de una canalización mediante DevOps](https://learn.microsoft.com/training/paths/implement-security-through-pipeline-using-devops/).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
