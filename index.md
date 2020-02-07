---

layout: default

---
<div class="text-al-center">
  {% if site.posts.size == 0 %}
    <h2>Nenhum post ainda :(</h2>
  {% else %}
    {% for post in site.posts %}
      <div>
        <a href="{{post.url}}">
          <h2 class="list-post-title">
            {{ post.title }}
          </h2>
        </a>
        <div>
          <time>{{ post.date | date_to_string }}</time>
        </div>
        <div>
          <p>{{ post.description}}</p>
          <a href="{{post.url}}"><p>Leia mais</p></a>
          <hr class="divisor-post">
        </div>
      </div>
    {% endfor %}
  {% endif %}
</div>