---
title: 小程序wx.navigateTo中的闭包
comments: true
banner_img_height: 30
date: 2023-11-02 18:04:38
categories:
- 小程序
tags:
---

# 序
最近的开发任务中处理了这样一个情况:

1. 打开小程序是从onLoad判断是否携带参数
2. 判断用户是否登录
3. 如果没有登录需要跳转登录并跳转回来
4. 回来后继续使用(1)中携带的参数处理问题

听起来没有什么难点，我们只需要设置`app.globleData`就可以很轻松的完成需求。但是我思考问题的点在于登录环节：如果每次有类似的需求都要去改动登录页的代码，那会让登录页面的`if`条件越来越多，代码可读性也会持续下降。

于是我开始查找对登录页面修改最少的解决方案：
- 使用`wx.router`来完成需求，但是我不是专业写前端的，而是被拉来救急的，让我去看文档写代码是在太慢了。而且如果整个小程序都改用`wx.router`，这其中的修改成本也太高了。
- 使用`app.globleData`前面说了，也不太可取
- 最终我发现了`wx.navigateTo`中的`success`可以处理我的问题。

所以在这里将我的思路写下来做记录，也提供给有同样需求的人参考。

# 还是要用到`app.globleData`

是的，我的方式中还是要对`app.globleData`进行设置，但是约定了统一的格式，无论在哪个页面。
```js
var pages = getCurrentPages()
var curPage = pages[pages.length-1]

app.globleData.navigateInfo = {
    url:"/"+curPage.route,
    success: function (e) {
        var page = getCurrentPages().pop()
        if (page == undefined || page == null) return;
        page.selectComponent("#coupon").runCoupon();
    }
}
```
在登录页面只需要判断`app.globleData.navigateInfo`这一个参数，并在完成登录操作后直接将`navigateInfo对象`交给`wx.navigateTo`方法执行就跳回了原来的页面。

这里面神奇的点在于可以直接执行组件中的某个方法

