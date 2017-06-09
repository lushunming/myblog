---
layout: post
title: 使用spring的requestBody实现List绑定
description: 使用spring的requestBody实现List绑定
tagline: 
image: 
date: 2017/1/2
categories: [web]
tags: [java]
---
# 使用spring的requestBody实现List绑定


最近有很多一对多关系的表单需要保存，比如一个人有好几本书，他会在一个表单提交所有的数据，我的后台参数需要绑定一个`List`。
    下面是人和书的model：
    
```
    public class User {

	/** 自增型主键 */
	private Integer id;
	/** 姓名 */
	private String name;
	private String groupId;
	private List<Book> books;
}
```
    
```
    public class Book {

	/** 书籍 */
	private Integer id;
	/** 借书人（借书人的） */
	private Integer borrower;
	/** 预订人 */
	private Integer booker;
	/** 书名 */
	private String bookName;
	/** 书本页数 */
	private Integer pageCount;
	/** 价格 */
	private Double price;
	/** 作者 */
	private String author;
	/** 出版社 */
	private String press;
	/** 书籍类别（对应类别表的） */
	private Integer catgory;
    
```
我的后台参数就是`User user`，其实用form提交也是可以的，只要把book的每一行设为`books[i].bookName`这样也能提交，但是公司用的是`EasyUI`的 `dataGrid`，所以name就不能由我控制。于是我查了很多资料，有了下一种方法，使用 `ajax`和`@RequestBody`的方法提交。

在前台，我们用`ajax`

```
	$.ajax({
				url : '/ztree/save',
				type : "post",
				data : JSON.stringify({
					"name" : "a",
					"books" : [{"bookName":"a","price":12.3},{"bookName":"b"}]
				}),
				contentType : "application/json",
				beforeSend : function() {
					return $("#form").valid();
				}
			});
```
我们把`dataGrid`中的数据变为一个数组，然后将整个表单的数据变为一个json String，`contentType`设置为  ` "application/json"`,需要校验表单就在`ajax`的 `beforeSend`中调用。

后台我们使用`@RequestBody`来接受`json`类型的数据
```
@RequestMapping("save")
	@ResponseBody
	public String save(@RequestBody User user, HttpServletRequest request) {
		System.out.println(user.toString());
		return "ztree";
	}
```

这样就可以直接填充`user`参数。
     
    	
    
    
    
