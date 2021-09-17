---
title: Vue组件传值
date: 2021-09-15 16:39:47
categories:
    - Vue
tags: [组件,数据]
---


# 前言

作为一个小前端，基础菜的一笔，然后还记忆不太好，为了巩固和记忆，大佬勿喷！！！

**vue的组件传值分为三种方式：父传子、子传父、非父子组件传值引用官网的一句话：父子组件的关系可以总结为 prop 向下传递，事件向上传递父组件通过 prop 给子组件下发数据，子组件通过事件给父组件发送消息，如下图所示：**

![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/technologicalprocess.png)


下面我们就开始用代码（`一言不合就上代码`）详细的介绍vue组件传值的三种方式

### Vue组件传值

1. **父传子**
   子组件的代码：
 
    ```
        <template>
            <div id="container">
                {{msg}}
            </div>
        </template>
        <script>
        export default {
          data() {
            return {};
          },
          props:{
            msg: String
          }
        };
        </script>
        <style scoped>
        #container{
            color: red;
            margin-top: 50px;
        }
        </style>
     ```
   
   父组件的代码：
   
    ```
    <template>
        <div id="container">
            <input type="text" v-model="text" @change="dataChange">
            <Child :msg="text"></Child>
        </div>
    </template>
    <script>
    import Child from "@/components/Child";
    export default {
      data() {
        return {
          text: "父组件的值"
        };
      },
      methods: {
        dataChange(data){
            this.msg = data
        }
      },
      components: {
        Child
      }
    };
    </script>
    <style scoped>
    </style>
    ```

   `父传子的实现方式就是通过props属性，子组件通过props属性接收从父组件传过来的值，而父组件传值的时候使用 v-bind 将子组件中预留的变量名绑定为data里面的数据即可`
   
2. **子传父**

    
    子组件代码：
    
        ```
        <template>
            <div id="container">
                <input type="text" v-model="msg">
                <button @click="setData">传递到父组件</button>
            </div>
        </template>
        <script>
        export default {
          data() {
            return {
              msg: "传递给父组件的值"
            };
          },
          methods: {
            setData() {
              this.$emit("getData", this.msg);
            }
          }
        };
        </script>
        <style scoped>
        #container {
          color: red;
          margin-top: 50px;
        }
        </style>
        ```
    父组件代码：
    
        ```
        <template>
            <div id="container">
                <Child @getData="getData"></Child>
                <p>{{msg}}</p>
            </div>
        </template>
        <script>
        import Child from "@/components/Child";
        export default {
          data() {
            return {
              msg: "父组件默认值"
            };
          },
          methods: {
            getData(data) {
              this.msg = data;
            }
          },
          components: {
            Child
          }
        };
        </script>
        <style scoped>
        </style>
         ```
    `子传父的实现方式就是用了 this.$emit 来遍历 getData 事件，首先用按钮       来触发 setData 事件，在 setData 中 用 this.$emit 来遍历 getData 事           件，最后返回 this.msg`
    
    
    **总结：**

    *     子组件中需要以某种方式例如点击事件的方法来触发一个自定义事件
    *     将需要传的值作为$emit的第二个参数，该值将作为实参传给响应自定义事件的方法
    *     在父组件中注册子组件并在子组件标签上绑定对自定义事件的监听

    
    
3. **非父子** 
    
    `vue 中没有直接子对子传参的方法，建议将需要传递数据的子组件，都合并为一个组件如果一定需要子对子传参，可以先从传到父组件，再传到子组件（相当于一个公共bus文件）为了便于开发，vue 推出了一个状态管理工具 vuex，可以很方便实现组件之间的参数传递`
    
    
### poprs的详解
   
**看一下官方文档：**

* 组件实例的作用域是孤立的。这意味着不能 (也不应该) 在子组件的模板内直接      引用父组件的数据。父组件的数据需要通过 prop 才能下发到子组件中。

* 也就是props是子组件访问父组件数据的唯一接口。


**详细一点解释就是：**

* 一个组件可以直接在模板里面渲染data里面的数据（双大括号）。
* 子组件不能直接在模板里面渲染父元素的数据。
* 如果子组件想要引用父元素的数据，那么就在prop里面声明一个变量（比如a），这个变量就可以引用父元素的数据。然后在模板里渲染这个变量（前面的a），这时候渲染出来的就是父元素里面的数据。

**重点**

`单向数据流： props是单向绑定的`

* 当父组件的属性变化时，将传导给子组件，但是反过来不会。
* 每次父组件更新时，子组件的所有 prop 都会更新为最新值。
* 不要在子组件内部改变 prop。如果你这么做了，Vue 会在控制台给出警告。

`在两种情况下，我们很容易忍不住想去修改 prop 中数据： `

* Prop 作为初始值传入后，子组件想把它当作局部数据来用。
* Prop 作为原始数据传入，由子组件处理成其它数据输出。


对这两种情况，正确的应对方式是
定义一个局部变量，并用 prop 的值初始化它：

    ```
      props: ['initialCounter'], 
        data: function () { 
            return {counter: this.initialCounter }
       }
    ```
 
定义一个计算属性，处理 prop 的值并返回：
     
     ```
        props: ['size'], 
            computed: { normalizedSize: function () { 
            return   this.size.trim().toLowerCase() }
        }
     ```

`注意在 JavaScript 中对象和数组是引用类型，指向同一个内存空间，如果 prop 是一个对象或数组，在子组件内部改变它会影响父组件的状态。`
 
举个例子：

    ```
        <div id="app3">
            <my-component :object='object'></my-component>
        </div>
        <script src="http://vuejs.org/js/vue.min.js"></script>
        <script>
            //
            var mycom = Vue.component('my-component', {
                //添加一个input改变子组件的childOject，那么父元素的object也会被改变，但是Vue没有报错！
                template: '<p>{{ object.name }} is {{ object.age }} years old.<br><input v-model="childObject.name" type="text"></p>',
                props: ['object','school'],
                data: function () {
                    // 子组件的childObject 和 父组件的object 指向同一个对象
                    return {
                        childObject: this.object
                    }
                }
            });
            var app3 = new Vue({
                el: '#app3',
                data: {
                    object:{
                        name: 'Xueying',
                        age: '21',
                    },
                    school:'SCUT',
                },
            })
        </script>

    ```

1. props验证
   可以为prop指定验证规则，如果传入的数据不符合要求，Vue会发出警告。
   

具体验证规则见官方文档：Prop验证规则

2. $parent
    $parent 也可以用来访问父组件的数据。
    

而且子组件可以通过$parent 来直接修改父组件的数据，不会报错！可以使用props的时候，尽量使用props显式地传递数据（可以很清楚很快速地看出子组件引用了父组件的哪些数据）。另外在一方面，直接在子组件中修改父组件的数据是很糟糕的做法，props单向数据流就没有这种顾虑了。





   
    