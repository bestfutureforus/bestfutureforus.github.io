---
layout:     post
title:      "SpringMVC学习笔记(5)——参数绑定"
subtitle:   ""
date:       2017-04-11 22:00:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 框架
---
参数绑定是指Web API将**HTTP请求数据**绑定到一个动作方法的参数中。
在SpringMVC中，从客户端请求key/value数据，经过参数绑定，将key/value数据绑定到**Handler(Controller)方法的形参**上，接收页面提交的数据通过方法的形参来接收。处理器适配器调用SpringMVC提供的**参数绑定组件（converter）**将请求的key/value数据转成Handler(Controller)方法的形参。
下面将根据参数的不同类型，来介绍参数绑定的不同方法。

# 默认支持的类型

在Handler(Controller)方法的形参中添加如下类型的参数处理适配器会默认识别并进行赋值：
+ **HttpServletRequest：**通过request对象获取请求信息
+ **HttpServletResponse：**通过response处理响应信息
+ **HttpSession：**通过session对象得到session中存放的对象
+ **Model/ModelMap：**model是一个接口，modelMap是一个接口实现 ，作用是将model数据填充到request域。

# 简单类型

通过`@RequestParam`对简单类型的参数进行绑定。
如果不使用`@RequestParam`，要求request传入参数名称和Controller方法的形参名称一致，方可绑定成功。
如果使用`@RequestParam`，不用限制request传入参数名称和controller方法的形参名称一致。
通过`required`属性指定参数是否必须要传入，如果设置为`true`，则必须传入参数。

应用实例：
```java
@RequestMapping(value = "/editItems", method = {RequestMethod.POST, RequestMethod.GET})
    //@RequestParam里边指定request传入参数名称和形参进行绑定。
    //通过required属性指定参数是否必须要传入
    //通过defaultValue可以设置默认值，如果id参数没有传入，将默认值和形参绑定。
    public String editItems(Model model, @RequestParam(value = "id", required = true) Integer items_id) throws Exception {

        //调用service根据商品id查询商品信息
        ItemsCustom itemsCustom = itemsService.findItemsById(items_id);

        //判断商品是否为空，根据id没有查询到商品，抛出异常，提示用户商品信息不存在
        //if(itemsCustom == null){
        //throw new CustomException("修改的商品信息不存在!");
        //}

        //通过形参中的model将model数据传到页面
        //相当于modelAndView.addObject方法
        model.addAttribute("items", itemsCustom);

        return "items/editItems";
    }
```

# pojo绑定

页面中input的name和Controller的pojo形参中的属性名称一致，将页面中数据绑定到pojo。

**页面定义：**
```html
<table width="100%" border=1>
<tr>
	<td>商品名称</td>
	<td><input type="text" name="name" value="${itemsCustom.name }"/></td>
</tr>
<tr>
	<td>商品价格</td>
	<td><input type="text" name="price" value="${itemsCustom.price }"/></td>
</tr>
```
**pojo定义如下：**
```
public class Items {
    private String name;

    private Float price;
    
    ...
```
**Contrller方法定义如下：**
```java
@RequestMapping("/editItemSubmit")
public String editItemSubmit(Items items)throws Exception{
    ...
```
请求的参数名称和pojo的属性名称一致，会自动将请求参数赋值给pojo的属性。


# 包装类型pojo参数绑定

如果想在商品查询Controller方法中实现商品查询条件传入，主要有两种方法：
1. 在形参中 添加**HttpServletRequest request**参数，通过request接收查询条件参数。
2. 在形参中让**包装类型的pojo**接收查询条件参数。

页面传参数具有复杂性和多样性的特点，条件包括 ：用户账号、商品编号、订单信息等等，
如果将用户账号、商品编号、订单信息等放在简单pojo（属性是简单类型）中，pojo类属性比较多，比较乱，因此我们使用包装类型的pojo，pojo中的属性也是pojo。

### 页面参数和Controller方法形参定义

**页面参数：**
```xml
商品名称：<input name="itemsCustom.name" />
```
**注意：**itemsCustom和包装pojo中的属性一致即可。

