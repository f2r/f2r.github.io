---
layout: default
title: "Bienvenue sur F2R Articles"
---

# Bienvenue sur F2R Articles

Choisissez une langue / Choose your language :

- [Articles en français](/fr/)
- [English posts](/en/)

## Derniers articles

### Français
{% for post in site.categories.fr limit: 3 %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}

### English
{% for post in site.categories.en limit: 3 %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}
