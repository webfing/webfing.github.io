---
layout: home
---

<div class="index-content blog">
    <div class="section">
    <ul class="artical-cate">
        <li class="on"><a href="/" title="study"><span>求知欲</span></a></li>
        <li style="text-align:center"><a href="/think" title="think"><span>控索癖</span></a></li>
        <li style="text-align:right"><a href="/project" title="project"><span>项目经</span></a></li>
    </ul>

    <div class="cate-bar"><span id="cateBar"></span></div>

    <ul class="artical-list">
    {% for post in site.categories.study %}
        <li>
            <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
            <div class="title-desc">{{ post.description }}</div>
        </li>
    {% endfor %}
    </ul>
    </div>
    <div class="aside">
        <div class="aside-wrap">
            <div class="aside-content">
                <a href="" class="avartar"><img src="http://webfing.qiniudn.com/avartarB.jpg" alt=""></a>
                <h1>悟道前端</h1>
                <small>闻道有先后，术业有专攻</small>
                <hr>
                <p class="brief">生于1990，毕业于非知名学校的非专业码农，希望成为前端翘楚，并一直努力着</p>
                <div class="link">
                    <nav>
                        <ul>
                            <li>
                                <a href="">求知欲</a>
                            </li>
                            </li>
                                <a href="">控索癖</a>
                            </li>
                            </li>
                                <a href="">项目经</a>
                            </li>
                            </li>
                                <a href="">关于我</a>
                            </li>
                        </ul>
                    </nav>
                </div>
                <div class="social">
                    <nav>
                        <ul>
                        <!-- Weibo -->
                        <li>
                        <a href="http://weibo.com/onevcat" title="@king 的微博" target="_blank">
                          <i class="social fa fa-weibo"></i>
                          <span class="label">Weibo</span>
                        </a>
                        </li>

                        <!-- Github -->
                        <li>
                        <a href="https://github.com/onevcat" title="@king 的 Github" target="_blank">
                          <i class="social fa fa-github"></i>
                          <span class="label">Github</span>
                        </a>
                        </li>

                        <!-- Twitter -->
                        <li>
                        <a href="http://twitter.com/onevcat" title="@king" target="_blank">
                          <i class="social fa fa-twitter"></i>
                          <span class="label">Twitter</span>
                        </a>
                        </li>

                        <!-- Google Plus -->
                        <li>
                        <a href="https://plus.google.com/107108267983477358170" rel="author" title="Google+" target="_blank">
                          <i class="social fa fa-google-plus-square"></i>
                          <span class="label">Twitter</span>
                        </a>
                        </li>

                        <!-- RSS -->
                        <li>
                        <a href="http://onevcat.com/rss/" rel="author" title="RSS" target="_blank">
                          <i class="social fa fa-rss"></i>
                          <span class="label">RSS</span>
                        </a>
                        </li>

                        <!-- Email -->
                        <li>
                        <a href="mailto:onev@onevcat.com" title="邮件联系我">
                          <i class="social fa fa-envelope"></i>
                          <span class="label">Email</span>
                        </a>
                        </li>

                        </ul>
                        </nav>
                </div>
            </div>
        </div>
    </div>
</div>
