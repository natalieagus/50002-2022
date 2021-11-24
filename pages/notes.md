---
layout: default
title: Lecture Notes
permalink: /notes/
feature-img: "assets/img/pexels/travel.jpeg"
---
<div id="main" class="call-out"
      style="background-image: url('{{ site.header_feature_image | relative_url }}')">
    <h1> {{ site.header_text | default: "Change <code>header_text</code> in <code>_config.yml</code>"}} </h1>
</div>


<div class="posts">
    {% unless site.notes %}
    <article><section class="post-content"><p>There are no posts</p></section></article>
    {% endunless %}
    {% for post in site.notes %}
    <div class="post-teaser">
        {% if post.thumbnail %}
        <div class="post-img">
            <a aria-label="{{ post.title }}" href="{{ post.url | relative_url }}">
                <img alt="{{ post.title }}" src="{{ post.thumbnail | relative_url }}">
            </a>
        </div>
        {% endif %}
        <span>
          <header>
            <h3>
              <a aria-label="{{ post.title }}" class="post-link" href="{{ post.url | relative_url }}">
                {{ post.title }}
              </a>
            </h3>
          </header>
              <div class="excerpt">
                     {{ post.description | strip_html | truncate: '250' | escape }}
              </div>
      </span>
    </div>
    {% endfor %}
</div>

{% include blog/blog_nav.html %}