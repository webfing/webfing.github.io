---
layout: home
---

<div class="index-content about">
    {% include aside.html %}
    <div class="section">

        <div id="content">
            <div class="about">
                <h2>关于King</h2>
                <div>前端悟道，悟道前端，我是King</div>
                <p>
                    从大二开始专注前端开发，学习上求知欲强，工作中喜欢探索提高生产力的方法，并乐于与同事分享交流，使用mac的伪果粉，重度网虫，已放弃治疗。乐爱互联网，享受代码重构、抽象、解耦的乐趣，关注用户体验及性能优化，追崇国内前端领军人物。目前从经验和年龄上相比大牛还很屌丝，但坚信通过自己的不断努力定能成为前端翘楚。
                </p>
                <ul>
                    <li><span>QQ:</span>10010011011011101</li>
                    <li><span>age:</span>0x16</li>
                    <li><span>birthplace:</span>341800</li>
                    <li><span>company:</span>QQ</li>
                    <li><span>avatar:</span>100110101011011101101010111011011</li>
                </ul>
            </div>
        </div>

        <div id="disqus_container">
            <div id="disqus_thread"></div>
        </div>

    </div>
</div>

<script>
    $(function(){

        $.getScript('http://' + disqus_shortname + '.disqus.com/embed.js');

    })
</script>