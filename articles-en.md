---
layout: index
title: "All articles in English"
---

# All articles in English

{% assign posts = site.en | sort: 'date' | reverse %}
{% for post in posts %}
- {{ post.date | date: "%d/%m/%Y" }}: [{{ post.title }}]({{ post.url }})
{% endfor %}
