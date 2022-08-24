# JQuery

jQuery 就是方便程序员使用JS开发的一个库

要使用jQuery库，就需要在html的script 标签中把它包含进来：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>

<script type="text/javascript" src="../script/jquery-1.7.2.js"></script>
<script type="text/javascript">

	//使用$()代替window.onload
	$(
        function(){
			//使用选择器获取按钮对象，随后绑定单击响应函数
			$("#btnId").click(
                function(){
					//弹出Hello
					alert('Hello');
				}
            );
        }
    );

</script>


</head>
<body>

	<button id="btnId">SayHello</button>

</body>
</html>
```

## 1.jQuery的核心函数

**$ ** 是 jQuery 的核心函数，它有很多功能：

1、 传入参数为 [ 函数 ] 时：

​	表示页面加载完成之后。 相当于 `window.onload = function(){}`

2、 传入参数为 [ HTML 字符串 ] 时：

​	会创建这个 html 标签对象

3、 传入参数为 [ 选择器字符串 ] 时：

`$("#id属性值")`		id 选择器， 根据 id 查询标签对象

`$("标签名")` 			标签名选择器， 根据指定的标签名查询标签对象

`$(".class 属性值")` 	类型选择器， 可以根据 class 属性查询标签对象

4、 传入参数为 [ DOM 对象 ] 时：

会把这个 dom 对象转换为 jQuery 对象



## 2.jQuery对象和DOM对象

**Dom 对象**
通过 getElementById()查询出来的标签对象是 Dom 对象
通过 getElementsByName()查询出来的标签对象是 Dom 对象
通过 getElementsByTagName()查询出来的标签对象是 Dom 对象
通过 createElement() 方法创建的对象， 是 Dom 对象
	

**jQuery 对象**
通过 jQuery 提供的 API 创建的对象， 是 jQuery 对象
通过 jQuery 包装的 Dom 对象， 也是 jQuery 对象
通过 jQuery 提供的 API 查询到的对象， 是 jQuery 对象

**jQuery对象的实质：jQuery 对象是 dom 对象的数组 + jQuery 提供的一系列功能函数**  

jQuery 对象不能使用 DOM 对象的属性和方法

DOM 对象也不能使用 jQuery 对象的属性和方法  



## 3.jQuery对象和DOM对象互转

1）DOM ----> jQuery

​	`$(DOM对象)`  就可以转换成为 jQuery 对象  

2）jQuery ---->DOM

​	`jQuery对象[下标]` 取出相应的 DOM 对象  



## 4.jQuery选择器

### 1）基本选择器

**#ID 选择器**： 根据 id 查找标签对象

**.class 选择器**： 根据 class 查找标签对象

**element 选择器**： 根据标签名查找标签对象

***选择器**： 表示任意的， 所有的元素

**selector1, selector2** 组合选择器： 选择器 1， 选择器 2 的并集



`p.myClass  `表示标签名必须是 p 标签， 而且 class 类型还要是 myClass  



### 2）层级选择器

**ancestor descendant** 后代选择器 ： 在给定的祖先元素下匹配所有的后代元素

**parent > child** 子元素选择器： 在给定的父元素下匹配所有的子元素

**prev + next** 相邻元素选择器： 匹配所有紧接在 prev 元素后的 next 元素

**prev ~ sibings** 之后的兄弟元素选择器： 匹配 prev 元素之后的所有 siblings 元素



### 3）过滤选择器

#### 基本过滤器：

**:first**                                    获取第一个元素
**:last**                                    获取最后个元素
**:not(selector)**                    去除所有与给定选择器匹配的元素
**:even**                                  匹配所有索引值为偶数的元素，从 0  开始计数
**:odd**                                    匹配所有索引值为奇数的元素，从 0  开始计数
**:eq(index)**                          匹配一个给定索引值的元素
**:gt(index)**                           匹配所有大于给定索引值的元素
**:lt(index)**                            匹配所有小于给定索引值的元素
**:header**                              匹配如 h1, h2, h3 之类的标题元素
**:animated**                          匹配所有正在执行动画效果的元素



#### 内容过滤器：

**:contains(text)**         匹配包含给定文本的元素
**:empty**                       匹配所有不包含子元素或者文本的空元素
**:parent**                  匹配含有子元素或者文本的元素
**:has(selector)**           匹配含有选择器所匹配的元素的元素



#### 属性过滤器：

**[attribute]**                              匹配包含给定属性的元素。
**[attribute=value]**                   匹配给定的属性是某个特定值的元素
**[attribute!=value]**                    匹配所有不含有指定的属性，或者属性不等于特定值的元素。
**[attribute^=value]**                   匹配给定的属性是以某些值开始的元素
**[attribute$=value]**                    匹配给定的属性是以某些值结尾的元素
**[attribute*=value]**                    匹配给定的属性是包含某些值的元素
**[attrSel1] [attrSel2] [attrSelN]**    复合属性选择器，需要同时满足多个条件时使用。



#### 表单过滤器：

**:input**                         匹配所有 input, textarea, select  和 button  元素
**:text**                           匹配所有 文本输入框
**:password**                 匹配所有的密码输入框
**:radio**                         匹配所有的单选框
**:checkbox**                 匹配所有的复选框
**:submit**                      匹配所有提交按钮
**:image**                       匹配所有 img 标签
**:reset**                         匹配所有重置按钮
**:button**                      匹配所有 input type=button<button>按钮
**:file**                             匹配所有 input type=file 文件上传
**:hidden**                      匹配所有不可见元素 display:none  或 input type=hidden



#### 表单对象属性过滤器：

**:enabled**                    匹配所有可用元素
**:disabled**                   匹配所有不可用元素
**:checked**                   匹配所有选中的单选，复选，和下拉列表中选中的 option 标签对象
**:selected**                   匹配所有选中的 option



### 4）jQuery元素筛选函数

eq()                   获取给定索引的元素

first()                 获取第一个元素

last()                  获取最后一个元素

filter(exp)         留下匹配的元素

is(exp)               判断是否匹配给定的选择器，只要有一个匹配就返回true 

has(exp)           返回包含有匹配选择器的元素的元素

not(exp)           删除匹配选择器的元素 

children(exp)   返回匹配给定选择器的子元素 

find(exp)          返回匹配给定选择器的后代元素

next()                返回当前元素的下一个兄弟元素

nextAll()           返回当前元素后面所有的兄弟元素

nextUntil()       返回当前元素到指定匹配的元素为止的后面元素

parent()            返回父元素

prev(exp)         返回当前元素的上一个兄弟元素

prevAll()           返回当前元素前面所有的兄弟元素 

prevUnit(exp)  返回当前元素到指定匹配的元素为止的前面元素

siblings(exp)    返回所有兄弟元素

add()                 把 add 匹配的选择器的元素添加到当前 jquery 对象中



## 5.jQuery属性操作

### html() 

它可以设置和获取起始标签和结束标签中的内容。 跟 dom 属性 innerHTML 一样。

### text() 

它可以设置和获取起始标签和结束标签中的文本。 跟 dom 属性 innerText 一样。

### val()

它可以设置和获取表单项的 value 属性值。 跟 dom 属性 value 一样 。

如果val函数有参数，表示设置值，如果无参数，则表示获取值。

val 方法同时设置多个表单项的选中状态：  

```html
 	<script type="text/javascript" src="script/jquery-1.7.2.js"></script>
    <script type="text/javascript">
        $(function () {
            /*
            // 批量操作单选
            $(":radio").val(["radio2"]);
            // 批量操作筛选框的选中状态
            $(":checkbox").val(["checkbox3","checkbox2"]);
            // 批量操作多选的下拉框选中状态
            $("#multiple").val(["mul2","mul3","mul4"]);
            // 操作单选的下拉框选中状态
            $("#single").val(["sin2"]);
            */
            $("#multiple,#single,:radio,:checkbox").val(["radio2","checkbox1","checkbox3","mul1","mul4","sin3"]
            );
        });
    </script>


