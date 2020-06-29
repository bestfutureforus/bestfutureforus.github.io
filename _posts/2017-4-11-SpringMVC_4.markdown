---
layout:     post
title:      "SpringMVC学习笔记(4)——商品修改功能开发"
subtitle:   ""
date:       2017-04-11 12:00:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 框架
---
在之前的文章中，已经针对商品订单管理案例，使用SSM框架完成了商品列表查询的功能，本文则介绍了另一个商品修改功能的开发，同时也是为后续文章所介绍的技术提供一个案例。

# 需求

操作流程：
1. 进入商品查询列表页面
2. 点击修改，进入商品修改页面，页面中显示了要修改的商品（从数据库查询）
	要修改的商品从数据库查询，根据商品id(主键)查询商品信息
3. 在商品修改页面，修改商品信息，修改后，点击提交

# 开发mapper

要求：
+ 根据id查询商品信息
+ 根据id更新Items表的数据

可直接使用逆向工程生成的代码：
```java
public interface ItemsMapper {
    int countByExample(ItemsExample example);

    int deleteByExample(ItemsExample example);

    int deleteByPrimaryKey(Integer id);

    int insert(Items record);

    int insertSelective(Items record);

    List<Items> selectByExampleWithBLOBs(ItemsExample example);

    List<Items> selectByExample(ItemsExample example);

    Items selectByPrimaryKey(Integer id);

    int updateByExampleSelective(@Param("record") Items record, @Param("example") ItemsExample example);

    int updateByExampleWithBLOBs(@Param("record") Items record, @Param("example") ItemsExample example);

    int updateByExample(@Param("record") Items record, @Param("example") ItemsExample example);

    int updateByPrimaryKeySelective(Items record);

    int updateByPrimaryKeyWithBLOBs(Items record);

    int updateByPrimaryKey(Items record);
}
```
其中我们用到的内容是：
```java
public interface ItemsMapper {
    Items selectByPrimaryKey(Integer id);
    
    int updateByPrimaryKeyWithBLOBs(Items record);
}
```

# 开发Service

接口功能：
+ 根据id查询商品信息
+ 修改商品信息

```java
//根据id查询商品信息
ItemsCustom findItemsById(Integer id) throws Exception;

//修改商品信息
void updateItems(Integer id, ItemsCustom itemsCustom) throws Exception;
```

# 开发Controller

方法：
+ 商品信息修改页面显示
+ 商品信息修改提交

```java
//使用@Controller来标识它是一个控制器
@Controller
//为了对url进行分类管理 ，可以在这里定义根路径，最终访问url是根路径+子路径
//比如：商品列表：/items/queryItems.action
//@RequestMapping("/items")
public class ItemsController {

    @Autowired
    private ItemsService itemsService;

    //商品查询列表
    @RequestMapping("/queryItems")
    //实现 对queryItems方法和url进行映射，一个方法对应一个url
    //一般建议将url和方法写成一样
    public ModelAndView queryItems() throws Exception{
        //调用service查找数据库，查询商品列表
        List<ItemsCustom> itemsList = itemsService.findItemsList(null);

        //返回ModelAndView
        ModelAndView modelAndView = new ModelAndView();
        //相当于request的setAttribute方法,在jsp页面中通过itemsList取数据
        modelAndView.addObject("itemsList",itemsList);

        //指定视图
        //下边的路径，如果在视图解析器中配置jsp的路径前缀和后缀，修改为items/itemsList
        //modelAndView.setViewName("/WEB-INF/jsp/items/itemsList.jsp");
        //下边的路径配置就可以不在程序中指定jsp路径的前缀和后缀
        modelAndView.setViewName("items/itemsList");

        return modelAndView;
    }


    //商品信息修改页面显示
    @RequestMapping("/editItems")
    //限制http请求方法，可以post和get
	//@RequestMapping(value="/editItems",method={RequestMethod.POST, RequestMethod.GET})
	public ModelAndView editItems()throws Exception {

		//调用service根据商品id查询商品信息
		ItemsCustom itemsCustom = itemsService.findItemsById(1);

		// 返回ModelAndView
		ModelAndView modelAndView = new ModelAndView();

		//将商品信息放到model
		modelAndView.addObject("itemsCustom", itemsCustom);

		//商品修改页面
		modelAndView.setViewName("items/editItems");

		return modelAndView;
	}

    //商品信息修改提交
    @RequestMapping("/editItemsSubmit")
    public ModelAndView editItemsSubmit(HttpServletRequest request, Integer id, ItemsCustom itemsCustom)throws Exception {

        //调用service更新商品信息，页面需要将商品信息传到此方法
        itemsService.updateItems(id, itemsCustom);

        //返回ModelAndView
        ModelAndView modelAndView = new ModelAndView();
        //返回一个成功页面
        modelAndView.setViewName("success");
        return modelAndView;
    }
}
```

