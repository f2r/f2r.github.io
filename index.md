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

[Tous les articles](/articles-fr)

### English
{% assign posts = site.en | sort: 'published' | reverse %}
{% for post in posts limit: 5 %}
- {{post.date | date: "%d/%m/%Y"}}: [{{ post.title }}]({{ post.url }})
{% endfor %}

[All articles](/articles-en)
