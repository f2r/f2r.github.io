---
layout: index
title: "Bienvenue sur F2R Articles"
---

# Bienvenue sur F2R Articles

## Derniers articles / Last posts

### Français
{% for post in site.fr limit: 3 %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}

### English
{% for post in site.en limit: 3 %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}