# 注解@RequestMapping的使用

### 作用于Controller中的方法

`@RequestMapping`作用于方法时主要有两种功能：
+ **url映射**
定义controller方法对应的url，进行处理器映射使用。
+ **限制http请求方法**
出于安全性考虑，对http的链接进行方法限制。
如果限制请求为post方法，进行get请求将会报错。

具体用法如下：
```java
//商品信息修改页面显示
    //@RequestMapping("/editItems")
    //限制http请求方法，可以post和get
	@RequestMapping(value="/editItems",method={RequestMethod.POST, RequestMethod.GET})
	public ModelAndView editItems()throws Exception {
```

### 作用于Controller类

作用：
+ **窄化请求映射**
```java
//使用@Controller来标识它是一个控制器
@Controller
//为了对url进行分类管理 ，可以在这里定义根路径，最终访问url是根路径+子路径
//比如：商品列表：/items/queryItems.action
@RequestMapping("/items")
public class ItemsController {
```

# Controller方法的返回值

### 返回ModelAndView

在之前的例子中，使用的均是此种做法。需要方法结束时，定义ModelAndView，将model和view分别进行设置。

### 返回String

如果Controller方法返回String，可以有以下三种方法：

**1、表示返回逻辑视图名。**
真正视图(jsp路径)=前缀+逻辑视图名+后缀

```java
@RequestMapping(value="/editItems",method={RequestMethod.POST,RequestMethod.GET})
//@RequestParam里边指定request传入参数名称和形参进行绑定。
//通过required属性指定参数是否必须要传入
//通过defaultValue可以设置默认值，如果id参数没有传入，将默认值和形参绑定。
//public String editItems(Model model, @RequestParam(value="id",required=true) Integer items_id)throws Exception {
public String editItems(Model model)throws Exception {

    //调用service根据商品id查询商品信息
    ItemsCustom itemsCustom = itemsService.findItemsById(1);

    //通过形参中的model将model数据传到页面
    //相当于modelAndView.addObject方法
    model.addAttribute("itemsCustom", itemsCustom);

    return "items/editItems";
}
```
**2、 redirect重定向**
商品修改提交后，重定向到商品查询列表。
redirect重定向特点：浏览器地址栏中的url会变化。修改提交的request数据无法传到重定向的地址，因为重定向后重新进行request（request无法共享）。
```java
//重定向到商品查询列表
return "redirect:queryItems.action";
```
**3、forward页面转发**
通过forward进行页面转发，浏览器地址栏url不变，request可以共享。

```java
//页面转发
return "forward:queryItems.action";
```

### 返回void

在Controller方法形参上可以定义request和response，使用request或response指定响应结果：

**1、使用request转向页面**，如下：
```java
request.getRequestDispatcher("页面路径").forward(request, response);
```
**2、通过response页面重定向**：
```java
response.sendRedirect("url")
```

**3、通过response指定响应结果**，例如响应json数据如下：
```java
response.setCharacterEncoding("utf-8");
response.setContentType("application/json;charset=utf-8");
response.getWriter().write("json串");
```

