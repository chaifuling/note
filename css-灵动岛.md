# web动画基础

## 1. CSS与JS在动画实现上的边界

随着设备对`css3`的支持度越来越高，在大部分场景上完全能取代js来实现复杂且精美的动画效果。但同时也导致一些人在选择上的困惑: 同样的一个动画场景是使用css3还是js来实现呢？答案是：相互协同，取长补短。

由于js单线程的特性，天生不适合做大量的密集运算，所以用作动画过程的渲染时，常常会出现不流畅的效果。而这恰恰是css3的强项，尤其在给元素添加`translateZ(0)`开启GPU硬件加速后，在动画的绘制性能方面是明显强于JS的。JS作为一门图灵完备的编程语言，它的强项在于对动画流程的控制。比如在实现“灵动岛”动画的连续播放时，纯css3的解决方案是：

```css
.dynamic-island{
  ...  
  animation: 动画1,动画2,动画3;
  ...
}
复制代码
```

但这种仅仅能实现最简单的自动连续播放需求，但是想实现诸如：(1)通过一个点击事件触发播放; (2)整个动画组合循环轮播等等这些稍微复杂点的需求，纯CSS的方案就有局限了。那么这时就必须使用擅长逻辑控制的`js`，配合丰富的动画事件来实现：

```js
   // 灵动岛对应dom
    const box = document.querySelector(".dynamic-island");
   // 以类名定义所有动画类型，以类名切换，实现动画切换   
    const animationList = ["longer", "divide", "fusion", "bigger"];
    box.addEventListener("click", () => {
      box.classList.add(animationList[index]);
    });
    let index = 0;
    // 每一个动画结束都会触发此事件（包括子元素及不同属性动画结束时）
    box.addEventListener("animationend", (e) => {
      if (
        e.animationName === "divide-right" ||
        e.animationName === "fusion-right"
      ) {
        return;
      }
      index++;
      setTimeout(() => {
        if (index <= animationList.length - 1) {
          box.classList.add(animationList[index]);
        } else {
          index = 0;
        }
      }, 800);
    });
复制代码
```

总结：`js`擅长处理对动画的流程控制及基于事件的对整个动画过程的感知，`css3`则在动画渲染的性能及动画关键帧定义的便利性方面更有优势，适合用于动画过程的渲染。

## 2. transition 与 animation 的选择

苹果“灵动岛”的动画，更多的实际上可看作是一种“过渡”动画: 由元素的一种状态向另一种状态的过渡。所以我首先尝试的就是 `transition` 属性，但做出来总感觉差点意思，缺少一种所谓的“灵动感”。在仔细观看官网的动画细节后发现，这些动画在结尾部分常常表现出一种“超出边界继续放大，接着又往回收缩”的类似拉扯橡皮筋的效果，如下GIF所示。

![IMG_3506.GIF](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5291529bae54aa68c826764d327082c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image?)

这是`transiton`无法实现的，所以果断换用`animation`。

~~~css
```
  @keyframes bigger {
    0% {
    }
    60% {
      width: 81vw;
      height: 400px;
      border-radius: 100px;
    }
    80% {
      transform: scaleX(1.04);
    }
    100% {
      width: 81vw;
      height: 400px;
      border-radius: 100px;
      transform: scaleX(1);
    }
  }
```
复制代码
~~~

总结就是：`transition`只适用于元素两个状态间的切换（开始、结束），一旦所需切换状态超过两个，就需要用`animation`的百分比来定义中间的动画帧了。

## 3. JS控制动画播放的三种方式

- 切换class类名 (推荐)

  ```js
     box.classList.toggle('longer');
  复制代码
  ```

- 直接覆盖animation属性

  ```js
     box.style.animation = `longer 800ms ease-in-out`; 
  复制代码
  ```

  缺点是由于动画属性值较长，保存多个动画所需的字符串会较长，不如将动画属性封装在一个个的css类名下，通过切换类名来的简洁方便。

- animationPlayState属性

  ```js
     box.style.animationPlayState="paused" // runing播放，paused暂停。
  复制代码
  ```

  这里有关于此[属性](https://link.juejin.cn?target=https%3A%2F%2Fwww.runoob.com%2Fcssref%2Fcss3-pr-animation-play-state.html)的介绍。但这种方式只适用于控制单个动画的播放状态，但对预期的“灵动岛”多个动画切换的场景，就明显不适用了。

## 3. 非线形动画

IOS系统相比安卓原生采用的[Material-Design](https://link.juejin.cn?target=https%3A%2F%2Fmuse-ui.org%2F%23%2Fzh-CN%2Fpopover) 在动画设计方面最显著的区别，就是大量采用了**非线性动画**。大白话解释就是，动画的速度不是恒定的，可能忽快忽慢。这项功能使用css3实现非常简单，通过定义[CSS3 animation-timing-function 属性](https://link.juejin.cn?target=https%3A%2F%2Fwww.runoob.com%2Fcssref%2Fcss3-pr-animation-timing-function.html)，即可完成。内置的几种属性值基本就可满足大部分需求，笔者采用的是`ease-in-out`慢进慢出的方式，这与苹果官网的效果接近，当然如果你不嫌麻烦，也可以通过自定义`cubic-bezier(n,n,n,n)`贝赛尔曲线函数来量身定制。这里也多说一句：动画的开发中，难的不是技术实现，而是动画细节的调整。快一点、慢一点对开发者来说也许就是一些参数的差别，但对优秀的设计师而言，1px的差异、毫秒级别的快慢，也会影响整体的用户体验，甚至决定整个系统的“气质”。

## 4. 动画结束后如何让元素停留在结束时的状态

css动画结束后，默认不会应用最后动画帧的元素状态，也就是会打回原形，这往往不符合需求，这里提供两种思路。

- `animation-fill-mode:forwards;`（推荐） [属性介绍](https://link.juejin.cn?target=https%3A%2F%2Fwww.runoob.com%2Fcssref%2Fcss3-pr-animation-fill-mode.html)
- js在动画结束时，主动查询一次style属性，并给dom重新赋值一遍 但是因为dom样式的查询会触发提前重绘，所以是极不推荐的方式，只用来处理一些特殊场景。

## 5. translate 与 postion 在实现位移上的区别

位移是最常见的动画场景，这两个属性均可实现。但两者还是有明显区别的，首先`transform: translate` 只是表现层面的位移，并不会实际影响dom的位置，所以它也不会触发重排等影响页面性能的行为。优点当然是性能好，但如果需要在动画过程中即时查询dom的`offsetTop、offsetLeft`等信息，采用`postion`去实现动画会是一个相对更加保险的方案。

# 



作者：郑鱼咚
链接：https://juejin.cn/post/7142412129520812046
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。