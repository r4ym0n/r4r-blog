---
title: CodeWar
url: 859.html
id: 859
categories:
  - 未分类
date: 2019-04-24 19:00:17
tags:
---

### 数组的优雅排序和拼接

    &#039;&#039;.join(sorted(str(num), reverse=True))

### 快速进制转换

    int(&quot;&quot;.join(map(str, arr)), 2)

### reduce 的练习

    def namelist(names):
        name_list = [i for x in names for i in x.values()]
        return &#039;&#039; if name_list == [] else reduce(lambda x,y: x+&#039;, &#039;+y if y!=name_list[-1] else x+&#039; &amp; &#039;+y, name_list)