<body>
单选：
<input name="radio" type="radio" value="radio1" />radio1
<input name="radio" type="radio" value="radio2" />radio2
<br/>
多选：
<input name="checkbox" type="checkbox" value="checkbox1" />checkbox1
<input name="checkbox" type="checkbox" value="checkbox2" />checkbox2
<input name="checkbox" type="checkbox" value="checkbox3" />checkbox3
<br/>
下拉多选 ：
<select id="multiple" multiple="multiple" size="4">
    <option value="mul1">mul1</option>
    <option value="mul2">mul2</option>
    <option value="mul3">mul3</option>
    <option value="mul4">mul4</option>
</select>
<br/>
下拉单选 ：
<select id="single">
    <option value="sin1">sin1</option>
    <option value="sin2">sin2</option>
    <option value="sin3">sin3</option>
</select>
</body>

```



### attr()

单参数为读取，双参数为设置

可以设置和获取属性的值， **不推荐操作 checked、 readOnly、 selected、 disabled 等属性**

attr 方法还可以操作非标准的属性。 比如自定义属性： abc,bbj  



### prop()

也是用来设置和获取属性的值，用于与attr() 互补，**适合操作 checked、 readOnly、 selected、 disabled 等属性**。



**例如：**

prop 和 attr 在操作单选框的checked属性的时候，如果没有选中，attr会返回**undefined**，而prop 会返回**false**



```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    <script type="text/javascript" src="../../script/jquery-1.7.2.js"></script>
    <script type="text/javascript">
        $(function () {
            // 给全选绑定单击事件
            $("#checkedAllBtn").click(function () {
                $(":checkbox").prop("checked", true);
            });
            // 给全不选绑定单击事件
            $("#checkedNoBtn").click(function () {
                $(":checkbox").prop("checked", false);
            });
            // 反选单击事件
            $("#checkedRevBtn").click(function () {// 查询全部的球类的复选框
                $(":checkbox[name='items']").each(function () {
                    // 在 each 遍历的 function 函数中， 有一个 this 对象。 这个 this 对象是当前正在遍历到的 dom 对象
                    this.checked = !this.checked;
                });
                // 要检查 是否满选
                // 获取全部的球类个数
                var allCount = $(":checkbox[name='items']").length;
                // 再获取选中的球类个数
                var checkedCount = $(":checkbox[name='items']:checked").length;
                // if (allCount == checkedCount) {
                // $("#checkedAllBox").prop("checked",true);
                // } else {
                // $("#checkedAllBox").prop("checked",false);
                // }
                $("#checkedAllBox").prop("checked", allCount == checkedCount);
            });
            // 【提交】 按钮单击事件
            $("#sendBtn").click(function () {
                // 获取选中的球类的复选框
                $(":checkbox[name='items']:checked").each(function () {
                    alert(this.value);
                });
            });
            // 给【全选/全不选】 绑定单击事件
            $("#checkedAllBox").click(function () {
                // 在事件的 function 函数中， 有一个 this 对象， 这个 this 对象是当前正在响应事件的 dom 对象
                // alert(this.checked);
                $(":checkbox[name='items']").prop("checked", this.checked);
            });
            // 给全部球类绑定单击事件
            $(":checkbox[name='items']").click(function () {
                // 要检查 是否满选
                // 获取全部的球类个数
                var allCount = $(":checkbox[name='items']").length;
                // 再获取选中的球类个数
                var checkedCount = $(":checkbox[name='items']:checked").length;
                $("#checkedAllBox").prop("checked", allCount == checkedCount);
            });
        });
    </script>
