---
title: "给 Hugo Blog 添加分类和标签"
date: 2023-07-05T11:10:45+08:00
tags: ["hugo", "blog"]
categories: ["博客改造"]
draft: true
---

## 0. 序

不久前把自己心心念念的 Blog 搭建起来了，但是不能给文章添加标签和分类，显得不够友好。

搜了下资料，发现 Hugo 可以给文章添加标签和分类，那就把这个功能用起来吧，优化下我的 Blog。

## 1. 开启分类功能

默认情况下，Hugo 就支持分类和标签，就算不填写下面的配置也会生效：

```toml
# 在 hugo.toml 配置文件里面
[taxonomies]
  category = 'categories'
  tag = 'tags'
```

如果想要关闭分类和标签，需要添加下面的配置：

```toml
# 在 hugo.toml 配置文件里面
disableKinds = ['taxonomy', 'term']
```

当开启分类和标签功能之后，还需要在文章中添加对应的信息，像下面这样（在 new-hugo-blog.md 文件中）：

```markdown
---
title: "使用 Hugo 建立自己的 Blog"
date: 2023-07-01T11:25:40+08:00
tags: ["hugo", "blog"]
categories: ["博客改造"]
draft: false
---
```

如果运气好的话，启动 hugo 之后就能在文章内容底部看到设置的标签了。

![image-20230705151914464](https://raw.githubusercontent.com/Idiot-Alex/picgo-repo/main/storage/add-post-tag/202307051519864.png)

为啥说是运气好呢？

是因为这个设置跟我们使用的主题有关系，我目前使用的 paper 主题，从上面的结果上看就只能看到 tags 生效了，但是 categories 并没有生效。

当我们使用的主题不满足我们需求时，就需要考虑自己替换这部分主题的模板了。

在 themes/paper/layouts/_default/single.html 文件里面（paper 换成你自己使用的主题名），我们可以找到配置分类和标签的地方，类似下面这样的代码：

```html
<!-- Post Tags -->
{{ if .Params.tags }}
<footer class="mt-12 flex flex-wrap">
  {{ range .Params.tags }} {{ $href := print (absURL "tags/") (urlize .) }}
  <a
    class="mb-1.5 mr-1.5 rounded-lg bg-black/[3%] px-5 py-1.5 no-underline dark:bg-white/[8%]"
    href="{{ $href }}"
    >{{ . }}</a
  >
  {{ end }}
</footer>
{{ end }}
```

简单的描述下就是把文章中的 tags 属性值遍历输出来展示在页面上，而不显示 categories 的原因也是因为没有类似的代码。

为解决这个问题，我们直接把主题下面的 single.html 文件复制到 layouts/_default/single.html，并且添加一段新代码来展示 categories 属性：

```html
<!-- Post Categories -->
{{ with .Params.categories }}
<ul id="categories">
  {{ range . }}
    <li><a href="{{ "categories" | absURL}}/{{ . | urlize }}">{{ . }}</a> </li>
  {{ end }}
</ul>
{{ end }}
```

重新访问页面，效果如下：

![image-20230705153131536](https://raw.githubusercontent.com/Idiot-Alex/picgo-repo/main/storage/add-post-tag/202307051531290.png)
