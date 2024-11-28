## Articles en français
<ul class="posts">
  {% for post in site.categories.posts_fr limit:10 %}
    <li class="post">
      <a href="{{ post.url }}">{{ post.title }}</a>
      <time class="publish-date" datetime="{{ post.date | date: '%F' }}">
        {{ post.date | date: "%d/%m/%Y" }}
      </time>
    </li>
  {% endfor %}
</ul>