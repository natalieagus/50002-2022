---
layout: default
title: 50.002 Lecture Notes
permalink: /notes/
feature-img: "assets/img/pexels/travel.jpeg"
tags: [Page]
---
<div id="main" class="call-out"
      style="background-image: url('{{ site.header_feature_image | relative_url }}')">
    <h1> {{ site.header_text | default: "Change <code>header_text</code> in <code>_config.yml</code>"}} </h1>
</div>

<!-- {% for t in site.notes %}

<h2><a href="{{ t.url"" | prepend: site.baseurl | prepend: site.url }}">{{ t.title }}</a></h2>


{% endfor %}   -->
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
            <h2>
              <a aria-label="{{ post.title }}" class="post-link" href="{{ post.url | relative_url }}">
                {{ post.title }}
              </a>
            </h2>
            {% include blog/post_info.html author=post.author date=post.date %}
          </header>
          {% if site.excerpt or site.theme_settings.excerpt %}
              <div class="excerpt">
                  {% if site.excerpt == "truncate" %}
                     {{ post.content | strip_html | truncate: '250' | escape }}
                  {% else %}
                     {{ post.excerpt | strip_html | escape }}
                  {% endif %}
              </div>
          {% endif %}
      </span>
    </div>
    {% endfor %}
</div>
<!-- ---
layout: page
title: About
permalink: /about/
feature-img: "assets/img/pexels/travel.jpeg"
tags: [Page]
---

Type on Strap is based on Type Theme, a free and open-source theme for [Jekyll](http://jekyllrb.com/), licensed under the MIT License.

Head over to the [theme's documentation](https://github.io/sylhare/Type-on-Strap) for much more information about Type on Strap or to install this theme on your own Jekyll site.

This file is an example of a page in Jekyll, that automatically shows up in the header navigation, you can delete or modify this file freely.
  -->