</head>
<body>
<form method="post" action="">
    你爱好的运动是？ <input type="checkbox" id="checkedAllBox"/>全选/全不选
    <br/>
    <input type="checkbox" name="items" value="足球"/>足球
    <input type="checkbox" name="items" value="篮球"/>篮球
    <input type="checkbox" name="items" value="羽毛球"/>羽毛球
    <input type="checkbox" name="items" value="乒乓球"/>乒乓球
    <br/>
    <input type="button" id="checkedAllBtn" value="全 选"/>
    <input type="button" id="checkedNoBtn" value="全不选"/>
    <input type="button" id="checkedRevBtn" value="反 选"/>
    <input type="button" id="sendBtn" value="提 交"/>
</form>
</body>
</html>
```



## 6.DOM的增删改

### appendTo()                               

a.appendTo(b)   	把a添加为b的最后一个子元素。

### prependTo() 

a.prependTo(b) 	把a添加为b的第一个子元素

### insertAfter() 

a.insertAfter(b)  	把a插入到b的后面，即ba

### insertBefore() 

a.insertBefore(b) 	把a插入到b的前面，即ab

### replaceWith() 

a.replaceWith(b) 	用 b 替换掉 a

### replaceAll() 

a.replaceAll(b) 		用 a 替换掉所有 b

### remove() 

a.remove()				删除 a 标签

### empty() 

a.empty()					清空 a 标签里的内容



示例代码：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
	<style type="text/css">
		select {
			width: 100px;
			height: 140px;
		}
		
		div {
			width: 130px;
			float: left;
			text-align: center;
		}
	</style>
	<script type="text/javascript" src="script/jquery-1.7.2.js"></script>
	<script type="text/javascript">
		// 页面加载完成
		$(function () {
			// 第一个按钮 【选中添加到右边】
			$("button:eq(0)").click(function () {
				$("select:eq(0) option:selected").appendTo($("select:eq(1)"));
			});
			// 第二个按钮 【全部添加到右边】
			$("button:eq(1)").click(function () {
				$("select:eq(0) option").appendTo($("select:eq(1)"));
			});

			// 第三个按钮 【选中删除到左边】
			$("button:eq(2)").click(function () {
				$("select:eq(1) option:selected").appendTo($("select:eq(0)"));
			});

			// 第四个按钮 【全部删除到左边】
			$("button:eq(3)").click(function () {
				$("select:eq(1) option").appendTo($("select:eq(0)"));
			});
		});
	</script>
</head>
<body>

	<div id="left">
		<select multiple="multiple" name="sel01">
			<option value="opt01">选项1</option>
			<option value="opt02">选项2</option>
			<option value="opt03">选项3</option>
			<option value="opt04">选项4</option>
			<option value="opt05">选项5</option>
			<option value="opt06">选项6</option>
			<option value="opt07">选项7</option>
			<option value="opt08">选项8</option>
		</select>
		
		<button>选中添加到右边</button>
		<button>全部添加到右边</button>
	</div>
	<div id="rigth">
		<select multiple="multiple" name="sel02">
		</select>
		<button>选中删除到左边</button>
		<button>全部删除到左边</button>
	</div>

</body>
</html>
```



