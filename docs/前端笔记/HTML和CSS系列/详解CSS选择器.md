> CSS 选择器用于选取页面元素并应用样式，掌握各种选择器及其用法，能帮助我们更灵活、精准地控制页面样式。

## 1. 基础选择器

基础选择器用于直接选择 HTML 元素，根据元素类型、类名或 ID 来控制样式，是最常用的选择器类型。

### 1.1 类型选择器

类型选择器直接通过 HTML 标签名来选择页面上的元素，如 `p`、`div`、`a` 等。它可以用于批量地控制相同类型元素的样式。

```html
<p>这是一个段落。</p>
<p>这是另一个段落。</p>
```

```css
/* 选择所有 p 元素 */
p {
    color: blue;
}
```

> **注意**：类型选择器一般用于全局样式的统一控制，过多使用可能影响样式的精确度。

### 1.2 类选择器

类选择器用来选择带有指定 `class` 属性的元素。一个元素可以同时包含多个类，这使得类选择器适合复用和组合。

```html
<p class="highlight">这是一个高亮段落。</p>
<p>这是普通段落。</p>
```

```css
/* 选择 class="highlight" 的元素 */
.highlight {
    background-color: yellow;
}
```

### 1.3 ID 选择器

ID 选择器用于选择具有特定 `id` 属性的元素。由于 `id` 在整个页面中应唯一，因此该选择器优先级较高，适用于页面中独特的元素。

```html
<p id="special">这是一个独特的段落。</p>
```

```css
/* 选择 id="special" 的元素 */
#special {
    font-weight: bold;
}
```

> **小提示**：ID 选择器的优先级高于类选择器，尽量避免使用过多的 ID 选择器，以防样式覆盖混乱

### 1.4 通配选择器

通配选择器 `*` 用于选择页面上的所有元素，通常用于全局样式的初始化

```html
<div>容器</div>
<p>段落</p>
```

```css
/* 选择页面上的所有元素 */
* {
    margin: 0;
    padding: 0;
}
```

## 2. 组合选择器

组合选择器用于将多个选择器结合在一起，以实现更精确的样式控制。

### 2.1 后代选择器

后代选择器用于选择嵌套在某元素内的所有特定元素。

```html
<div class="container">
    <p>段落1</p>
    <p>段落2</p>
</div>
```

```css
/* 选择 container 内部的所有 p 元素 */
.container p {
    color: green;
}
```

> **注意**：后代选择器的匹配范围较广，过度使用可能会影响页面性能。

### 2.2 子选择器

子选择器 `>` 用于选择某元素的直接子元素。

```html
<div class="container">
    <p>直接子段落</p>
    <div><p>嵌套段落</p></div>
</div>
```

```css
/* 只选择 container 的直接子 p 元素 */
.container > p {
    color: red;
}
```

### 2.3 相邻兄弟选择器

相邻兄弟选择器 `+` 选择紧邻某元素之后的兄弟元素。

```html
<h2>标题</h2>
<p>紧邻的段落</p>
<p>另一个段落</p>
```

```css
/* 选择紧邻 h2 的第一个 p 元素 */
h2 + p {
    color: purple;
}
```

### 2.4 一般兄弟选择器

一般兄弟选择器 `~` 选择某元素之后的所有兄弟元素。

```html
<h2>标题</h2>
<p>所有后续段落</p>
<p>又一个段落</p>
```

```css
/* 选择 h2 后面的所有 p 元素 */
h2 ~ p {
    color: orange;
}
```

## 3. 属性选择器

属性选择器可以选择具有特定属性或属性值的元素，适合用于动态内容和样式应用场景。

### 3.1 精确匹配选择器 `[attr=value]`

精确匹配选择器选择属性值等于指定值的元素。

```html
<a href="https://example.com">Example</a>
<a href="https://google.com">Google</a>
```

```css
/* 选择 href 属性等于 https://google.com 的链接 */
a[href="https://google.com"] {
    color: blue;
}
```

### 3.2 空格分隔匹配选择器 `[attr~=value]`

空格分隔选择器 `[attr~=value]` 选择属性值包含空格分隔特定值的元素。

```html
<p class="text highlight">内容1</p>
<p class="text">内容2</p>
```

```css
/* 选择 class 属性包含 "highlight" 的元素 */
p[class~="highlight"] {
    background-color: yellow;
}
```

### 3.3 以指定值开头的选择器 `[attr^=value]`

`[attr^=value]` 选择属性值以指定值开头的元素。

```html
<a href="https://example.com">Example</a>
<a href="http://example.com">Another Example</a>
```

```css
/* 选择 href 属性以 "https" 开头的链接 */
a[href^="https"] {
    font-weight: bold;
}
```

### 3.4 以指定值结尾的选择器 `[attr$=value]`

选择属性值以指定值结尾的元素。

```html
<a href="doc.pdf">PDF Document</a>
<a href="doc.docx">Word Document</a>
```

