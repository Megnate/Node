通过地址栏访问控制器（`Controller`），通过`Models`过滤校验，在`Views`中显示出来

插值字符串：`$`，表示在这其中的字符串的几对括号之间`({})`可以放值几乎任何表达式，最终会显示出来该表达式的结果的`ToString`表示，也就是会对写在`{}`之中的表达式调用`String.Format`函数，使用这个符号可以使代码更加简洁

### 文件夹

`App_Start`：配置文件夹
	`BundleConfig.cs`：打包器——打包`js,css`文件
	`FilterConfig.cs`：过滤器——书写过滤规则，哪些能进哪些不能进
	`RouteConfig.cs`：路由配置，这里面现在可以添加方法名（如`Index`），默认配置的就是`Index`

`Content`：
	存放`bookstrap`的一些样式以及一个`site.css`

`Controllers`：控制器文件夹
	`HomeController.cs`中的方法名在`Views/Home`文件夹中会存放对应的页面，调用不同的`action`方法就可以出现不同的页面内容
	可以右键添加**控制器**（空白的MVC），然后添加视图（可以选择是否使用**母板页**：`_Layout.cshtml`）
	每一个`Controller`都会在`Views`文件夹中多出一个对应的文件夹

`fonts`:存放`bookstrap`的字体，也就是图标文件

`Views`：
	底下有`Home`和`Shared`两个文件夹以及一个文件`_ViewStart.cshtml`
	`Home`文件夹存放对应`action`页面，可以在代码页面指定**母板页**：`Layout = "模板页位置"`，这样就可以修改**特定页面**默认的**母板页**，如果想直接修改**母版页**，在`_ViewStart.cshtml`中直接修改
	`Shared`文件夹存放的是公共，共享的
		其中有`_Layout.cshtml`是完整的页面（有头有尾，中间内容就是`about`），存在一个`RenderBody()`占位符，后期会被替换成`Home`文件夹中的`action`页面文件中的代码
	`_ViewStart.cshtml`中存在一句话：`Layout = "~/Views/Shared/_Layout.cshtml";`，其中`~/`表示根目录，这个定位到的就是**母板页**，所以前面可以套用就是在这设置了

#### 打包器

打包器可以结合debug来验证是否使用了需要打包的文件。`<compilation debug="true" targetFramework=>`，只有是`true`的时候才会在网页源代码看到调用了什么文件

`bundless.Add`添加一个打包设置。`new ScriptBundle`表示添加的是一个**脚本打包器**，`new StyleBundle`表示添加一个**样式打包器**

```C#
bundles.Add(new ScriptBundle("~/bundles/jquery").Include("~/Scripts/jQquery-{version}.js"))
    // 这个{version}利用后期升级，无论jQuery的版本号怎么变化都不影响
```

首先跟着的路径是**打包器**的名字路径，`Include`表示这个打包器包含了符合后面路径的文件

文件名包含有`min`的表示是压缩文件，会自动忽略，包含有`vsdoc`表示是提示文档文件，这样在其他地方使用这个路径`"~/bundles/jquery"`引用的文件就会忽略这些文件

在`_Layout.cshtml`中会引用**脚本打包器**中添加的路径：

```c#
@Scripts.Render("~/bundles/jquery")
@Scripts.Render("~/bundles/bootstrap")
```

在`_Layout.cshtml`中会引用**样式打包器**中添加的路径：

```c#
@Styles.Render("~/Content/css")
```

### 页面

#### 管道

页面栏的地址是可以调用：`Controller`和其中的方法`action`，实际上是通过**调用管道**来实现的：每一个**请求**都会进入**MVC框架**的**管道**之中，中间件会解析地址栏的信息——`Controller`名字和`action`名字，再通过反射的方式实例化`Controller`并调用`action`方法

#### 方法

在`action`方法中存在**内置对象**

- Request，请求

  服务器接收客户端数据

  `return Content(Request.QueryString["name"])`：其中`QueryString`表示查询字符串，在地址栏中是——`?name=12`也就是**问号**后面的内容

  ​	`Request.QueryString`对应的是`get请求`，请求后面**参数**的数据，如果存在多个参数，在地址栏中可以加入`&`来拼接

  `Request.Form["loginname"]`对应的使`post`请求，也就是传的是一个**表单**，一般在`html`页面中的`<form>`中实现

  `Request.MapPath()`将虚拟路径（相对路径）转换成物理路径（绝对路径）

  `Request.Files()`是`post`请求的文件（文件上传）

  `Request.Headers[]`采用请求头的方式将其中的`header`的信息获取，和**爬虫**相似，也可以增加`header`的元素

