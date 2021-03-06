---
layout:     post
title:      "Web开发中三层架构之 - junit单元测试"
subtitle:   " \"JUnit是一个Java语言的单元测试框架，快速学习Junit...\""
date:       2032-05-06 00:30:00
author:     "RyanYans"
header-img: "img/post-bg-junit.jpg"
catalog: true
tags:
    - junit Test
    - Java web
    - junit单元测试
---

# Web开发中三层架构之 - junit单元测试  

###### 在日常的开发当中，绝大多数都是协助式模块开发模式。假如一个项目非常的大，测试的东西非常的多，如果不用Junit的话，那么这个工作量是非常大的。因此，每完成一个功能模块就应需要单独测试一下。
  
## junit是什么？

JUnit是一个Java语言的单元测试框架。它由Kent Beck和Erich Gamma建立。多数Java的开发环境都已经集成了JUnit作为单元测试的工具。  来自JUnit的体验对测试驱动开发是很重要的，所以一些 JUnit知识经常 和测试驱动开发的讨论融合在一起。可以参考Kent Beck的 《Test-Driven Development: By Example》一书（有中文版和影印版）。

 
* junit(4.0)有什么特点？

 	
 	* 方法都是public的  
 	* 方法都没有返回值  
 	* 方法都没有参数  
 	* 方法都有@Test注解
	

Unit 4.0 的"Hello world" ：
		
	  public class HelloWorld
	  {
	    @Test 
		public void testMultiplication()
	    {
	      // Testing if 3*2=6:
	      assertEquals ("Multiplication", 6, 3*2);
	    }
	  }   
	//testMultiplication由 Test的标签所定义。
##1. junit入门

### 1、在MyEclipse中新建一个JUnit测试类  
![newJunit.PNG](https://ooo.0o0.ooo/2016/05/06/572cb0f2c5eb2.png)

![newJunit2.PNG](https://ooo.0o0.ooo/2016/05/06/572cb07776711.png)

### 2. 项目中的Junit

	private IUserService service = new UserServiceImpl();
	
		@Test
		public void testFindUser() {
			User user = service.findUser("test", "1234");
			System.out.println(user.toString());
			/*
			 * 断言方法：
			 * 	assertEquals方法有两个参数
			 * 		参数1：是预期的结果
			 * 		参数2：实际的结果
			 */
			assertEquals("1233@qq.com",user.getEmail());
		}
	
		@Test
		public void testFindUserByName() {
			User user = service.findUserByName("test");
			System.out.println(user.toString());
		} 
	
		@Test
		public void testSaveUser() throws Exception{
			SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
			Date date = format.parse("2015-06-15");
			User user = new User("test","1234",date,"1233@qq.com");
			int res = service.saveUser(user);
			//使用断言方法，看看保存是否成功
			assertEquals(1,res);
		}
	} 

### 3.Run Junit Test  

![runTest.PNG](https://ooo.0o0.ooo/2016/05/06/572cb0781dfce.png)

* 正常运行  

![True.PNG](https://ooo.0o0.ooo/2016/05/06/572cb078238c7.png)  

* 运行出错  

![错误.PNG](https://ooo.0o0.ooo/2016/05/06/572cb07871718.png)


###### 使用junit测试比使用main方法测试有很大的不同的，每个标识为@Test的方法都是一个可运行的方法，并且他们之间互不影响，例如testAddd方法出现问题了，并不影响testMinus方法的运行。这就是单元测试的好处。  

--------------------------

###### ·· 生命在于折腾 ··

--------------------------
