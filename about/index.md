---
layout: home
---

<div class="index-content about">
    {% include aside.html %}
    <div class="section">
        <div id="content">
            <div class="brief">
                <h2>关于King</h2>
                <h3>前端悟道，悟道前端，我是King</h3>
                <p>
                    从大二开始专注前端开发，学习上求知欲强，工作中喜欢探索提高生产力的方法，并乐于与同事分享交流，使用mac的伪果粉，重度网虫，已放弃治疗。乐爱互联网，享受代码重构、抽象、解耦的乐趣，关注用户体验及性能优化，追崇国内前端领军人物。目前从经验和年龄上相比大牛还很屌丝，但坚信通过自己的不断努力定能成为前端翘楚。
                </p>
                <ul>
                    <li><span>QQ:</span>1c20942e</li>
                    <li><span>age:</span>11000</li>
                    <li><span>birthplace:</span>1010011011100101000</li>
                    <li><span>company:</span>\u71\u69\u66\u75\u6e</li>
                    <li><span>avatar:</span>aHR0cDovL3dlYmZpbmcucWluaXVkbi5jb20vbWouanBn</li>
                </ul>
            </div>
            <div id="disqus_container">
                <div id="disqus_thread"></div>
            </div>
        </div>
    </div>
</div>

<script>
    $(function(){
        window.disqus_shortname = 'cpjmj'
        $.getScript('http://' + disqus_shortname + '.disqus.com/embed.js');

    })
</script>