---
title: オンラインでホスティングされている手順
permalink: index.html
layout: home
---

# Microsoft Fabric の演習

次の演習は、[Microsoft Learn](https://aka.ms/learn-fabric) のモジュールをサポートするように設計されています。

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| モジュール | ラボ |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

