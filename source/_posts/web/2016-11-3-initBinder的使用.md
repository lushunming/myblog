---
layout: post
title: Spring中initBinder的使用
description: InitBinder的作用和用法
tagline: 
image: 
date: 2016/11/4
categories: [web]
tags: [java,Spring]
---

# Spring中@initBinder的使用


这一周开发的时候，发现前台的date类型form数据不能传值到controller中的参数中，看到后台提示date类型转换失败。于是我想到以前是加了`@initBinder`就解决了。于是我就打开了`spring`的文档，查看了`@initBinder`。

`@initBinder`可以直接在你的`controller`中提供数据绑定。initbinder 方法不能有返回值，一般是返回`void`。下面的例子是给所有的`java.util.Date `类型的属性配置一个`CustomDateEditor`。


```
@Controller
public class MyFormController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...

}
```
当然，`@initBinder`还可以配置别的属性编辑器，例如`CustomNumberEditor、CustomBooleanEditor`等，实现数据绑定。
