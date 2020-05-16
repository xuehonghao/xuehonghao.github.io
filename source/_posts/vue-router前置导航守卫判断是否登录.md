---
title: vue-router前置导航守卫判断是否登录
date: 2020-05-16 18:33:36
categories: vue-router
---

src/router/index.js

```js
import VueRouter from 'vue-router'
import Vue from 'vue'

// 导入组件
import Login from '@/views/login'
import Home from '@/views/home'
import Welcome from '@/views/welcome'
import NotFound from '@/views/404'
import auth from '@/utils/auth'
import Article from '@/views/article'
import Image from '@/views/image'
import Publish from '@/views/publish'
import Comment from '@/views/comment'
import Fans from '@/views/fans'
import Setting from '@/views/setting'

Vue.use(VueRouter)

const router = new VueRouter({
  routes: [ // 路由规则
    { path: '/login', component: Login },
    {
      path: '/', component: Home, children: [
        { path: '/', component: Welcome },
        { path: '/article', component: Article },
        { path: '/image', component: Image },
        { path: '/publish', component: Publish },
        { path: '/comment', component: Comment },
        { path: '/fans', component: Fans },
        { path: '/setting', component: Setting }
      ]
    },
    { path: '*', component: NotFound }
  ]
})

// 重点：前置导航守卫
router.beforeEach((to, from, next) => {
  // 如果访问的页面不是登录页面，并且从sessionStorage获取不到token值，就跳到登录页面
  if (to.path !== '/login' && !auth.getUser().token) return next('/login')
  next()
})


export default router
```

src/utils/auth.js

```js
//提供了用户信息(token)操作相关函数
//使用 sessionStorage 存储用户信息
//约定：
//key是 USER_INFO
//value是json字符串
const KEY = 'USER_INFO'
//存储用户信息
const setUser = (user) => {
  window.sessionStorage.setItem(KEY, JSON.stringify(user));
}
//获取用户信息
const getUser = () => {
  //考虑：有可能之前未存储过用户信息，获取到的值null
  //返回：最好是对象，需要转换成对象，所以如果没有获取到值 默认一个 {} 对象即可
  return JSON.parse(window.sessionStorage.getItem(KEY) || '{}')
}
//删除用户信息
const delUser = () => {
  window.sessionStorage.removeItem(KEY)
}
//导出以上三个函数
export default {
  setUser,
  getUser,
  delUser
}
```

------

前置导航守卫总结：

语法：

```js
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

触发时机：导航触发时

参数：

- ​	**`to`**：即将要进入的目标 (路由对象)


- ​	**`from`**：当前导航正要离开的路由


- ​	**`next`**：一定要调用该方法来 **resolve** 这个钩子。执行效果依赖 `next` 方法的调用参数。

**确保要调用 `next` 方法，否则钩子就不会被 resolved。**