动态添加，删除表结构

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Untitled Document</title>
<link rel="stylesheet" type="text/css" href="styleB/css.css" />
<script type="text/javascript" src="../../script/jquery-1.7.2.js"></script>
<script type="text/javascript">

	$(function () {

		// 创建一个用于复用的删除的function函数
		var deleteFun = function(){

			// alert("删除 操作 的function : " + this);

			// 在事件响应的function函数中，有一个this对象。这个this对象是当前正在响应事件的dom对象。
			var $trObj = $(this).parent().parent();

			var name = $trObj.find("td:first").text();

			/**
			 * confirm 是JavaScript语言提供的一个确认提示框函数。你给它传什么，它就提示什么<br/>
			 * 当用户点击了确定，就返回true。当用户点击了取消，就返回false
			 */
			if( confirm("你确定要删除[" + name +  "]吗？") ){
				$trObj.remove();
			}

			// return false; 可以阻止 元素的默认行为。
			return false;
		};




		// 给【Submit】按钮绑定单击事件
		$("#addEmpButton").click(function () {
			// 获取输入框，姓名，邮箱，工资的内容
			var name = $("#empName").val();
			var email = $("#email").val();
			var salary = $("#salary").val();


			// 创建一个行标签对象，添加到显示数据的表格中
			var $trObj = $("<tr>" +
					"<td>" + name +  "</td>" +
					"<td>" + email + "</td>" +
					"<td>" + salary + "</td>" +
					"<td><a href=\"deleteEmp?id=002\">Delete</a></td>" +
					"</tr>");


			// 添加到显示数据的表格中
			$trObj.appendTo( $("#employeeTable") );

			// 给添加的行的a标签绑上事件
			$trObj.find("a").click( deleteFun );


		});


		// 给删除的a标签绑定单击事件
		$("a").click( deleteFun );


	});

	
