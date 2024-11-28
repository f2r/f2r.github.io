## Articles en français

{% for post in site.fr %}
- [{{ post.title }}]({{ post.url }}) ({{ post.updated | date: "%d/%m/%Y" }})
{% endfor %}