```html
//在html页面中规划
<body>
<form action="/Home/FileData" method="post" enctype="multipart/form-data">  //只有这样才可以上传文件
    <input type="file" name="file"/>  //因为上传的是文件，所以type就是file
    <button>
        提交
    </button>
    </form>
</body>

//在Controller文件中接收这个文件
public ActionResult FileData(){
	//SaveAs()方法需要物理路径（绝对路径），文件名可以使用自己的名字：Request.Files["file"].FileName
	Request.Files["file"].SaveAs(Request.MapPath("虚拟路径"+Request.Files["file"].FileName))  //这里的名字就是上传文件的name名字
	return Content("ok")
}
```

- Response，响应

  服务器给客户端的结果
  `Response.Write`向客户端输出内容

  `Response.Redirect("http://www.baidu.com")`表示重新定向，重新请求另一个路径

  `Response.StatusCode`表示的是此时的**状态码**，比如：404

- Session，会话

  以**键值对**的方式存储**重要的数据**

  可以认为是所有人自己的数据存储空间（持续时间是20分钟，在新的动作之后会重新计时）

  数据保存在服务器中，存储的安全性比`Cookie`要高，但是性能比较差，因为存储在服务器中会占用空间（并发的人较少）

  所以一般只存储**账号信息**，90%用于身份识别，存储少量重要数据

  此时所有的`action`都可以在**规定的时间**内读取到`Session`的数据：`Session["user"]`

  删除**Session**中的内容有两种办法：`Session.Clear()`和`Session.Abandon()`

```c#
public ActionResult SessionData(){
    //将post请求过来的数据存储在Session中
    Session["user"] = Resquest.Form["user"]
}

//在html页面中
<body>
<form action="/Home/SessionData" method="post" enctype="multipart/form-data">  
    <input type="text" name="user"/>  
    <button>
        提交
    </button>
    </form>
</body>
```

- Cookie，缓存（数据的客户端存储）

  客户端存储的数据——不安全数据，可以在客户端修改

  想要存储不想被修改的数据，需要使用`jwt`，因为目前某些客户端不支持`cookie`

  保存使用：`Response`，请求使用：`Request`，移除使用：修改**时效性**

  ```c#
  //添加新的Cookie
  public ActionResult CookieSave(){
      //发过去的时候cookie就过期了，所以需要加入保存时间
      Response.Cookies.Add(new HttpCookie("cookie的名字"){
          value="123",
          //表示在浏览器保存7天的时效性
          Expires = DateTime.Now.AddDays(7)
      });
  }
  
  //获取cookie的值
  public ActionResult GetCookie(){
      return Content(Request.Cookies["cookie名字"].Value);
  }
  
  //移除cookie值
  public ActionResult CookieClear(){
      //发过去的时候cookie就过期了，所以需要加入保存时间
      Response.Cookies.Add(new HttpCookie("cookie的名字"){
          //表示时效性很早就过期了
          Expires = DateTime.Now.AddDays(-1)
      });
  }
  ```

- Application，当前网站对象

  也是在服务器端，但是是整个网站共享的，使用较少，使用方式和`Session`基本相同

  `HttpContext.Application["user"]="123"`，无论换不换浏览器都可以获得`Application`的值

- Server，服务器对象

  包含服务器的常用方法，主要是进行转换工作

  `Server.Transfer("~/Home/HomeDemo")`：应用于转发，路径不变，内容发生变化

  `Server.HtmlEncode/HtmlDncode/UrlEncode/UrlDncode`都是转义内容

### 推数据

添加空白的MVC界面，首先添加控制器`HomeControllers`，对该方法进行添加视图操作（右键方法名），在`Views`文件夹下因为之前添加的**控制器**而出现的文件夹`Home`出现了新的文件`Index.cshtml`（默认的就是`Index action`），在根目录下的`Content`文件夹下安装了相应的包

将数据推到`Index.cshtml`文件中有以下几种方法：
	一般**传递不主要的数据**，比如新闻界面旁边的广告页面

- `ViewBag`，对应的页面可以在`ViewBag`中获取数据

  ```c#
  public ActionResult Index(){
      ViewBag.Content = "这是Controller里的数据";
      return View();
  }
  
  //在相应的页面中，也就是Index.cshtml中写入以下代码，在这其中，灰色底色显示的代码是C#代码，黑色底色的是html代码
  @ViewBag.Content
  ```

  `ViewBag`的参数类型是动态类型，也就是说属性和方法是没有固定的，也就是不一定是`Content`，也可以是别的

- `ViewData["Age"] = 40  取出： @ViewData["Age"]`此时**取出写成**`@ViewBag.Age`也是可以的，因为都相当于集合，存储方式是一样的

- `TempData["hello"] = "world"  取出： @TempData["hello"]`和`Session`类似，可以**跨页面**，但是存储之后，**只读取一次**数据就被**清除**

