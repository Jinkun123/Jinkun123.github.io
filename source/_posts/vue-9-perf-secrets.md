---
title: vue-9-perf-secrets
date: 2021-09-28 14:36:05
categories:
    - Vue
tags: [优化,方法]
---
# 前言

`又是菜菜的我来了，所谓水群就是学习的宗旨，昨天我们伟大的群主大大推荐了一个非常好的项目，经过昨天到今天的研究百度，已经有了大概的了解，并以此感叹大佬真的好厉害。把一些好的知识要点记录，并加深印象。(大佬勿喷！)`



这篇文章主要参考了 Vue.js 核心成员 Guillaume Chau 在 19 年美国的 Vue conf 分享的主题：9 Performance secrets revealed，分享中提到了九个 Vue.js 性能优化的技巧。


这个分享可谓是非常实用了，但是知道和关注的人似乎并不多，到目前为止，该项目也只有可怜的几百个 star。虽然距大佬的分享已经有两年时间，但是其中的优化技巧并没有过时，为了让更多的人了解并学习到其中的实用技巧，我决定对他的分享做二次加工，详细阐述其中的优化原理，并做一定程度的扩展和延伸。


建议你在学习这篇文章的时候可以拉取项目的源码，并且本地运行，查看优化前后的效果差异。


### 接下来就是正文了

#### 1. Functional components

   第一个技巧，函数式组件，你可以查看这个在线示例

   优化前的组件代码如下：
    
 

```
    <template>
          <div class="cell">
            <div v-if="value" class="on"></div>
            <p v-else class="off"></p>
          </div>
        </template>

        <script>
        export default {
          props: ['value'],
        }
        </script>
```



优化后的组件代码如下：
    
  
```
    <template functional>
       <div class="cell">
          <div v-if="props.value" class="on"></div>
          <p v-else class="off"></p>
        </div>
    </template> 
```
  

然后我们在父组件各渲染优化前后的组件 800 个，并在每一帧内部通过修改数据来触发组件的更新，开启 Chrome 的 Performance 面板记录它们的性能，得到如下结果。
   
优化前：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/FunctionalComponentsOff.png)

优化后：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/FunctionalComponentsOn.png)


对比这两张图我们可以看到优化前执行 script 的时间要多于优化后的，而我们知道 JS 引擎是单线程的运行机制，JS 线程会阻塞 UI 线程，所以当脚本执行时间过长，就会阻塞渲染，导致页面卡顿。而优化后的 script 执行时间短，所以它的性能更好。

那么，为什么用函数式组件 JS 的执行时间就变短了呢？这要从函数式组件的实现原理说起了，你可以把它理解成一个函数，它可以根据你传递的上下文数据渲染生成一片 DOM。

函数式组件和普通的对象类型的组件不同，它不会被看作成一个真正的组件，我们知道在 patch 过程中，如果遇到一个节点是组件 vnode，会递归执行子组件的初始化过程；而函数式组件的 render 生成的是普通的 vnode，不会有递归子组件的过程，因此渲染开销会低很多。

因此，函数式组件也不会有状态，不会有响应式数据，生命周期钩子函数这些东西。你可以把它当成把普通组件模板中的一部分 DOM 剥离出来，通过函数的方式渲染出来，是一种在 DOM 层面的复用。
    
#### 2. Child component splitting
第二个技巧，子组件拆分，你可以查看这个在线示例。

优化前的组件代码如下：
```
<template>
  <div :style="{ opacity: number / 300 }">
    <div>{{ heavy() }}</div>
  </div>
</template>
 
<script>
export default {
  props: ['number'],
  methods: {
    heavy () {
      const n = 100000
      let result = 0
      for (let i = 0; i < n; i++) {
        result += Math.sqrt(Math.cos(Math.sin(42)))
      }
      return result
    }
  }
}
</script>
```
优化后的组件代码如下：
```
<template>
  <div :style="{ opacity: number / 300 }">
    <ChildComp/>
  </div>
</template>
 
<script>
export default {
  components: {
    ChildComp: {
      methods: {
        heavy () {
          const n = 100000
          let result = 0
          for (let i = 0; i < n; i++) {
            result += Math.sqrt(Math.cos(Math.sin(42)))
          }
          return result
        },
      },
      render (h) {
        return h('div', this.heavy())
      }
    }
  },
  props: ['number']
}
</script>
```
然后我们在父组件各渲染优化前后的组件 300 个，并在每一帧内部通过修改数据来触发组件的更新，开启 Chrome 的 Performance 面板记录它们的性能，得到如下结果。