**Controller方法形参：**
```java
public ModelAndView queryItems(HttpServletRequest request,ItemsQueryVo itemsQueryVo) throws Exception
```
**包装类ItemsQueryVo中部分属性**
```java
public class ItemsQueryVo {

    //商品信息
    private Items items;

    //为了系统 可扩展性，对原始生成的po进行扩展
    private ItemsCustom itemsCustom;
```
其中，ItemsQueryVo的属性itemsCustom和页面参数中的一致

# 集合类型绑定

## 数组绑定

**需求：**商品批量删除，用户在页面选择多个商品，批量删除。

#### 表现层实现

将页面选择(多选)的商品id，传到Controller方法的形参，方法形参使用数组接收页面请求的多个商品id。

**Controller方法定义：**
```java
 // 批量删除 商品信息
    @RequestMapping("/deleteItems")
    public String deleteItems(Integer[] items_id) throws Exception {

        // 调用service批量删除商品
        // ...
        System.out.println(items_id);

        return "success";

    }
```
**页面定义：**
```vbscript-html
<c:forEach items="${itemsList }" var="item">
            <tr>
                <td><input type="checkbox" name="items_id" value="${item.id}"/></td>
                <td>${item.name }</td>
                <td>${item.price }</td>
                <td><fmt:formatDate value="${item.createtime}" pattern="yyyy-MM-dd HH:mm:ss"/></td>
                <td>${item.detail }</td>
                <td><a href="${pageContext.request.contextPath }/items/editItems.action?id=${item.id}">修改</a></td>
            </tr>
</c:forEach>
```

## list绑定

**需求：**
通常在需要批量提交数据时，将提交的数据绑定到list<pojo>中，比如：成绩录入（录入多门课成绩，批量提交）。
本例子需求：批量商品修改，在页面输入多个商品信息，将多个商品信息提交到Controller方法中。

#### 表现层实现

**Controller方法定义：**
1. 进入批量商品修改页面(页面样式参考商品列表实现)
2. 批量修改商品提交

使用list接收页面提交的批量数据，通过包装pojo接收，在包装pojo中定义`list<pojo>`属性
```java
public class ItemsQueryVo {

    //商品信息
    private Items items;

    //为了系统 可扩展性，对原始生成的po进行扩展
    private ItemsCustom itemsCustom;

    //批量商品信息
    private List<ItemsCustom> itemsList;
```
```java
// 批量修改商品提交
// 通过ItemsQueryVo接收批量提交的商品信息，将商品信息存储到itemsQueryVo中itemsList属性中。
@RequestMapping("/editItemsAllSubmit")
public String editItemsAllSubmit(ItemsQueryVo itemsQueryVo) throws Exception {

    return "success";
}
```
**页面定义**
```html
<c:forEach items="${itemsList }" var="item" varStatus="status">
    <tr>

        <td><input name="itemsList[${status.index }].name" value="${item.name }"/></td>
        <td><input name="itemsList[${status.index }].price" value="${item.price }"/></td>
        <td><input name="itemsList[${status.index }].createtime" value="<fmt:formatDate value="${item.createtime}" pattern="yyyy-MM-dd HH:mm:ss"/>"/></td>
        <td><input name="itemsList[${status.index }].detail" value="${item.detail }"/></td>

    </tr>
</c:forEach>
```

## map绑定

通过在包装pojo中定义map类型属性。
在包装类中定义map对象，并添加get/set方法，action使用包装对象接收。

**包装类中定义Map对象如下：**
```java
public class QueryVo {
	private Map<String, Object> itemInfo = new HashMap<String, Object>();
	//get/set方法..
}
```
**页面定义如下：**
```html
<tr>
<td>学生信息：</td>
<td>
	姓名：<inputtype="text"name="itemInfo['name']"/>
	年龄：<inputtype="text"name="itemInfo['price']"/>
	.. .. ..
</td>
</tr>
```
**Contrller方法定义如下：**
```java
public String useraddsubmit(Model model,QueryVo queryVo)throws Exception{
	System.out.println(queryVo.getStudentinfo());
}
```
