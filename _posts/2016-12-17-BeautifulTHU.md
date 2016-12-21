---
layout: post
title:  "BeautifulTHU项目中遇到的问题"
date:   2016-12-17 12:00:00
categories: web
excerpt:
---

* content
{:toc}

## 一

- 导航栏、多组件采用怎样的模式进行组织？

使用props进行数据共享，加上`<router-view :data="data">`

- 如何在router中判断vueapp中ajax的结果？

this.$http returns a promse. simply return it:

```javascript
one_method: function() {
  return this.$http.post('/path').then(function(res) {
    //get response here;
  });
}

// in router:
beforeEnter: function(to, from, next) {
  this.app.one_method().then( function(result) {
    result.whatever ? next() : next(false)
  });
}
```

详见：https://forum.vuejs.org/t/vue-router-beforeenter-access-vue-apps-methods/4056/3

- 登录后通讯录内容需要改变，个人资料又依赖于此。还要考虑到刷新之后处于登录状态但是数据清空了。

舍弃原先的不刷新机制，改为每次点击导航栏都重新从服务器获取，方便info和contact两边修改后及时体现。在当前是contact时，手动调用update。

- role还是app里也存一份吧，不然计算属性里似乎无法检测变动

## 二

- `<a>`这类标签需要设置成`display:block`或`display:inline-block`才可以设置宽度等属性，`inline`的无法通过`margin:0 auto`居中。

- vue-resource已经几乎停止维护更新了，推荐使用新的AJAX库[axios](https://github.com/mzabriskie/axios)，官博：https://medium.com/the-vue-point/retiring-vue-resource-871a82880af4

## 总结

设计API真的是需要经验啊，不然后期很容易迫不得已修改API...