优化前：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/ChildComponentSplittingOff.png)

优化后：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/ChildComponentSplittingOn.png)


对比这两张图我们可以看到优化后执行 script 的时间要明显少于优化前的，因此性能体验更好。

那么为什么会有差异呢，我们来看优化前的组件，示例通过一个 heavy 函数模拟了一个耗时的任务，且这个函数在每次渲染的时候都会执行一次，所以每次组件的渲染都会消耗较长的时间执行 JavaScript。

而优化后的方式是把这个耗时任务 heavy 函数的执行逻辑用子组件 ChildComp 封装了，由于 Vue 的更新是组件粒度的，虽然每一帧都通过数据修改导致了父组件的重新渲染，但是 ChildComp 却不会重新渲染，因为它的内部也没有任何响应式数据的变化。所以优化后的组件不会在每次渲染都执行耗时任务，自然执行的 JavaScript 时间就变少了。

不过针对这个优化的方式我提出了一些不同的看法，详情可以点开这个 issue，我认为这个场景下的优化用计算属性要比子组件拆分要好。得益于计算属性自身缓存特性，耗时的逻辑也只会在第一次渲染的时候执行，而且使用计算属性也没有额外渲染子组件的开销。

在实际工作中，使用计算属性是优化性能的场景会有很多，毕竟它也体现了一种空间换时间的优化思想。

#### 3. Local variables

第三个技巧，局部变量。

优化前的组件代码如下：

```
    <template>
      <div :style="{ opacity: start / 300 }">{{ result }}</div>
    </template>

    <script>
    export default {
      props: ['start'],
      computed: {
        base () {
          return 42
        },
        result () {
          let result = this.start
          for (let i = 0; i < 1000; i++) {
            result += Math.sqrt(Math.cos(Math.sin(this.base))) + this.base * this.base + this.base + this.base * 2 + this.base * 3
          }
          return result
        },
      },
    }
    </script>
```

优化后的组件代码如下：
```
    <template>
      <div :style="{ opacity: start / 300 }">{{ result }}</div>
    </template>

    <script>
    export default {
      props: ['start'],
      computed: {
        base () {
          return 42
        },
        result ({ base, start }) {
          let result = start
          for (let i = 0; i < 1000; i++) {
            result += Math.sqrt(Math.cos(Math.sin(base))) + base * base + base + base * 2 + base * 3
          }
          return result
        },
      },
    }
    </script>
```
然后我们在父组件各渲染优化前后的组件 300 个，并在每一帧内部通过修改数据来触发组件的更新，开启 Chrome 的 Performance 面板记录它们的性能，得到如下结果。

优化前：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/LocalVariablesOff.png)

优化后：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/LocalVariablesOn.png)


对比这两张图我们可以看到优化后执行 script 的时间要明显少于优化前的，因此性能体验更好。

这里主要是优化前后组件的计算属性 result 的实现差异，优化前的组件多次在计算过程中访问 this.base，而优化后的组件会在计算前先用局部变量 base 缓存 this.base，后面则直接访问 base变量。

那么为啥这个差异会造成性能上的差异呢，原因是你每次访问 this.base 的时候，由于 this.base 是一个响应式对象，所以会触发它的 getter，进而会执行依赖收集相关逻辑代码。类似的逻辑执行多了，像示例这样，几百次循环更新几百个组件，每个组件触发 computed 重新计算，然后又多次执行依赖收集相关逻辑，性能自然就下降了。

从需求上来说，this.base 执行一次依赖收集就够了，因此我们只需要把它的 getter 求值结果返回给局部变量 base，后续再次访问 base 的时候就不会触发 getter，也不会走依赖收集的逻辑了，性能自然就得到了提升。

这是一个非常实用的性能优化技巧。因为很多人在开发 Vue.js 项目的时候，每当取变量的时候就习惯性直接写 this.xxx 了，因为大部分人并不会注意到访问 this.xxx 背后做的事情。在访问次数不多的时候，性能问题并没有凸显，但是一旦访问次数变多，比如在一个大循环中多次访问，类似示例这种场景，就会产生性能问题了。

我之前给 ZoomUI 的 Table 组件做性能优化的时候，在 render table body 的时候就使用了局部变量的优化技巧，并写了 benchmark 做性能对比：渲染 1000 * 10 的表格，ZoomUI Table 的更新数据重新渲染的性能要比 ElementUI 的 Table 性能提升了近一倍。


