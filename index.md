---

layout: default

---


<div class="content list">
  {% if site.posts.size == 0 %}
    <h2>Nenhum post ainda :(</h2>
  {% else %}
    {% for post in site.posts %}
      <div class="list-item">
        <a href="{{post.url}}">
          <h2 class="list-post-title">
            {{ post.title }}
          </h2>
        </a>
        <div class="list-post-date">
          <time>{{ post.date | date_to_string }}</time>
        </div>
        <div class="text-description-header">
          <p>{{ post.description}}</p>
          <a href="{{post.url}}"><p>Leia mais </p></a>
          <hr class="divisor-post">
        </div>
      </div>
    {% endfor %}
  {% endif %}
</div>