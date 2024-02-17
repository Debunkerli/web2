---
title: 玩转hexo # 文章标题  
date: 2023/1/22 22:00:00  # 文章发表时间
tags:
thumbnail: https://lixudongself.eu.org/hexo封面.png # 略缩图
---


## 1.添加和删除导航栏的快捷图标

![](https://lixudongself.eu.org/hexo_image/post%20(1).png)

Miccall 主题内涵盖有常用的图标位置在`...\web1\themes\miccall\source\css`目录下`font-awesome.min.css`文件内

![](https://lixudongself.eu.org/hexo_image/post%20(2).png)



格式如上，仅需要将`onedrive`位置更换为需要的名称，图标在http://fontawesome.io/icons/#brand该网站获取

在搜索到想要的icon之后

![](https://lixudongself.eu.org/hexo_image/post%20(3).png)

将`“f0a0”`填入`content`

接下来来到`...\web1\themes\miccall\_config.yml`

![](https://lixudongself.eu.org/hexo_image/post%20(4).png)

在已有的元素后添加刚才设定的名称 并按照格式粘贴链接即可

## 2.开始创作文章

文章目录`web1/source/_post`下新建一个`.md`文件：

```
---
title: # 文章标题  
date: 2017/3/27 13:48:25  # 文章发表时间
tags:
- 标签1
- 标签2 (可选)
categories: Algorithm # 分类
thumbnail: https://xxxxxxxxxx.png # 略缩图
---

文章正文
```