</script>
</head>
<body>

	<table id="employeeTable">
		<tr>
			<th>Name</th>
			<th>Email</th>
			<th>Salary</th>
			<th>&nbsp;</th>
		</tr>
		<tr>
			<td>Tom</td>
			<td>tom@tom.com</td>
			<td>5000</td>
			<td><a href="deleteEmp?id=001">Delete</a></td>
		</tr>
		<tr>
			<td>Jerry</td>
			<td>jerry@sohu.com</td>
			<td>8000</td>
			<td><a href="deleteEmp?id=002">Delete</a></td>
		</tr>
		<tr>
			<td>Bob</td>
			<td>bob@tom.com</td>
			<td>10000</td>
			<td><a href="deleteEmp?id=003">Delete</a></td>
		</tr>
	</table>

	<div id="formDiv">
	
		<h4>添加新员工</h4>

		<table>
			<tr>
				<td class="word">name: </td>
				<td class="inp">
					<input type="text" name="empName" id="empName" />
				</td>
			</tr>
			<tr>
				<td class="word">email: </td>
				<td class="inp">
					<input type="text" name="email" id="email" />
				</td>
			</tr>
			<tr>
				<td class="word">salary: </td>
				<td class="inp">
					<input type="text" name="salary" id="salary" />
				</td>
			</tr>
			<tr>
				<td colspan="2" align="center">
					<button id="addEmpButton" value="abc">
						Submit
					</button>
				</td>
			</tr>
		</table>

	</div>

</body>
</html>

```



## 7.用于调整CSS样式的一些函数

### addClass()

这个是为指定元素添加 **class属性**，但是一般用于为指定元素添加CSS样式。



### removeClass()

移除元素的 **class属性**



### toggleClass()

如果元素有参数代表的 **class属性**，则删除该class属性，如果元素没有该属性，则添加该class属性：

`$divEle.toggleClass('redDiv')` 假如div元素的class属性有redDiv这个值，则删除，如果没有，则添加redDiv值。



### offset()

 获取和设置元素的坐标。



示例代码：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    <style type="text/css">

        div {
            width: 100px;
            height: 260px;
        }

        div.whiteborder {
            border: 2px white solid;
        }

        div.redDiv {
            background-color: red;
        }

        div.blueBorder {
            border: 5px blue solid;
        }

    </style>

    <script type="text/javascript" src="script/jquery-1.7.2.js"></script>
    <script type="text/javascript">

        $(function () {

            var $divEle = $('div:first');

            $('#btn01').click(function () {
                //addClass() - 向被选元素添加一个或多个类
                $divEle.addClass('redDiv blueBorder');
            });

            $('#btn02').click(function () {
                //removeClass() - 从被选元素删除一个或多个类
                $divEle.removeClass();
            });


            $('#btn03').click(function () {
                //toggleClass() - 对被选元素进行添加/删除类的切换操作
                $divEle.toggleClass('redDiv')
            });


            $('#btn04').click(function () {
                //offset() - 返回第一个匹配元素相对于文档的位置。
                var pos = $divEle.offset();
                console.log(pos);

                $divEle.offset({
                    top: 100,
                    left: 50
                });

            });

        })
    </script>
</head>
<body>
<table align="center">
    <tr>
        <td>
            <div class="border">
            </div>
        </td>

        <td>
            <div class="btn">
                <input type="button" value="addClass()" id="btn01"/>
                <input type="button" value="removeClass()" id="btn02"/>
                <input type="button" value="toggleClass()" id="btn03"/>
                <input type="button" value="offset()" id="btn04"/>
            </div>
        </td>
    </tr>
</table>
<br/> <br/>
<br/> <br/>
</body>
</html>
```



# 8.jQuery 动画

### show() 

​	将隐藏的元素显示

### hide() 

​	将可见的元素隐藏。

### toggle() 

​	切换，可见就隐藏， 不可见就显示。

### fadeIn() 

​	淡入（慢慢可见）

### fadeOut() 

​	淡出（慢慢消失）

### fadeTo() 