#### 4. Reuse DOM with v-show

第四个技巧，使用 v-show 复用 DOM，你可以查看这个在线示例。

优化前的组件代码如下：

```
<template functional>
  <div class="cell">
    <div v-if="props.value" class="on">
      <Heavy :n="10000"/>
    </div>
    <p v-else class="off">
      <Heavy :n="10000"/>
    </p>
  </div>
</template>
```
 
 优化后的组件代码如下：
 
```
 <template functional>
  <div class="cell">
    <div v-show="props.value" class="on">
      <Heavy :n="10000"/>
    </div>
    <p v-show="!props.value" class="off">
      <Heavy :n="10000"/>
    </p>
  </div>
</template>
```

然后我们在父组件各渲染优化前后的组件 200 个，并在每一帧内部通过修改数据来触发组件的更新，开启 Chrome 的 Performance 面板记录它们的性能，得到如下结果。

优化前：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/DomShowOff.png)


优化后：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/DomShowOn.png)


对比这两张图我们可以看到优化后执行 script 的时间要明显少于优化前的，因此性能体验更好。

优化前后的主要区别是用 v-show 指令替代了 v-if 指令来替代组件的显隐，虽然从表现上看，v-show 和 v-if 类似，都是控制组件的显隐，但内部实现差距还是很大的。

v-if 指令在编译阶段就会编译成一个三元运算符，条件渲染，比如优化前的组件模板经过编译后生成如下渲染函数：

```
    function render() {
      with(this) {
        return _c('div', {
          staticClass: "cell"
        }, [(props.value) ? _c('div', {
          staticClass: "on"
        }, [_c('Heavy', {
          attrs: {
            "n": 10000
          }
        })], 1) : _c('p', {
          staticClass: "off"
        }, [_c('Heavy', {
          attrs: {
            "n": 10000
          }
        })], 1)])
      }
    }
```

当条件 props.value 的值变化的时候，会触发对应的组件更新，对于 v-if 渲染的节点，由于新旧节点 vnode 不一致，在核心 diff 算法比对过程中，会移除旧的 vnode 节点，创建新的 vnode 节点，那么就会创建新的 Heavy 组件，又会经历 Heavy 组件自身初始化、渲染 vnode、patch 等过程。

因此使用 v-if 每次更新组件都会创建新的 Heavy 子组件，当更新的组件多了，自然就会造成性能压力。

而当我们使用 v-show 指令，优化后的组件模板经过编译后生成如下渲染函数：

```
    function render() {
      with(this) {
        return _c('div', {
          staticClass: "cell"
        }, [_c('div', {
          directives: [{
            name: "show",
            rawName: "v-show",
            value: (props.value),
            expression: "props.value"
          }],
          staticClass: "on"
        }, [_c('Heavy', {
          attrs: {
            "n": 10000
          }
        })], 1), _c('p', {
          directives: [{
            name: "show",
            rawName: "v-show",
            value: (!props.value),
            expression: "!props.value"
          }],
          staticClass: "off"
        }, [_c('Heavy', {
          attrs: {
            "n": 10000
          }
        })], 1)])
      }
    }
```

当条件 props.value 的值变化的时候，会触发对应的组件更新，对于 v-show 渲染的节点，由于新旧 vnode 一致，它们只需要一直 patchVnode 即可，那么它又是怎么让 DOM 节点显示和隐藏的呢？

原来在 patchVnode 过程中，内部会对执行 v-show 指令对应的钩子函数 update，然后它会根据 v-show 指令绑定的值来设置它作用的 DOM 元素的 style.display 的值控制显隐。

因此相比于 v-if 不断删除和创建函数新的 DOM，v-show 仅仅是在更新现有 DOM 的显隐值，所以 v-show 的开销要比 v-if 小的多，当其内部 DOM 结构越复杂，性能的差异就会越大。

但是 v-show 相比于 v-if 的性能优势是在组件的更新阶段，如果仅仅是在初始化阶段，v-if 性能还要高于 v-show，原因是在于它仅仅会渲染一个分支，而 v-show 把两个分支都渲染了，通过 style.display 来控制对应 DOM 的显隐。

在使用 v-show 的时候，所有分支内部的组件都会渲染，对应的生命周期钩子函数都会执行，而使用 v-if 的时候，没有命中的分支内部的组件是不会渲染的，对应的生命周期钩子函数都不会执行。