```css
/* 选择 href 属性以 ".pdf" 结尾的链接 */
a[href$=".pdf"] {
    color: red;
}
```

### 3.5 包含特定值的选择器 `[attr*=value]`

选择属性值中包含特定子串的元素。

```html
<a href="https://google.com">Google</a>
<a href="https://example.com">Example</a>
```

```css
/* 选择 href 属性包含 "google" 的链接 */
a[href*="google"] {
    text-decoration: underline;
}
```



## 4. 伪类选择器

伪类用于选择处于特定状态的元素。

### 4.1 hover

选择鼠标悬停时的元素。

```html
<button>悬停按钮</button>
```

```css
button:hover {
    background-color: lightblue;
}
```

### 4.2 nth-child(n)

`:nth-child(n)` 根据子元素在其父元素中的位置来选择元素。与 `:nth-of-type()` 不同的是，它不限定类型，而是按所有子元素的顺序。

```html
<ul>
    <li>项1</li>
    <li>项2</li>
    <li>项3</li>
</ul>
```

```css	
/* 选择 ul 的第二个 li 元素 */
ul li:nth-child(2) {
    color: green;
}
```

### 4.3 first-child 和 last-child

选择父元素的第一个或最后一个子元素。

```css
/* 选择 ul 的第一个和最后一个 li 元素 */
ul li:first-child {
    font-weight: bold;
}
ul li:last-child {
    font-style: italic;
}
```

### 4.4  nth-of-type() 选择器

用于选择特定类型的子元素（如 `p`、`li` 等），并根据索引 `n` 来匹配符合条件的元素。这与 `:nth-child(n)` 不同，后者是按所有子元素的位置选择，而 `:nth-of-type(n)` 只会关注指定类型的子元素。

支持的 `n` 值可以是 `even`、`odd`，或者公式形式如 `2n+1`

```html
<ul>
    <li>第一个列表项</li>
    <li>第二个列表项</li>
    <li>第三个列表项</li>
    <li>第四个列表项</li>
</ul>
```

```css
/* 选择 ul 内第偶数个 li 元素 */
li:nth-of-type(even) {
    color: green;
}
```

### 4.5 is 选择器

`:is()` 让你把多个选择器合并成一个，减少重复书写。它会选中 `:is()` 内所有符合条件的元素，这样就不必重复写每一个选择器。

```css
/* 不用 :is() */
h1, div p {
    color: green;
}

/* 用 :is() */
:is(h1, div p) {
    color: blue;
}

```

> 这两者在样式应用效果上是一样的，但如果同时出现，color 将会是 blue。

在不使用 `:is()` 时，每个选择器的优先级独立计算，且不影响其他选择器。但在 `:is()` 中，整个选择器的优先级会等于 `:is()` 内最高优先级的选择器。这在特定场景中让我们更灵活地控制优先级。

```css
/* 不用 :is() */
h1, .my-class p {
    color: blue;
}

/* 用 :is() */
:is(h1, .my-class p) {
    color: blue;
}

```

- `h1, .my-class p`：`h1` 的优先级低，`.my-class p` 优先级较高。每个选择器是独立的，不会相互影响。
- `:is(h1, .my-class p)`：整体优先级会等于内部优先级最高的选择器（`.my-class p`），因为 `:is()` 会取内部选择器的最高优先级。

### 4.6 where 选择器

`:where()` 和 `:is()` 类似，也可以匹配多个选择器，但不同之处在于 `:where()` 永远具有 **零优先级**。这意味着即便内部选择器包含高优先级的选择器，也不会影响整个规则的优先级，使得它适合用于定义不会干扰其他样式的基础样式。

```html	
<h1>h1标签</h1>
<div>
  <div class="d1">div标签</div>
  <p class="p1">p标签</p>
</div>
<p id="p1">外面的p标签</p>
```

```css
div {
  background-color: aqua;
  color: yellow;
}
p {
  color: red;
}
:where(h1, div p, #p1, div) {
  color: green;
}
```

> 最终颜色:
>  h1 是 green
> .d1 是 yellow
> .p1 是 red

## 5. 伪元素选择器

伪元素用于选择元素的特定部分，例如首字母、首行等。

### 5.1 ::before 和 ::after

在元素内容前后插入内容。

```html
<p>重要通知。</p>
```

```css
p::before {
    content: "⚠️ ";
    color: red;
}
```

### 5.2 ::first-line

选择文本的第一行。

```css
p::first-line {
    font-weight: bold;
}
```

## 6. 选择器优先级和权重

在 CSS 中，不同选择器具有不同的优先级。优先级的计算规则如下：

1. 内联样式优先级最高，权重值为 1000。
2. ID 选择器优先级次之，权重值为 100。
3. 类选择器和属性选择器的权重值为 10。
4. 类型选择器和伪元素选择器的权重值为 1。
5. 通配符选择器 `*` 的优先级最低。