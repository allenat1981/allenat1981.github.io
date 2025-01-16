---
layout: single
title: "常用的指令"
permalink: /commands
---

## markdown 文件之間的連結

若要在 markdown 文件中從 A 文件連結到 B 文件，可在 markdown 中使用語法

{% raw %}

```txt
格式：
[連結名稱]({{ site.baseurl }}{% link 目錄/檔名 %})

範例：
[連結到安裝php84]({{ site.baseurl }}{% link _posts/php/2024-12-24-安裝php84.md %})
```

{% endraw %}