因此你要搞清楚它们的原理以及差异，才能在不同的场景使用适合的指令。

总结一下，用自己的话来说就是：v-if操作虚拟DOM，v-show是CSS的display；所以v-if有局部卸载添加节点 更耗费性能，而v-show并没有删除所以性能更优，
另外补充一下 v-if更适用于首次加载之后改变次数较少，v-show适合频繁切换。



#### 5. KeepAlive

第五个技巧，使用 KeepAlive 组件缓存 DOM，你可以查看这个在线示例。

优化前的组件代码如下：
```
<template>
  <div id="app">
    <router-view/>
  </div>
</template>
```
优化后的组件代码如下：
```
<template>
  <div id="app">
    <keep-alive>
      <router-view/>
    </keep-alive>
  </div>
</template>
```

我们点击按钮在 Simple page 和 Heavy Page 之间切换，会渲染不同的视图，其中 Heavy Page 的渲染非常耗时。我们开启 Chrome 的 Performance 面板记录它们的性能，然后分别在优化前后执行如上的操作，会得到如下结果。

优化前：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/KeepAliveOff.png)

优化后：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/KeepAliveNo.png)


对比这两张图我们可以看到优化后执行 script 的时间要明显少于优化前的，因此性能体验更好。

在非优化场景下，我们每次点击按钮切换路由视图，都会重新渲染一次组件，渲染组件就会经过组件初始化，render、patch 等过程，如果组件比较复杂，或者嵌套较深，那么整个渲染耗时就会很长。

而在使用 KeepAlive 后，被 KeepAlive 包裹的组件在经过第一次渲染后，的 vnode 以及 DOM 都会被缓存起来，然后再下一次再次渲染该组件的时候，直接从缓存中拿到对应的 vnode 和 DOM，然后渲染，并不需要再走一次组件初始化，render 和 patch 等一系列流程，减少了 script 的执行时间，性能更好。

但是使用 KeepAlive 组件并非没有成本，因为它会占用更多的内存去做缓存，这是一种典型的空间换时间优化思想的应用。


#### 6. Deferred features

第六个技巧，使用 Deferred 组件延时分批渲染组件。

优化前的组件代码如下：
```
<template>
  <div class="deferred-off">
    <VueIcon icon="fitness_center" class="gigantic"/>
 
    <h2>I'm an heavy page</h2>
    <Heavy v-for="n in 8" :key="n"/>
    <Heavy class="super-heavy" :n="9999999"/>
  </div>
</template>
```
优化后的组件代码如下：
```
<template>
  <div class="deferred-on">
    <VueIcon icon="fitness_center" class="gigantic"/>
 
    <h2>I'm an heavy page</h2>
    <template v-if="defer(2)">
      <Heavy v-for="n in 8" :key="n"/>
    </template>
    <Heavy v-if="defer(3)" class="super-heavy" :n="9999999"/>
  </div>
</template>
<script>
import Defer from '@/mixins/Defer'
export default {
  mixins: [
    Defer(),
  ],
}
</script>
```

我们点击按钮在 Simple page 和 Heavy Page 之间切换，会渲染不同的视图，其中 Heavy Page 的渲染非常耗时。我们开启 Chrome 的 Performance 面板记录它们的性能，然后分别在优化前后执行如上的操作，会得到如下结果。

优化前：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/DeferredFeaturesOff.png)

对比这两张图我们可以发现，优化前当我们从 Simple Page 切到 Heavy Page 的时候，在一次 Render 接近结尾的时候，页面渲染的仍然是 Simple Page，会给人一种页面卡顿的感觉。而优化后当我们从 Simple Page 切到 Heavy Page 的时候，在一次 Render 靠前的位置页面就已经渲染了 Heavy Page 了，并且 Heavy Page 是渐进式渲染出来的。

优化前后的差距主要是后者使用了 Defer 这个 mixin，那么它具体是怎么工作的，我们来一探究竟：

```
    export default function (count = 10) {
      return {
        data () {
          return {
            displayPriority: 0
          }
        },

        mounted () {
          this.runDisplayPriority()
        },

        methods: {
          runDisplayPriority () {
            const step = () => {
              requestAnimationFrame(() => {
                this.displayPriority++
                if (this.displayPriority < count) {
                  step()
                }
              })
            }
            step()
          },

          defer (priority) {
            return this.displayPriority >= priority
          }
        }
      }
    }
```