### View（）

这里通常用来**传递主要的数据**

这里面可以存放：`object model 和 string ViewName（换母版页） 和 IView view 和 什么也不放，使用当前页面和默认母板页 `

在`model`文件夹下面可以新建一个类`Student`，在其中写入`get set方法`，在`action`页面调用：

```c#
public ActionResult ShowData(){
    return View(new Student(){
        Id = 1,
        Name = "张三",
        Age = 20
    });
}

//在视图中获取放在View中的数据，也就是对应的ShowData.cshtml页面

//需要先声明传递来的数据类型是什么
@model WebApplication1.Models.Student   //其中WebApplicaiton1是项目的解决方案名字，Student是自己所新建的类
    只有加上了这个视图才会变成强类型视图，才会出现提示。没有model声明就是弱类型视图

@Model.Id  @Model.Name  @Model.Age
    
这里面的注释方法：@* 这里是注释内容 *@
```

#### 重载显示页

`return View("ShowData2", new Student())`，此时指定的页面不再是`ShowData.cshtml`，而是`ShowData2.cshtml`

```c#
public ActionResult ShowData(){
    return View("ShowData2", new Student(){
        Id = 1,
        Name = "张三",
        Age = 20
    });
} //此时页面就发生了改变
```

#### 重载母版页

```C#
public ActionResult ShowData(){
    return View("ShowData2","_Layout1", new Student(){
        Id = 1,
        Name = "张三",
        Age = 20
    });
} //此时母页面就发生了改变，原先的母版页是_Layout
```

原先的母版页（默认的）存放在`Views/Shared/_Layout.cshtml`，是通过`Views/_ViewStart.cshtml`文件来指定母版

自己写的**新母版**也放在`Views/Shared`文件夹里面

`View("显示页名", "母版页名", 主要数据类)`，这种形式就是常见的，三个参数都可以去掉，也可以保留其中几个

==Tips==：也可以不在这指定母版页，可以在新的显示页中指定母版页

### 通过参数传递数据

直接获取`Request.QuertString["name"]`，这个和`return Content(Request.QueryString["name"])`是一样的

```c#
public ActionResult Index(string name){
    return Content(name);
}

//在浏览器的地址栏中填入：localhost:55065/Demo/Index?name=123

这样在页面上显示出来的就是123
```

推荐使用**参数**来传递数据，因为这个**参数**不仅仅是`get参数`，也可以得到`post数据`：

```c#
public ActionResult ShowData(){
    return View();
}
//右键给ShowData添加视图（也就是添加页面），在其中获取表单数据

public ActionResult Index(string name){
    return Content(name);
}
```

```html
<!--这是ShowData产生的页面代码-->
<form action="/Demo/Index" method="post">
    <input type="text" name="name"/>
    <button>
        提交
    </button>
</form>
```

此时在`localhost:55065/Demo/ShowData`页面上提交了数据，就会跳转到`localhost:55065/Demo/Index`页面显示出来

==Tips==：如果仅仅知识想让`Index`获得`get请求数据`，那么可以在这个代码体前面加上：`[HttpGet]`（在方法外面），`post请求数据`对应`[HttpPost`]

#### 多个参数传递

当参数越来越多，那么代码也会越来越长，所以不提倡在多参数时使用

多参数时，可以将多个参数保存在一个**对象**之中（写一个类），比如模拟登录界面，就需要写一个登陆的类，类名：`LoginViewModel`

```c#
public class LoginViewModel
{
    public string Email{get; set;}
    public string Password{get; set;}
}
```

在**控制器**中写传递数据代码，此时的控制器名字是：`DemoController`

```c#
public ActionResult ShowData(){
    return View();
}

public ActionResult Login(Models.LoginViewModel model)
{
    if (model.Email == "admin" && model.Password == "123")
        return Content("ok");
    else
        return Content("no");
}
```

此时对应的`action视图`页面，名字是：`ShowData.cshtml`

```html
<!--此时的name应该和属性表一一对应-->
<form action="/Demo/Index" method="post">
    <input type="text" name="Email"/>
    <input type="password" name="Password"/>
    <button>
        提交
    </button>
</form>
```

将多个参数写成一个对象的**优点**：
	可以对参数做一定的**限制**

```c#
public class LoginViewModel
{
    [Required, StringLength(10, MinimumLength=2)]
    public string Email{get; set;}
    
    [Required, MinimumLength(2)]
    public string Password{get; set;}
}
//这种叫做：数据注解，数据注解是判断数据是否合格的标准.
数据注解的校验都是服务器端的校验
```

