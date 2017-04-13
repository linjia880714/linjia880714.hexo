---
title: Hexo一些技巧
date: 2017-04-13 10:26:07
tags: Hexo
categories:
- IT
- Hexo
---

<!-- toc -->

---

### 1.链接到站内文章
```bash
{% post_path hello-world %}
// /2015/01/16/hello-world/
{% post_link hello-world %}
// <a href="/2015/01/16/hello-world/">Hello World</a>
{% post_link hello-world Custom Title %}
// <a href="/2015/01/16/hello-world/">Custom Title</a>

{% asset_path example.jpg %}
// /2015/01/16/hello-world/example.jpg
{% asset_link example.jpg %}
// <a href="/2015/01/16/hello-world/example.jpg">example.jpg</a>
{% asset_link example.jpg Example %}
// <a href="/2015/01/16/hello-world/example.jpg">Example</a>
{% asset_img slug %}
// <img src="/2015/01/16/hello-world/example.jpg">
```


还有其他的一些可以参考[Breaking Changes in Hexo 3.0](https://github.com/hexojs/hexo/wiki/Breaking-Changes-in-Hexo-3.0#render-pipeline-changed)