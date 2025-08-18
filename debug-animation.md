```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <style>
      h1 > span {
        opacity: 0;
        transform: translateY(-100%);
        transition: transform 500ms ease-in, opacity 0ms 500ms;
        display: inline-block;
      }
      h1:hover > span {
        opacity: 1;
        transform: translateY(0);
        transition: transform 500ms ease-out;
      }

      ul > li {
        position: relative;
      }

      .container {
        margin: 0 auto;
        height: 90vh;
        position: relative;
      }

      .box {
        width: 100px;
        height: 100px;
        position: absolute;
        top: 10px;
        left: 10px;
        animation: move 3s ease infinite;
      }

      @keyframes move {
        50% {
          transform: translate(calc(90vw - 200px), calc(90vh - 160px));
        }
      }
    </style>
  </head>
  <body>

    <div>
      <h1>C<span>icada</span></h1>
      <ul>
        <li>...</li>
        <li>... </li>
        <li>...</li>
      </ul>
    </div>
    <div class="container">
      <div class="box"></div>
    </div>
  </body>
</html>
```

## 问题现象

当鼠标悬停在 `h1` 元素上时，触发 `span` 元素的过渡动画效果，会意外地引起 `ul` 和 `container` 元素的重绘（repaint）。

## 问题分析

### 堆叠上下文规则

根据 CSS 规范，当没有明确指定 `z-index` 属性时，元素的堆叠顺序遵循以下规则（从底层到顶层）：

1. 根元素的背景和边框
2. 后代非定位元素，按照在 HTML 中的出现顺序
3. 后代定位元素，按照在 HTML 中的出现顺序

> 参考：[Stacking without the z-index property](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout/Stacking_without_z-index?utm_source=chatgpt.com)

### z-index 属性的作用范围

`z-index` CSS 属性用于设置**定位元素**及其后代元素，或者 **flex** 和 **grid** 项目的 z 轴层级顺序。

> 参考：[Z-index](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index)

### 当前示例的堆叠顺序

在当前代码中，元素的堆叠顺序为（从下到上）：
1. `div`、`span`（非定位元素）
2. `li`、`.container`、`.box`（定位元素）

**Stacking context：** `span` 和 `.box` 元素

![堆叠顺序示意图](/images/staking.png)

### 问题根源

当我们通过各种方式创建更多的堆叠上下文时，在渲染初始变换时，不仅会重绘目标 `div` 元素，还会重绘堆叠上下文中位于其上方的所有元素。

## 解决方案

1. 
```css
h1 > span {
  position: relative;
  z-index: 1;
}
```

2. 在父元素中设置 `overflow: hidden;` 也可解决，猜测与 Block formatting context 有关
```css
h1 {
  overflow: hidden;
}
```