Defer 的主要思想就是把一个组件的一次渲染拆成多次，它内部维护了 displayPriority 变量，然后在通过 requestAnimationFrame 在每一帧渲染的时候自增，最多加到 count。然后使用 Defer mixin 的组件内部就可以通过 v-if="defer(xxx)" 的方式来控制在 displayPriority 增加到 xxx 的时候渲染某些区块了。

当你有渲染耗时的组件，使用 Deferred 做渐进式渲染是不错的注意，它能避免一次 render 由于 JS 执行时间过长导致渲染卡住的现象。


#### 7. Time slicing
第七个技巧，使用 Time slicing 时间片切割技术。

优化前的代码如下：

```
fetchItems ({ commit }, { items }) {
  commit('clearItems')
  commit('addItems', items)
}
```

优化后的代码如下：
```
fetchItems ({ commit }, { items, splitCount }) {
  commit('clearItems')
  const queue = new JobQueue()
  splitArray(items, splitCount).forEach(
    chunk => queue.addJob(done => {
      // 分时间片提交数据
      requestAnimationFrame(() => {
        commit('addItems', chunk)
        done()
      })
    })
  )
  await queue.start()
}
```

我们先通过点击 Genterate items 按钮创建 10000 条假数据，然后分别在开启和关闭 Time-slicing 的情况下点击 Commit items 按钮提交数据，开启 Chrome 的 Performance 面板记录它们的性能，会得到如下结果。

优化前：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/TimeSlicingOff.png)

优化后：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/TimeSlicingOn.png)


对比这两张图我们可以发现，优化前总的 script 执行时间要比优化后的还要少一些，但是从实际的观感上看，优化前点击提交按钮，页面会卡死 1.2 秒左右，在优化后，页面不会完全卡死，但仍然会有渲染卡顿的感觉。

那么为什么在优化前页面会卡死呢？因为一次性提交的数据过多，内部 JS 执行时间过长，阻塞了 UI 线程，导致页面卡死。

优化后，页面仍有卡顿，是因为我们拆分数据的粒度是 1000 条，这种情况下，重新渲染组件仍然有压力，我们观察 fps 只有十几，会有卡顿感。通常只要让页面的 fps 达到 60，页面就会非常流畅，如果我们把数据拆分粒度变成 100 条，基本上 fps 能达到 50 以上，虽然页面渲染变流畅了，但是完成 10000 条数据总的提交时间还是变长了。

使用 Time slicing技术可以避免页面卡死，通常我们在这种耗时任务处理的时候会加一个 loading 效果，在这个示例中，我们可以开启 loading animation，然后提交数据。对比发现，优化前由于一次性提交数据过多，JS 一直长时间运行，阻塞 UI 线程，这个 loading 动画是不会展示的，而优化后，由于我们拆成多个时间片去提交数据，单次 JS 运行时间变短了，这样 loading 动画就有机会展示了。


`这里要注意的一点，虽然我们拆时间片使用了 requestAnimationFrame API，但是使用 requestAnimationFrame 本身是不能保证满帧运行的，requestAnimationFrame 保证的是在浏览器每一次重绘后会执行对应传入的回调函数，想要保证满帧，只能让 JS 在一个 Tick 内的运行时间不超过 17ms。`


#### 8. Non-reactive data

第八个技巧，使用 Non-reactive data ，你可以查看这个在线示例。

优化前代码如下：
```
const data = items.map(
  item => ({
    id: uid++,
    data: item,
    vote: 0
  })
)
```

优化后代码如下：

```
const data = items.map(
  item => optimizeItem(item)
)
 
function optimizeItem (item) {
  const itemData = {
    id: uid++,
    vote: 0
  }
  Object.defineProperty(itemData, 'data', {
    // Mark as non-reactive
    configurable: false,
    value: item
  })
  return itemData
}
```


还是前面的示例，我们先通过点击 Genterate items 按钮创建 10000 条假数据，然后分别在开启和关闭 Partial reactivity 的情况下点击 Commit items 按钮提交数据，开启 Chrome 的 Performance 面板记录它们的性能，会得到如下结果。

优化前：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/Non-reactiveOff.png)

优化后：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/Non-reactiveOn.png)


对比这两张图我们可以看到优化后执行 script 的时间要明显少于优化前的，因此性能体验更好。

之所以有这种差异，是因为内部提交的数据的时候，会默认把新提交的数据也定义成响应式，如果数据的子属性是对象形式，还会递归让子属性也变成响应式，因此当提交数据很多的时候，这个过程就变成了一个耗时过程。