​	在指定时长内慢慢的将透明度修改到指定的值。 0 透明， 1 完成可见， 0.5 半透明

### fadeToggle()

 	淡入/淡出 切换



以上动画方法都可以添加参数。
1、 动画执行的时长， 以毫秒为单位
2、 动画的回调函数 (动画完成后自动调用的函数)



示例：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>品牌展示练习</title>
<style type="text/css">
* {
	margin: 0;
	padding: 0;
}

body {
	font-size: 12px;
	text-align: center;
}

a {
	color: #04D;
	text-decoration: none;
}

a:hover {
	color: #F50;
	text-decoration: underline;
}

.SubCategoryBox {
	width: 600px;
	margin: 0 auto;
	text-align: center;
	margin-top: 40px;
}

.SubCategoryBox ul {
	list-style: none;
}

.SubCategoryBox ul li {
	display: block;
	float: left;
	width: 200px;
	line-height: 20px;
}

.showmore , .showless{
	clear: both;
	text-align: center;
	padding-top: 10px;
}

.showmore a , .showless a{
	display: block;
	width: 120px;
	margin: 0 auto;
	line-height: 24px;
	border: 1px solid #AAA;
}

.showmore a span {
	padding-left: 15px;
	background: url(img/down.gif) no-repeat 0 0;
}

.showless a span {
	padding-left: 15px;
	background: url(img/up.gif) no-repeat 0 0;
}

.promoted a {
	color: #F50;
}
</style>
<script type="text/javascript" src="script/jquery-1.7.2.js"></script>
<script type="text/javascript">
	$(function() {
		// 基本初始状态
		$("li:gt(5):not(:last)").hide();

		// 给功能的按钮绑定单击事件
		$("div div a").click(function () {
			// 让某些品牌，显示，或隐藏
			$("li:gt(5):not(:last)").toggle();
			// 判断 品牌，当前是否可见
			if( $("li:gt(5):not(:last)").is(":hidden") ){
				// 品牌隐藏的状态 ：1 显示全部品牌    == 角标向下 showmore
				$("div div a span").text("显示全部品牌");

				$("div div").removeClass();
				$("div div").addClass("showmore");

				// 去掉高亮
				$("li:contains('索尼')").removeClass("promoted");

			} else {
				// 品牌可见的状态：2 显示精简品牌	 == 角标向上 showless
				$("div div a span").text("显示精简品牌");

				$("div div").removeClass();
				$("div div").addClass("showless");

				// 加高亮
				$("li:contains('索尼')").addClass("promoted");
			}

			return false;
		});
	});
</script>
</head>
<body>
	<div class="SubCategoryBox">
		<ul>
			<li><a href="#">佳能</a><i>(30440) </i></li>
			<li><a href="#">索尼</a><i>(27220) </i></li>
			<li><a href="#">三星</a><i>(20808) </i></li>
			<li><a href="#">尼康</a><i>(17821) </i></li>
			<li><a href="#">松下</a><i>(12289) </i></li>
			<li><a href="#">卡西欧</a><i>(8242) </i></li>
			<li><a href="#">富士</a><i>(14894) </i></li>
			<li><a href="#">柯达</a><i>(9520) </i></li>
			<li><a href="#">宾得</a><i>(2195) </i></li>
			<li><a href="#">理光</a><i>(4114) </i></li>
			<li><a href="#">奥林巴斯</a><i>(12205) </i></li>
			<li><a href="#">明基</a><i>(1466) </i></li>
			<li><a href="#">爱国者</a><i>(3091) </i></li>
			<li><a href="#">其它品牌相机</a><i>(7275) </i></li>
		</ul>
		<div class="showmore">
			<a href="more.html"><span>显示全部品牌</span></a>
		</div>
	</div>
</body>
</html>


