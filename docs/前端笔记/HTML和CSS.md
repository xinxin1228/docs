## 常见的效果

### 设置全局 css 变量

```css
// 变量声明
:root {
  --primary-color: #5296d5;
  --empty-color: #e0e0e0;
}

// 使用变量
div {
  color: var(--primary-color);
}
```

### 骨架屏加载

```css
:root {
  --skeleton-default-color: #f6f7f8;
  --skeleton-active-color: #eeeff1;
}

// 骨架屏水平从左往右
.skeleton-public {
  width: 100%;
  border-radius: 8px;
  background-image: linear-gradient(
    to right,
    var(--skeleton-default-color) 0%,
    var(--skeleton-active-color) 10%,
    var(--skeleton-default-color) 20%,
    var(--skeleton-default-color) 100%
  );
  background-size: 200% 100%;
  animation: skeleton2 1s linear infinite;
  background-position-x: 50%;
}

@keyframes skeleton2 {
  to {
    background-position-x: -150%;
  }
}

// 骨架屏斜着从左往右
.skeleton-public {
  width: 100%;
  border-radius: 8px;
  background: linear-gradient(
    100deg,
    var(--skeleton-default-color) 40%,
    var(--skeleton-active-color) 50%,
    var(--skeleton-default-color) 60%
  );
  background-size: 200% 100%;
  animation: skeleton 1s linear infinite;
  background-position-x: 180%;
}

@keyframes skeleton {
  to {
    background-position-x: -20%;
  }
}
```

### 平行四边形

```css
.box {
  transform: skewY(45deg);
}
```

### 英文字母大小写

```css
// 英文字母全部大写
text-transform: uppercase;

// 英文字母全部小写
text-transform: lowercase;

// 首字母大写
text-transform: capitalize；;
```

### 动画延迟执行

```css
transition-delay: 0.4s;

// 适用于transition类型的动画
```

### 设置动画旋转中心

```css
transform-origin: top left; // 改为左上为旋转中心点

// 默认是 center center
```

### 动画结束后停留在动画结束的位置

```css
// 设置 forwards 参数可以使动画停留在结束的位置
animation: selected 0.3s linear forwards;
```

### css 选择除最后一个以外的全部元素

```css
li {
  &:not(:last-of-type) {
    border-bottom: 1px solid #eee;
  }
}
```

### less中引用自己

```less
body{
  li{
    // 定义一个变量 方便数值修改
    @height:40px;
    .liss{
      color:red;
      height:@height;
      line-height:@height;
    }
    &.active:{
      color:green;
    }
  }
  
  // 如果想给li 添加一个active状态 只需要这样写
}
```

### 自定义滚动条样式

```js
//滚动条样式
::-webkit-scrollbar {/*滚动条整体样式*/
  width: 5px;     /*高宽分别对应横竖滚动条的尺寸*/
  height: 1px;
}
::-webkit-scrollbar-thumb {/*滚动条里面小方块*/
  border-radius: 5px;
  -webkit-box-shadow: inset 0 0 5px rgba(0,0,0,0.2);
  background: rgba(0,167,232,.3);
}
::-webkit-scrollbar-track {/*滚动条里面轨道*/
  -webkit-box-shadow: inset 0 0 5px rgba(0,0,0,0.2);
  border-radius: 10px;
  background: #EDEDED;
}
```
