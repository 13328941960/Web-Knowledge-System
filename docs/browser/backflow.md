# 回流与重绘

回顾一下渲染过程：

![An image](./img/1590632436(1).png)

## 回流

对DOM结构的修改导致几尺寸变化，会发生回流。

### 具体的例子
1. DOM元素的几何属性发生变化：`height`、`width`、`padding`、`margin`、`left`、`top`、`border`

2. 使 DOM 节点发生增减或者移动

3. 读写 offset族、scroll族和client族属性的时候，浏览器为了获取这些值，需要进行回流操作。

4. 调用 window.getComputedStyle 方法。

### 回流过程

依照上面的渲染流水线，触发回流的时候，如果 DOM 结构发生改变，则重新渲染 DOM 树，然后将后面的流程(包括主线程之外的任务)全部走一遍。

![An image](./img/1590632020(1).png)

相当于将解析和合成的过程重新又走了一篇，开销是非常大的

## 重绘

当 DOM 的修改导致了样式的变化，并且没有影响几何属性的时候，会导致**重绘(repaint)**。

由于没有导致 DOM 几何属性的变化，因此元素的位置信息不需要更新，从而省去布局的过程。

### 重绘流程

![An image](./img/1590632105(1).png)

跳过了生成布局树和建图层树的阶段，直接生成绘制列表，然后继续进行分块、生成位图等后面一系列操作。

可以看到，重绘不一定导致回流，但回流一定发生了重绘。

## 合成

例如利用 CSS3 的transform、opacity、filter这些属性就可以实现合成的效果，也就是大家常说的GPU加速。

### GPU加速的原因

在合成的情况下，会直接跳过布局和绘制流程，直接进入非主线程处理的部分，即直接交给合成线程处理。

### 好处

1. 能够充分发挥GPU的优势。合成线程生成位图的过程中会调用线程池，并在其中使用GPU进行加速生成，而GPU 是擅长处理位图数据的。


2. 没有占用主线程的资源，即使主线程卡住了，效果依然能够流畅地展示。


## 实践意义
知道上面的原理之后，对于开发过程有什么指导意义呢？

1. 避免频繁使用 style，而是采用修改class的方式。

2. 使用createDocumentFragment进行批量的 DOM 操作。

3. 对于 resize、scroll 等进行防抖/节流处理。

4. 添加 will-change: tranform ，让渲染引擎为其单独实现一个图层，当这些变换发生时，仅仅只是利用合成线程去处理这些变换，而不牵扯到主线程，大大提高渲染效率。当然这个变化不限于tranform, 任何可以实现合成效果的 CSS 属性都能用will-change来声明。这里有一个实际的例子，一行will-change: tranform拯救一个项目，<https://juejin.im/post/5da52531518825094e373372>。