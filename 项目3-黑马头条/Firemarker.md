FreeMarker 是一款 模板引擎： 即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。

# 依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
    </dependency>
</dependencies>
```



# application.yml

```yaml
spring:
  freemarker:
    cache: false  #关闭模板缓存，方便测试
    settings:
      template_update_delay: 0 #检查模板更新延迟时间，设置为0表示立即检查，如果时间大于0会有缓存不方便进行模板测试
    suffix: .ftl               #指定Freemarker模板文件的后缀名
    template-loader-path: classpath:/templates   #模板存放位置，默认是放在resources/templates目录下
```



# 快速入门

## 1. 创建模板

**在idea项目的resources目录下创建templates目录**，此目录为freemarker的默认模板存放目录。

在templates下创建模板文件 `01-basic.ftl` ，模板中的插值表达式最终会被freemarker替换成具体的数据。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello World!</title>
</head>
<body>
<b>普通文本 String 展示：</b><br><br>
Hello ${name} <br>
<hr>
<b>对象Student中的数据展示：</b><br/>
姓名：${stu.name}<br/>
年龄：${stu.age}
<hr>
</body>
</html>
```

## 2. Controller

创建Controller类，使用 Model对象来设置模型数据，最后返回模板文件的文件名。

```java
package com.xuecheng.test.freemarker.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.client.RestTemplate;

import java.util.Map;

@Controller
public class HelloController {

    @GetMapping("/basic")
    public String test(Model model) {
        //1.纯文本形式的参数
        model.addAttribute("name", "freemarker");
        //2.实体类相关的参数
        
        Student student = new Student();
        student.setName("小明");
        student.setAge(18);
        model.addAttribute("stu", student);
		//不需要加上模板文件名后缀
        return "01-basic";
    }
}
```



# 基础语法

##   **注释**

即`<#--  -->`

```velocity
<#--我是一个freemarker注释-->
```



## **插值（Interpolation）**

即 **`${..}`** 部分

```velocity
Hello ${name}
```

  

## FTL指令

和HTML标记类似，名字前加#予以区分

```velocity
<#FTL指令></#FTL指令> 
```

  

## 普通文本

仅文本信息，这些不是freemarker的注释、插值、FTL指令的内容会被freemarker忽略解析，直接输出内容。

```velocity
<#--freemarker中的普通文本-->
我是一个普通的文本
```



# 集合指令（List和Map）

## 数据模型

```java
@GetMapping("/list")
public String list(Model model){

    //------------------------------------
    Student stu1 = new Student();
    stu1.setName("小强");
    stu1.setAge(18);
    stu1.setMoney(1000.86f);
    stu1.setBirthday(new Date());

    //小红对象模型数据
    Student stu2 = new Student();
    stu2.setName("小红");
    stu2.setMoney(200.1f);
    stu2.setAge(19);

    //将两个对象模型数据存放到List集合中
    List<Student> stus = new ArrayList<>();
    stus.add(stu1);
    stus.add(stu2);

    //向model中存放List集合数据
    model.addAttribute("stus",stus);

    //------------------------------------

    //创建Map数据
    HashMap<String,Student> stuMap = new HashMap<>();
    stuMap.put("stu1",stu1);
    stuMap.put("stu2",stu2);
    // 3.1 向model中存放Map数据
    model.addAttribute("stuMap", stuMap);

    return "02-list";
}
```



## 模板文件

```html

    
<#-- list 数据的展示 -->

    <#list stus as stu>
        <tr>
            xxx_index表示索引，从0开始
            <td>${stu_index+1}</td>
            <td>${stu.name}</td>
            <td>${stu.age}</td>
            <td>${stu.money}</td>
     </tr>
</#list>

    
<#-- Map 数据的展示 -->

    
    
方式一：通过map['keyname'].property
${stuMap['stu1'].name}
${stuMap['stu1'].age}

方式二：通过map.keyname.property
${stuMap.stu2.name}
${stuMap.stu2.age}

遍历map中两个学生信息：
    <#list stuMap?keys as key >
        <tr>
            <td>${key_index}</td>
            <td>${stuMap[key].name}</td>
            <td>${stuMap[key].age}</td>
            <td>${stuMap[key].money}</td>
        </tr>
    </#list>
```



# if指令

指令格式

```html
<#if>
<#else>
</#if>
```