而优化后我们把新提交的数据中的对象属性 data 手动变成了 configurable 为 false，这样内部在 walk 时通过 Object.keys(obj) 获取对象属性数组会忽略 data，也就不会为 data 这个属性 defineReactive，由于 data 指向的是一个对象，这样也就会减少递归响应式的逻辑，相当于减少了这部分的性能损耗。数据量越大，这种优化的效果就会更明显。

其实类似这种优化的方式还有很多，比如我们在组件中定义的一些数据，也不一定都要在 data 中定义。有些数据我们并不是用在模板中，也不需要监听它的变化，只是想在组件的上下文中共享这个数据，这个时候我们可以仅仅把这个数据挂载到组件实例 this 上，例如：

```
export default {
  created() {
    this.scroll = null
  },
  mounted() {
    this.scroll = new BScroll(this.$el)
  }
}
```
这样我们就可以在组件上下文中共享 scroll 对象了，即使它不是一个响应式对象。

#### 9. Virtual scrolling

第九个技巧，使用 Virtual scrolling。

优化前组件的代码如下：

```
<div class="items no-v">
  <FetchItemViewFunctional
    v-for="item of items"
    :key="item.id"
    :item="item"
    @vote="voteItem(item)"
  />
</div>
```

优化后代码如下：

```
<recycle-scroller
  class="items"
  :items="items"
  :item-size="24"
>
  <template v-slot="{ item }">
    <FetchItemView
      :item="item"
      @vote="voteItem(item)"
    />
  </template>
</recycle-scroller>
```

还是前面的示例，我们需要开启 View list，然后点击 Genterate items 按钮创建 10000 条假数据（注意，线上示例最多只能创建 1000 条数据，实际上 1000 条数据并不能很好地体现优化的效果，所以我修改了源码的限制，本地运行，创建了 10000 条数据），然后分别在 Unoptimized 和 RecycleScroller 的情况下点击 Commit items 按钮提交数据，滚动页面，开启 Chrome 的 Performance 面板记录它们的性能，会得到如下结果。

优化前：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/VirtualScrollingOff.png)

优化后：
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/VirtualScrollingOn.png)

对比这两张图我们发现，在非优化的情况下，10000 条数据在滚动情况下 fps 只有个位数，在非滚动情况下也就十几，原因是非优化场景下渲染的 DOM 太多，渲染本身的压力很大。优化后，即使 10000 条数据，在滚动情况下的 fps 也能有 30 多，在非滚动情况下可以达到 60 满帧。

之所以有这个差异，是因为虚拟滚动的实现方式：是只渲染视口内的 DOM。这样总共渲染的 DOM 数量就很少了，自然性能就会好很多。

虚拟滚动组件也是 Guillaume Chau 写的，感兴趣的同学可以去研究它的源码实现。它的基本原理就是监听滚动事件，动态更新需要显示的 DOM 元素，计算出它们在视图中的位移。

虚拟滚动组件也并非没有成本，因为它需要在滚动的过程中实时去计算，所以会有一定的 script 执行的成本。因此如果列表的数据量不是很大的情况，我们使用普通的滚动就足够了。


### 总结

通过这篇文章，我希望你能了解到 Vue.js 的九种性能优化技巧，并能运用到实际的开发项目中。除了上述技巧之外，还有懒加载图片、懒加载组件、异步组件等等常用的性能优化手段。

在做性能优化前，我们需要分析性能的瓶颈在哪，才能因地制宜。另外，性能优化都需要数据支撑的，你在做任何性能优化前，需要先采集优化前的数据，这样优化后才能够通过数据对比看到优化的效果。

希望你在日后的开发过程中，不再只满足于实现需求，写每一行代码的时候，都能思考它可能产生的性能方面的影响。

### 参考资料

* vue-9-perf-secrets slidse：https://slides.com/akryum/vueconfus-2019
* vue-9-perf-secrets 分享演讲视频：https://www.vuemastery.com/conferences/vueconf-us-2019/9-performance-secrets-revealed/
* vue-9-perf-secrets 项目源码：https://github.com/Akryum/vue-9-perf-secrets
* vue-9-perf-secrets 在线演示地址：https://vue-9-perf-secrets.netlify.app/
* vue-9-perf-secrets 讨论 issue：https://github.com/Akryum/vue-9-perf-secrets/issues/1
* vue-virtual-scroller 项目源码：https://github.com/Akryum/vue-virtual-scroller




