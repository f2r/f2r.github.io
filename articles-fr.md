---
layout: index
title: "Tous les articles en français"
---

# Tous les articles en français

{% assign posts = site.fr | sort: 'date' | reverse %}
{% for post in posts %}
- {{ post.date | date: "%d/%m/%Y" }}: [{{ post.title }}]({{ post.url }})
{% endfor %}
