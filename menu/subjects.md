---
layout: page
title: Materias
subjects: [Algoritmos y Estructuras de Datos,Organización de Computadoras,Programación Orientada a Objetos,Proyecto Informático I]
---
<ul class="posts">
  {% for subject in page.subjects %}
    <h3>{{ subject }}</h3>
<!--
    {% unless post.next %}
      <h3>{{ post.date | date: '%Y' }}</h3>
    {% else %}
      {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
      {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %}
      {% if year != nyear %}
        <h3>{{ post.date | date: '%Y' }}</h3>
      {% endif %}
    {% endunless %}
-->
    {% for post in site.posts %}

      {% if post.subject == subject %}
      <li itemscope>
        <a href="{{ site.github.url }}{{ post.url }}">{{ post.title }}</a>
        <p class="post-date"><span><i class="fa fa-calendar" aria-hidden="true"></i> {{ post.date | date: "%B %-d" }} - <i class="fa fa-clock-o" aria-hidden="true"></i> {% include read-time.html %}</span></p>
      </li>
      {% endif %}

    {% endfor %}

  {% endfor %}
</ul>
