---
layout: home
---

<div class="index-content opinion">
    <div class="section">
        <ul class="artical-cate">
            <li><a href="/" title="study"><span>求知欲</span></a></li>
            <li class="on" style="text-align:right"><a href="/think" title="think"><span>控索癖</span></a></li>
            <li style="text-align:center"><a href="/project" title="project"><span>项目经</span></a></li>

        </ul>

        <div class="cate-bar"><span id="cateBar"></span></div>

        <ul class="artical-list">
        {% for post in site.categories.think %}
            <li>
                <h2>
                    <a href="{{ post.url }}">{{ post.title }}</a>
                </h2>
                <div class="title-desc">{{ post.description }}</div>
            </li>
        {% endfor %}
        </ul>
    </div>
    <div class="aside">
    </div>
</div>