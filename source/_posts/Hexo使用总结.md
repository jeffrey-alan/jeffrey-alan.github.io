---
title: Hexo使用总结
date: 2019-12-09 17:13:09
tags: hexo
categories: 
- web前端
---

# 引子

关于hexo的一些使用方法总结

<!-- more -->


# 初次使用

```
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```

# 安装主题

```
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

## 主题设置

## 站点配置

```
theme: next
```

## 设置中文

```
language: zh-CN
```

## 设置Scheme

```
 scheme: Muse    # 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
# scheme: Mist     # Muse 的紧凑版本，整洁有序的单栏外观
# scheme: Pisces  # 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
# scheme: Gemini  # 类似 Pisces
```

## 设置菜单

```
# 菜单示例配置
menu:
  home: / || home
  reading: /reading/ || book
  archives: /archives/ || archive
  categories: /categories/ || th
  #tags: /tags/ || tags
  about: /about/ || user
```

## 音乐插件
```
<!-- 网易云音乐插件 -->
 <div id="music163player">
      <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=280 height=86 src="//music.163.com/outchain/player?type=2&id=26090155&auto=1&height=66">
      </iframe>
</div>
```
## 文章预览

```
# Automatically Excerpt. Not recommand.
# Please use <!-- more --> in the post to control excerpt accurately.
auto_excerpt:
  enable: false
  length: 150
```



## 设置头像

```
# 将头像放置主题目录下的 source/uploads/ （新建uploads目录若不存在） 配置为：
avatar: /uploads/avatar.png
# 放置在 source/images/ 目录下, 配置为：
avatar: /images/avatar.png
# 完整的互联网 URI
avatar: 
   url: http://example.com/avatar.png
```


## 头像旋转特效

themes\next\source\css\_common\components\sidebar\sidebar-author.styl

```
.site-author-image {
  margin: 0 auto;
  padding: $site-author-image-padding;
  max-width: $site-author-image-width;
  height: $site-author-image-height;
  border: $site-author-image-border-width solid $site-author-image-border-color;

  border-radius: 50%;
  -webkit-border-radius: 50%;
  -moz-border-radius: 50%;
  transition: 1.4s all;
}

.site-author-image:hover {
    -webkit-transform: rotate(360deg);
    -moz-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    -transform: rotate(360deg);
}

.site-author-name {
  margin: $site-author-name-margin;
  text-align: $site-author-name-align;
  color: $site-author-name-color;
  font-weight: $site-author-name-weight;
}

.site-description {
  margin-top: $site-description-margin-top;
  text-align: $site-description-align;
  font-size: $site-description-font-size;
  color: $site-description-color;
}
```

## 本地搜索

npm install hexo-generator-searchdb --save

## 看板娘

next npm install --save hexo-helper-live2d


# 进阶使用

## 添加标签

```
hexo new page tags

title: 添加标签页面测试
tags: Test  #添加标签
categories: Test    #添加分类
comments: false
```

# 博客备份

利用github分支功能进行博客备份，思路说明:

* master分支：存放博客的静态网页(默认分支)。
* hexo分支：存放Hexo博客的源码文件。

## master分支

* 修改更新博客内容并保存。
* 执行hexo clean清除本地旧代码。
* 执行hexo g -d生成静态网站并部署到GitHub的master分支上。

进入站点配置文件编辑，搜索deploy：

```
deploy:
  type: git
  repo: https://github.com/你的github用户名/你的github用户名.github.io.git
  branch: master
```

## hexo 分支
* hexo分支配置hexo分支，该分支为博客源码分支。
* 使用git clone -b hexo 你的github仓库路径， 拷贝源码仓库。
* 修改hexo主配置_config.xml的deploy部分配置，设置静态页面的发布分支为master。
* 添加.gitignore文件，将静态网页的目录及其他无需提交的源文件及目录排除掉。



### 分支设置
* hexo分支，该分支为博客源码分支。
* 使用git clone -b hexo 你的github仓库路径， 拷贝源码仓库。
* 修改hexo主配置_config.xml的deploy部分配置，设置静态页面的发布分支为master。
* 添加.gitignore文件，将静态网页的目录及其他无需提交的源文件及目录排除掉。

### 博客源码更新

```
git checkout hexo
git add .
git commit -m 'Code update'
git push origin hexo
```


## 一键部署脚本

```
#!/bin/bash
DIR=`dirname $0`

# Generate blog
hexo clean
hexo generate
sleep 5

# Deploy
hexo deploy
sleep 5

# Push hexo code
git add .
current_date=`date "+%Y-%m-%d %H:%M:%S"`
git commit -m "Blog updated: $current_date"

sleep 2
git push origin hexo

echo "=====>Finish!<====="
```