---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
---

<h2 class="post-list-heading"> Selected Categories: </h2>

<h3> Spanning Trees: </h3>
<ul>
{% assign sortedPosts = site.categories['spanning-tree'] | sort: 'url' %}

{% for post in sortedPosts %} {% if post.title == page.title %}
<li>{{ post.title }}</li>
{% else %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endif %}
{% endfor %}
</ul>

<h2 class="post-list-heading"> Other Posts </h2>

{% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}

<ul class="post-list">
  {% for post in site.posts %}
        {% unless post.categories contains 'spanning-tree' %}
            <li>
                <span class="post-meta">{{ post.date | date: date_format }}</span>
                <h3>
                  <a class="post-link" href="{{ post.url | relative_url }}">
                    {{ post.title | escape }}
                  </a>
                </h3>
            </li>
         {% endunless %}
  {% endfor %}
</ul>


[comment]: <> (<span class="post-meta">Oct 29, 2021</span>)

[comment]: <> (<h3>)

[comment]: <> (<a class="post-link" href="/spanning-tree/2021/10/29/spanning-tree-4.html">)

[comment]: <> (Spanning Trees: Distributed Bellman-Ford)

[comment]: <> (</a>)

[comment]: <> (</h3>)
