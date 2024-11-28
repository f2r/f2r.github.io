## English posts

{% for post in site.en %}
- [{{ post.title }}]({{ post.url }}) ({{ post.updated | date: "%B %d, %Y" }})
{% endfor %}
