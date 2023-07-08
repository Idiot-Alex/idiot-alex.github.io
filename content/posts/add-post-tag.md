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

## 2. 调整分类标签的位置



## 3. 固定头部

搞定了分类和标签，但是每篇文章的内容在滚动时会把头部顶上去，我想把头部固定不随着页面滚动而消失。

跟调整分类和标签部分的方法一样，首先找到主题下对应的页面，然后把页面复制到 layouts 目录里面去。

比如我使用的是 paper 主题，就把 themes/paper/layouts/partials/header.html 文件复制到 layouts/partials/header.html，然后在这里面去修改样式就行。

```html
<!-- 修改之前的代码 -->
<header class="mx-auto flex h-[5rem] max-w-3xl px-8 lg:justify-center">
  <!-- 省略其他内容 -->
</header>

<!-- 修改之后的代码 -->
<header class="fixed mx-auto w-screen header-bg border-bottom">
  <div class="mx-auto flex h-[5rem] max-w-3xl px-8 lg:justify-center">
  <!-- 省略其他内容 -->
  </div>
</header>
```

可以看到，我就只是把原来的 header 标签换成了 div 标签，然后外面套了一层 header 标签，给它添加上我需要的样式，比如固定定位、设置宽度、加点边框等。

新增的样式可能不生效，就在 assets 目录下面新增一个 custom.css 文件，然后在这个文件里面写样式内容。下面是一些参考的样式：

```css
/* Place custom css here. */
img {
  border: 0.5rem solid #ebebe5;
  border-radius: 1rem;
  --tw-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --tw-shadow-colored: 0 10px 15px -3px var(--tw-shadow-color), 0 4px 6px -4px var(--tw-shadow-color);
  box-shadow: var(--tw-ring-offset-shadow,0 0 #0000),var(--tw-ring-shadow,0 0 #0000),var(--tw-shadow);
}

article pre {
  margin-left: 0rem;
  margin-right: 0rem;
}

article blockquote {
  margin-left: 0rem !important;
}

.w-screen {
  width: 100vw;
}

.header-bg {
  background: var(--bg);
  z-index: 10;
}

.border-bottom {
  border-bottom: 1px solid #ebebe5;
}

/* main 部分文章内容往下移一点 */
.pt-28 {
  padding-top: 7rem;
}

.min-h-\[calc\(100\%-5rem\)\] {
  min-height: calc(100% - 5rem);
}
```

> 我这里目前调整了头部的定位、背景；给图片添加上了圆角和阴影；给代码块调整了占位宽度（原本的样式会导致代码块不会和段落对齐，感觉很难看）；引用块也是一样。

完成之后的效果如下：

![image-20230708104335852](/Users/zhangxin/Library/Application Support/typora-user-images/image-20230708104335852.png)
