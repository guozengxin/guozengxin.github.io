---
layout: post
title: "django中用javascript发送带有CSRF token的请求"
description: "在前端用javascript语言获取csrf token，去掉CSRF verification failed错误."
category: javascript
tags: [javascript, cookie, getcookie, csrf, csrf token]
comments: true
---

### 事出有因

今天在用django做一个页面时，需要在前端js页面中给后端发一个post请求，但是总会出现"CSRF verification failed. Request aborted."错误，即CSRF验证失败。 [CSRF][]是跨站请求伪造，是一种网络攻击的方式，django中设置这个插件的作用是防止CSRF攻击。

去掉这个错误的一个方法是在settings目录中去掉`django.middleware.csrf.CsrfViewMiddleware`这个中间件。但是如果还想继续使用csrf插件，还想从js中发送post请求，该怎么办呢？

**需要在发送请求时带上csrf token!**

<!-- more -->

### js获取cookie

参考[django csrf的介绍][django-csrf]，其中有用js获取cookie的函数：

#### 直接在js的document对象中获取

```js
// using jQuery
function getCookie(name) {
    var cookieValue = null;
    if (document.cookie && document.cookie != '') {
        var cookies = document.cookie.split(';');
        for (var i = 0; i < cookies.length; i++) {
            var cookie = jQuery.trim(cookies[i]);
            // Does this cookie string begin with the name we want?
            if (cookie.substring(0, name.length + 1) == (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}
var csrftoken = getCookie('csrftoken');
```

`getCookie`函数可以得到当前cookie中对应的key的值（注意这个函数需要用jquery）。

#### 使用jQuery Cookie插件

[jQuery Cookie][]是基于jquery的一个cookie插件，可以对cookie做读、写、删等操作。

调用这个插件，可以用以下的方式获取cookie

```js
var csrftoken = $.cookie('csrftoken');
```

#### 其他方法

1. 复杂函数，增加验证: <http://stackoverflow.com/questions/4003823/javascript-getcookie-functions?answertab=votes#tab-top>
2. w3schools中的介绍: <http://www.w3schools.com/js/js_cookies.asp>

### 在post请求中加入csrf

很简单，在post请求的发送数据中加入一条key是"csrfmiddlewaretoken"，value是上面得到的csrftoken就可以了。像这样：

```js
$.ajax({
    url : url,
    type: "POST",
    data : {csrfmiddlewaretoken: csrftoken},
    dataType : "json",
    success: function( data ){
        // do something
    }
});
```


[CSRF]: http://baike.baidu.com/view/1609487.htm
[django-csrf]: https://docs.djangoproject.com/en/dev/ref/contrib/csrf/
[jQuery Cookie]: http://plugins.jquery.com/cookie/
