---
layout: index
title: "Bienvenue sur F2R Articles"
---

# Bienvenue sur F2R Articles

## Derniers articles / Last posts

### Français
{% assign posts = site.fr | sort: 'published' | reverse %}
{% for post in posts limit: 5 %}
- {{post.date | date: "%d/%m/%Y"}}: [{{ post.title }}]({{ post.url }})
{% endfor %}

### English
{% assign posts = site.en | sort: 'published' | reverse %}
{% for post in posts limit: 5 %}
- {{post.date | date: "%d/%m/%Y"}}: [{{ post.title }}]({{ post.url }})
{% endfor %}
