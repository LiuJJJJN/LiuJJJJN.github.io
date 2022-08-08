# VueRouter自学笔记




# Vue-Router

> 大连交通大学 信息学院 刘嘉宁
>
> 笔记摘自 表严肃



## 什么是 Vue 路由

- 是一个 Vue 的库，可以快速开发一个单页应用
- 指导网页层级、定位资源
- 避免后端通信整页刷新，做到不丢失状态、不丢失数据



## 使用 Vue-Router

1. 导入 Vue 和 Vue-Router 的 js 文件

```html
<script src="vue.js"></script>
<script src="vue-router..js"></script>
```

2. 定义路由 js 文件

```js
var routes = [
    {
        path:'/',
        component:{
            template:`
            <div>
                <h1>首页</h1>
            </div>
            `
        }
    },
    {
        path:'/about',
        component:{
            template:`
            <div>
                <h1>关于</h1>
            </div>
            `
        }
    }
];

// 这是 Vue-Router 暴露出来的构造器
var router = new VueRouter({
    routes: routes
});

// 创建 Vue 对象,指定 router 路由
new Vue({
    el:'#app',
    router: router
});
```

3. 设定路由链接及显示

```html
    <div id="app">
        <div>
            <!-- 指定对应的路由地址 -->
            <router-link to="/">首页</router-link>
            <router-link to="/about">关于</router-link>
        </div>
        <div>
            <!-- 路由显示的地方 -->
            <router-view></router-view>
        </div>
    </div>
```



## 传参

- 传递参数

```html
            <router-link to="/user/王梓良">王梓良</router-link>
            <router-link to="/user/沈斌">沈斌</router-link>
```

- 获取参数

```js
    {
        // 设置形参 name
        path:'/user/:name',
        component:{
            template:`
            <div>
                <!-- 获取 result风格 传递的参数 -->
                <h1>关于 用户: {{$route.params.name}}</h1>
                <!-- 获取 ?age=18&b=1 传递的参数: http://localhost/web/user/张三?age=18 -->
                <h1>关于 用户: {{$route.query.age}}</h1>
            </div>
            `
        }
    }
```



## 子路由

- 在父级路显示界面由中添加子路由显示界面

```js
        // 设置形参 name
        path:'/user/:name',
        component:{
            template:`
            <div>
                <!-- 子路由方式1 -->
                <!--<router-link to="more" append> more </router-link>-->
                <!--<router-view></router-view>-->
                <!-- 子路由方式2 【推荐】 -->
                <router-link :to="'/user/' + $route.params.name + '/more'"> more </router-link>
                <router-view></router-view>
            </div>
            `
        },
        children:[
            {
                path:'more',
                component:{
                    template:`
                    <div>
                        用户 {{$route.params.name}} 的更多信息
                    </div>
                    `
                }
            }
        ]
```



## 传参

```js
	,
    {
        // 设置形参 name
        path:'/user/:name',
        name:'user',  //为路由指定名字
        component:{
            template:`
            <div>
            </div>
            `
        }
    }

/*======================================================*/

// 创建 Vue 对象,指定 router 路由
new Vue({
    el:'#app',
    router: router,
    methods:{
        surf:function(){
            this.router.push({
                name:"user", // 同 为路由指定的名字
                params:{
                    name:'沈斌'
                }
            })
        }
    }
});
```



## 命名视图

- 可以同时存在多个 `<router-view>`

```js
<router-view name="sidebar"></router-view>
<router-view name="content"></router-view>

/*========================================================*/

	,
    {
        path:'/about',
        component:{
            sidebar:{
                template:`
                <div>
                    <h1>sidebar</h1>
                </div>
          	  	`
			},
            content:{
                template:`
                <div>
                    <h1>content</h1>
                </div>
          	  	`
			}
        }
    }
```



## 导航钩子

- 用来控制访问权限

- 略 