```



## 9.jQuery事件操作

`$( function(){} );` 和 `window.onload = function(){}`的区别：、

**他们分别是在什么时候触发？**

​	1、 jQuery 的页面加载完成之后是浏览器的内核解析完页面的标签创建好 DOM 对象之后就会马上执行。

​	2、 原生 js 的页面加载完成之后， 除了要等浏览器内核解析完标签创建好 DOM 对象， 还要等标签显示时需要的内容加载完成。

**如果同时存在，他们触发的先后顺序？**

​	1、 jQuery 先

​	2、 原生 js 后

**他们执行的次数？**

​	1、 原生 js 的页面加载完成之后， 只会执行最后一次的赋值函数。

​	2、 jQuery 的页面加载完成之后是全部把注册的 function 函数， 依次顺序全部执行。



### jQuery中的事件处理函数

#### click() 

单击事件

#### mouseover() 

鼠标移入事件

#### mouseout() 

鼠标移出事件



如果给以上函数传一个function，那么表示**绑定事件**

如果直接调用以上函数，不带参数，则表示**触发事件**





#### bind() 

可以给元素一次性绑定一个或多个事件。

#### one() 

使用上跟 bind 一样。 但是 one 方法绑定的事件只会触发一次。

#### unbind()

跟 bind 方法相反的操作， 解除事件的绑定

#### live()

它可以用来绑定选择器匹配的所有元素的事件。 哪怕这个元素是后面动态创建出来的也有效，也就是实时生效。



## 10.事件的冒泡

事件的冒泡是指，父子元素同时监听同一个事件。 当触发子元素的事件的时候， 同一个事件也被传递到了父元素的事件里去响应。  此时父子元素绑定的事件都会触发。

一般都要求只触发子元素的事件，也就是说让其不进行传递。

**在子元素绑定的事件的函数体内， return false 就可以阻止事件的冒泡传递。**  



## 11.JS事件对象

事件对象， 是封装有触发的事件信息的一个 javascript 对象。
我们重点关心的是怎么拿到这个 javascript 的事件对象，以及使用。  

如何获取呢 javascript 事件对象呢？
	在给元素绑定事件的时候， 在事件的触发函数 function( event ) 参数列表中添加一个参数， 这个参数名， 我们习惯取名为 event ，这个event就代表着相应的事件。

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    <style type="text/css">

        #areaDiv {
            border: 1px solid black;
            width: 300px;
            height: 50px;
            margin-bottom: 10px;
        }

        #showMsg {
            border: 1px solid black;
            width: 300px;
            height: 20px;
        }

    </style>
    <script type="text/javascript" src="jquery-1.7.2.js"></script>
    <script type="text/javascript">

        // 1.原生javascript获取 事件对象
        window.onload = function () {
        	document.getElementById("areaDiv").onclick = function (event) {
        		console.log(event);
        	}
        }
        // 2.JQuery代码获取 事件对象
        $(function () {

            //使用bind同时对多个事件绑定同一个函数,也只需要写一个event，通过event.type来区分不同的事件

            $("#areaDiv").bind("mouseover mouseout",function (event) {
                if (event.type == "mouseover") {
                    console.log("鼠标移入");
                } else if (event.type == "mouseout") {
                    console.log("鼠标移出");
                }
            });


        });



    </script>
</head>
<body>

<div id="areaDiv"></div>
<div id="showMsg"></div>

</body>
</html>
```



实例代码：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    <style type="text/css">
        body {
            text-align: center;
        }
        #small {
            margin-top: 150px;
        }
        #showBig {
            position: absolute;
            display: none;
        }
    </style>
    <script type="text/javascript" src="script/jquery-1.7.2.js"></script>
    <script type="text/javascript">
        $(function(){
            $("#small").bind("mouseover mouseout mousemove",function (event) {
                if (event.type == "mouseover") {
                    $("#showBig").show();
                } else if (event.type == "mousemove") {
                    console.log(event);
                    $("#showBig").offset({
                        /*
                        如果不加上这个10，就会导致鼠标从图片的左上角，向右下移动时，浏览器会判定此时鼠标在大图
                        片之上，从而判定事件为mouseout，然后就会造成一闪一闪的视觉效果
                         */
                        left: event.pageX + 10,
                        top: event.pageY + 10
                    });
                } else if (event.type == "mouseout") {
                    $("#showBig").hide();
                }
            });
        });
    </script>
</head>
<body>

<img id="small" src="img/small.jpg" />

<div id="showBig">
    <img src="img/big.jpg">
</div>

</body>
</html>
```