例子：

```html
<#list stus as stu >
    <#if stu.name='小红'>
        <tr style="color: red">
        <td>${stu_index}</td>
        <td>${stu.name}</td>
        <td>${stu.age}</td>
        <td>${stu.money}</td>
        </tr>
    <#else >
        <tr>
        <td>${stu_index}</td>
        <td>${stu.name}</td>
        <td>${stu.age}</td>
        <td>${stu.money}</td>
        </tr>
    </#if>
</#list>
```



# 运算符

**算数运算符**

- 加法： `+`

- 减法： `-`

- 乘法： `*`

- 除法： `/`

- 求模 (求余)： `%`

  

**比较运算符**

- **`=`**或者**`==`**:判断两个值是否相等. 
- **`!=`**:判断两个值是否不等. 
- **`>`**或者**`gt`**:判断左边值是否大于右边值 
- **`>=`**或者**`gte`**:判断左边值是否大于等于右边值 
- **`<`**或者**`lt`**:判断左边值是否小于右边值 
- **`<=`**或者**`lte`**:判断左边值是否小于等于右边值 

**比较运算符注意**

- **`=`**和**`!=`**可以用于字符串、数值和日期来比较是否相等
- **`=`**和**`!=`**两边必须是相同类型的值,否则会产生错误
- 字符串 **`"x"`** 、**`"x "`** 、**`"X"`**比较是不等的.因为FreeMarker是精确比较
- 其它的运行符可以作用于数字和日期,但不能作用于字符串
- 使用**`gt`**等字母运算符代替**`>`**会有更好的效果,因为 FreeMarker会把**`>`**解释成FTL标签的结束字符
- 可以使用括号来避免这种情况,如:**`<#if (x>y)>`**



**逻辑运算符**

- 逻辑与:&& 
- 逻辑或:|| 
- 逻辑非:! 



# 空值处理

**1、判断某变量是否存在使用 “??”**

用法为:`variable??`,如果该变量存在,返回true,否则返回false 

例：为防止stus为空报错可以加上判断如下：

```velocity
    <#if stus??>
    <#list stus as stu>
    	......
    </#list>
    </#if>
```



**2、缺失变量默认值使用 “!”**

- 使用 `!` 要以指定一个默认值，当变量为空时显示默认值

  例：  `${name!''}`表示如果name为空显示空字符串。

- 如果是嵌套对象则建议使用`()`括起来

  例： `${(stu.bestFriend.name)!''}`表示，如果stu或bestFriend或name为空默认显示空字符串。

  

# 内建函数

内建函数语法格式： **`变量+?+函数名称`**  



**1、某个集合的大小**

**`${集合名?size}`**



**2、日期格式化**

显示年月日: **`${today?date}`** 
显示时分秒：**`${today?time}`**   
显示日期+时间：**`${today?datetime}`**   
自定义格式化：  **`${today?string("yyyy年MM月")}`**



**3、内建函数`c`**

`model.addAttribute("point", 102920122);`

point是数字型，使用`${point}`会显示这个数字的值，每三位使用逗号分隔。

如果不想显示为每三位分隔的数字，可以使用c函数将数字型转成字符串输出

**`${point?c}`**



**4、将json字符串转成对象**

一个例子：

其中用到了 **#assign标签**，#assign的作用是定义一个变量。

```velocity
<#assign text="{'bank':'工商银行','account':'10101920201920212'}" />
<#assign data=text?eval />
开户行：${data.bank}  账号：${data.account}
```



# 生成静态html文件

之前的测试都是SpringMVC将Freemarker作为视图解析器（ViewReporter）来集成到项目中，工作中，有的时候需要使用Freemarker原生Api来生成静态内容，下面一起来学习下原生Api生成文本文件。

```java
import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateException;

@SpringBootTest(classes = FreemarkerDemoApplication.class)
@RunWith(SpringRunner.class)
public class FreemarkerTest {
	//1.注入Configuration对象
    @Autowired
    private Configuration configuration;

    @Test
    public void test() throws IOException, TemplateException {
        //2.freemarker的模板对象，获取Template对象
        Template template = configuration.getTemplate("02-list.ftl");
        //数据模型，键值对
        Map params = getData();
        //第一个参数 数据模型
        //第二个参数  输出流
        template.process(params, new FileWriter("d:/list.html"));
    }
}
```

