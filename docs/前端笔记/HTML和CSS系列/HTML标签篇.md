### `<header>`

定义页面或页面中的一个部分的页眉部分，用于标记页面的头部信息通常包含导航、logo、标题等。

**代码示例**：

```html
<header>
  <img src="logo.jpg">
  <h1>我的网站</h1>
 </header>
```

### `<nav>`

定义页面的导航部分，用于包含导航链接，如菜单栏或目录。

**代码示例**：

```html
<nav>
  <ul>
    <li><a href="#">首页</a></li>
    <li><a href="#">服务</a></li>
    <li><a href="#">联系</a></li>
  </ul>
</nav>
```

### `<section>`

定义页面的一个章节或部分，用于划分页面中的内容部分，通常用于标记主题或章节。

**代码示例**：

```html
<section>
  <h2>关于我们</h2>
  <p>这是一段关于我们的文字。</p>
</section>
<section>
  <h2>友情链接</h2>
  <p>这是一段友情链接。</p>
</section>
```

### `<article>`

定义一篇独立的内容，如文章、博客帖子、评论等，适用于可以单独存在或被引用的内容。

**代码示例**：

```html
<article>
  <h2>最新新闻</h2>
  <p>这是一个新闻文章的内容。</p>
</article>
```

### `<aside>`

定义与页面主要内容相关的附属信息，如侧边栏、引用、广告等。用于放置不属于主要内容的相关信息。

**代码示例**：

```html
<aside>
  <h3>推荐文章</h3>
  <ul>
    <li><a href="#">文章一</a></li>
    <li><a href="#">文章二</a></li>
  </ul>
</aside>
```

### `<footer>`

定义页面或部分的页脚部分，用于放置版权信息、联系信息等页脚内容。

**代码示例**：

```html
<footer>
  <p>© xxxx 版权所有.</p>
</footer>
```

### `<figure> 和 <figcaption>`

用于包含图片、图表、代码等，并为其提供说明。`<figure>` 用于封装独立的媒体内容，`<figcaption>` 为其添加描述。

**代码示例**：

```html
<figure>
  <img src="image.jpg" alt="描述图片">
  <figcaption>这是一张描述图片。</figcaption>
</figure>
```

### `<main>`

定义页面的主要内容部分，页面中的核心信息。

**代码示例**：

```html
<main>
  <h1>欢迎来到我的网站</h1>
  <p>这里是主要内容区域。</p>
</main>
```

### `<mark>`

用于突出显示文本，通常是黄色高亮的背景显示，通常用于搜索结果或重点内容。

**代码示例**：

```html
<p>请查看 <mark>重要内容</mark> 部分。</p>
```

### `<em>`

表示强调的文本，通过斜体字（*italic*）来呈现这些被强调的内容。

**代码示例**：

```html
<p>这个功能 <em>非常</em> 重要，请仔细阅读说明。</p>
```

### `<strong>`

用于表示语义上的重要性，通常浏览器会通过加粗（**bold**）来呈现这些被强调的重要内容。在视觉上的加粗效果，和使用CSS来实现（例如 `font-weight: bold;`）一样。

**代码示例**：

```html
<p><strong>注意：</strong> 请确保您已经保存了所有更改。</p>
```

### `<form>`

是 HTML 中用来定义表单的标签，它是表单元素的容器，通常包含用户输入的各种控件，如文本框、复选框、单选按钮、提交按钮等。

**`<form>` 标签的属性**

- `action`：表单提交时，数据将发送到的服务器地址（URL）。
- `method`：表单提交的 HTTP 方法，常用的值有 `GET` 和 `POST`。
- `enctype`：用于指定表单数据的编码类型，当 `method`  为 `POST` 时常用。常见值包括：
  - `application/x-www-form-urlencoded`：默认值，用于键值对的编码。
  - `multipart/form-data`：用于文件上传。
  - `text/plain`：发送未经编码的纯文本。

- `target`：用于指定表单提交后结果的显示方式，如 `_blank`（新窗口）、`_self`（当前窗口）等。
- `autocomplete`：启用或禁用表单的自动完成功能，值可以是 `on` 或 `off`。

**代码示例**：

```html
<form action="/submit-form" method="POST" enctype="multipart/form-data" target="_blank" id="form">
  <label for="name">姓名：</label>
  <input type="text" id="name" name="name">

  <label for="email">邮箱：</label>
  <input type="email" id="email" name="email">

  <input type="submit" value="提交">
  <!--  <button> 元素的默认类型是 submit，所以点击按钮时，和上面的作用一样。-->
  <!-- <button type="button">提交</button> 这样的话点击按钮时，就不会提交 -->
  <button class="btn">提交</button>
  <button type="reset">重置</button>
</form>

<!-- 获取表单值 -->
<script>
  const formEl = document.querySelector('#form')
  const submitEl = document.querySelector('.btn')

  submitEl.addEventListener('click', () => {
    const formData = new FormData(formEl)
    const form = {}

    for (const [key, value] of formData.entries()) {
    	form[key] = value
    }

    console.log(form)
  })
</script>
```

### `<fieldset>` 

用于在 Web 表单中将相关的输入和标签组合在一起。用于将表单中的元素进行分组，通常与 `<legend>` 标签结合使用，以提供分组的标题。这对于创建逻辑分组的表单部分非常有用，特别是当表单较长或包含多个部分时。

`fieldset` 元素是块级元素，这意味着它们出现在新的一行上。

`<fieldset>` 标签的属性：

- `disabled`：将整个字段集禁用，字段集中的所有控件都将被禁用，用户无法与其进行交互。
- `form`：`form` 属性的值应与目标表单的 `id` 属性相匹配，这样即使 `<fieldset>` 不在目标表单内，它仍然会被视为该表单的一部分。
- `name`：字段集的名称（较少使用）。

```html
<form action="/submit-form" method="POST">
  <fieldset>
    <legend>个人信息</legend>
    <label for="name">姓名：</label>
    <input type="text" id="name" name="name">

    <label for="email">邮箱：</label>
    <input type="email" id="email" name="email">
  </fieldset>

  <fieldset>
    <legend>账户信息</legend>
    <label for="username">用户名：</label>
    <input type="text" id="username" name="username">

    <label for="password">密码：</label>
    <input type="password" id="password" name="password">
  </fieldset>

  <input type="submit" value="提交">
</form>
```



### `<time>`

用于标记文档中表示特定时间或日期的部分。

**代码示例**：

```html
<time datetime="2024-08-19">2024年8月19日</time>
```

### `<details> 和 <summary>`

`<details>` 用于展示可展开/收起的内容，`<summary>` 用于显示该内容的标题。用于包含可以显示或隐藏的详细信息。

**代码示例**：

```html
<details>
  <summary>更多信息</summary>
  <p>这是隐藏的详细信息。</p>
</details>
```

### `<address>`

用于表示作者或网站所有者的联系方式。

**代码示例**：

```html
<address>
  联系我们：<a href="mailto:info@example.com">info@example.com</a>
</address>
```

### `<progress>`

显示进度条，常用于展示任务完成情况。

**代码示例**：

```html
<progress value="70" max="100">70%</progress>
```