```c#
public ActionResult ShowData(){
    return View();
}

public ActionResult Login(Models.LoginViewModel model)
{
    //在使用对象中的数据时，都需要判断是否符合数据注解
    if (ModelState.IsValid)
    {
        if (model.Email == "admin" && model.Password == "123")
        	return Content("ok");
    	else
        	return Content("no");
    }
    return Content("error，数据有误");
}
```

一般情况下推送数据页面**成对出现：**

```c#
[HttpGet]
//这个就是专门用来显示的
public ActionResult Login()
{
    return View();
}

[HttpPost]
//这个就是专门用来传递数据的
public ActionResult Login(Models.LoginViewModel model)
{
    if (ModelState.IsValid)
    {
        if (model.Email == "admin" && model.Password == "123")
        	return Content("ok");
    	else
        	return Content("no");
    }
    return Content("error，数据有误");
}
```

### 添加JS脚本作提示

右键`Login(Models.LoginViewModel model)`添加视图，选择模型为`Create`，模型类为`LoginViewModel(WebApplication1.Models)`，引用脚本库

此时会创建一个新的文件：`Login.cshtml`，此时登录所需要的填写提示就有了

#### 文件分析

`Login.cshtml`

`class = "form-group"`一般表示`{}`内部的东西是一组

`@using (Html.BeginForm()){}`：这一段表示会释放其中`BeginForm()`生成的东西，其生成的就是`<form action="/Demo/Index" method="post"></form>`，将`{}`中的内容放入`<form>`标签中，此时没有传入任何的参数，那么表示这个是**针对当前页面**——`Login`，**传参**的话可以给别的`action`或者**控制器**提交

部分**页面上需要修改**的地方，最好都是在**对象中修改**，也就是：`LoginViewModel`（比如将显示的`Email`变成`账号`）：

```c#
public class LoginViewModel
{
    [Display(Name = "账号")]   //如果不存在[Display]这个特性的时候，默认会显示属性的名称
    [EmailAddress]   //表明这是一个邮箱，会判断输入的是否符合邮箱格式，不是的话会提示
    [Required, StringLength(10, MinimumLength=2)]
    public string Email{get; set;}
    
    [Display(Name = "密码")]
    [DataType(DataType.Password)]   //表示这是一个密码类型，输入时不会显示密码
    [Required, MinLength(2)]
    public string Password{get; set;}
}

```

对于页面其他部分，如 `input`标签，可以按照`HTML`格式来进行修改

##### 防伪标识

`@Html.AntiForgeryToken()`

防止他人恶意改动代码并提交数据，所以会存在防伪标识码：`@Html.AntiForgeryToken()`，并且需要在`post请求`处理的时候加上**防伪戳**：`[ValidateAntiForgeryToken]`，也就是`form表单`提交的时候**防止他人伪造**需要加上这个，也就是此时我们之前**自己创建**的`ShowData.cshtml`就**无法传递数据**给这个页面了

```c#
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Login(Models.LoginViewModel model)
{}
```



##### 错误汇总

`@Html.ValidationSummary`

此时提示错误信息就**需要改变**了，有多少个错误就会被累计多少个错误

```c#
public ActionResult Login(Models.LoginViewModel model)
{
    if (ModelState.IsValid)
    {
        if (model.Email == "admin@qq.com" && model.Password == "123456")
            return View();
        else
        {
            //模型状态添加一个模型错误
            ModelState.AddModelError("这里一定要是空的，不加入任何东西", "这里加上错误的信息");
            //此时这里显示的错误信息就是@Html.ValidationSummary这个地方显示的
            return View(model);
        }
    }
    ModelState.AddModelError("", "数据有误");
    return View(model);
}
```

##### 其余代码

`@Html.LabelFor`显示的是`[Display(Name = "账号")]`

`@Html.EditorFor`显示的是文本框，也就是`[EmailAddress]`（没有指定，所以默认是文本框）和`[DatType(DataType.Password)]`（指定了是`Password`，所以是密码框）

`@Html.ValidationMessageFor`显示在文本框下面的错误提示，表示数据不符合要求

##### 修改错误提示

`[EmailAddress(ErrorMessage = "邮件名有误")]`

`[Required(ErrorMessage = "请填写邮箱"), StringLength(10, MinimumLength = 2, ErrorMessage = "2~10个字符")]`

##### 修改样式

可以直接按照HTML的格式进行布局的修改，如果想修改**灰色底**的代码，可以使用`htmlAttributes：new {}`，在这其中填写`HTML`属性

`@Html.LabelFor(model => model.Email, htmlAttributes:new {@class="redFont", title="鼠标提示"})`，因为`class`是关键字，所以需要在前面加上`@`表明这是`HTML`中的**类选择器**的名字，也可以添加其余的`html`内容的代码，在`{}`内

```html
<style>
    .redFont {
        color:red
    }
</style>
```

