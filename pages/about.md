---
layout: page
title: About Me
description: 个人简介
keywords: 高语越
comments: true
menu: 关于我
permalink: /about/
---

I'm Gao Yuyue


## 神奇的高先生

* 不为失败找借口，只为成功找方法。
* 平凡生活中乐趣一直在！

## 联系我

* GitHub：[@gaoyuyue](https://github.com/gaoyuyue)
* 博客：[{{ site.title }}]({{ site.url }})
* Email：gaoyuyue@outlook.com

## Skill Keywords

#### Software Language Keywords
<div class="btn-inline">
    {% for keyword in site.skill_language_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Web Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_web_app_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Development Utils Keywords
<div class="btn-inline">
    {% for keyword in site.skill_development_utils_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>
