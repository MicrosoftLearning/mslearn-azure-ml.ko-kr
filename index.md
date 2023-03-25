---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
---

# Microsoft Learn - 실습

다음 실습 연습은 [Microsoft Learn](https://docs.microsoft.com/training/) 교육을 지원하도록 설계되었습니다.

{% assign labs = site.pages | where_exp:"page", "page.url에 '/Instructions'" %}가 포함되어 있습니다.
| |
| --- | --- | 
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
