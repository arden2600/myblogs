## 前言<br>
* 说到ajax就会不可避免的面临两个问题，**第一个是AJAX以何种格式来交换数据?第二个是跨域的需求如何解决?**，这两个问题目前都是有着不同的解决方案，<br>
比如数据可以用自定义字符串或者用XML来描述，跨域可以通过服务器端代理来解决。但是到目前为止最被崇拜或者说首选方案还是用**JSON来传递数据交互，靠JSONP来跨域**。
<br>
* JSON和JSONP虽然只有一个字母的差别，但是其实根本不是一回事：JSON是一种数据交换格式，而JSONP是一种依靠开发人员的聪明才智创造出来的一种非官方跨域数据交互协议。<br>
我们可以拿`谍战片`来打个比方，JSON是地下党用书写和交换情报的`暗号`,而JSONP则是把暗号书写的情报传递给自己同志(**对方相当于跨域**)时使用的接头方式。<br>可以再打个比方，比如以前的`写信`，写信的格式就相当于json，至于写好的信怎么传递到指定对方那里，是用海陆空什么运输方式，就是jsonp来定制。<br>
总的来说，JSON是一个描述信息格式，用于`规范存储格式`的；JSONP则是用于信息传递双方约定的方法，是用来`保证安全传输数据的协议`。

---
## 什么是Json?
* Json的优点<br>
1.基于存文本，则可跨平台传递，及其简单。<br>
2.javascript原生支持，后台语言几乎全部支持。<br>
3.轻量级数据格式，占用字符数量极少，特别适合互联网传递，占用少流量。<br>
4.可读性较强，虽然比不上xml那么一目了然，但是在规范缩进格式后，还是很容易看懂的。<br>
5.容易编写和解析，当然前提是你要知道json的数据格式。

<br>
* Json格式(规则)<br>
JSON能够以非常简单的方式来描述数据,XML可以做的，json都可以做到。<br>
1.Json只能有两种数据类型描述符，大括号`{}`和方括号`[]`,其余英文冒号`:`是映射符,英文逗号`,`是分隔符，英文双引号""是定义符。<br>
2.大括号{}用来描述一组"不同类型的无序键值对集合"(每个键值对可以理解为OOP的属性)，方括号[]用来描述一组"相同类型的有序数据集合"(类似OOP数组)<br>
3.上述两种集合若有多个子项，则可以通过英文逗号`,`进行分隔。<br>
4.键值对以英文冒号:进行分隔，并且建议键名都要加上英文双引号"",以便于不同语言的解析。<br>
5.Json内部数据类型无非就是字符串，数字，布尔，日期，null这么几个，字符串必须用双引号引起来，其余的都可以不用，日期类型比较特殊，有兴趣可以自己去看看。<br>
只是建议如果客户端没有按照日期排序功能需求，那么可以把日期座位字符串传递就好，可以省去很多麻烦。<br>

<br>
## Json实例
```javascript
// 描述一个人

var person = {
    "Name": "Bob",
    "Age": 32,
    "Company": "IBM",
    "Engineer": true
}

// 获取这个人的信息
var personAge = person.Age;

// 描述几个人
var members = [
    {
        "Name": "Bob",
        "Age": 32,
        "Company": "IBM",
        "Engineer": true
    },
    {
        "Name": "John",
        "Age": 20,
        "Company": "Oracle",
        "Engineer": false
    },
    {
        "Name": "Henry",
        "Age": 45,
        "Company": "Microsoft",
        "Engineer": false
    }
]

// 读取其中John的公司名称

var johnsCompany = members[1].Company;

// 描述一次会议

var conference = {
    "Conference": "Future Marketing",
    "Date": "2012-6-1",
    "Address": "Beijing",
    "Members": 
    [
        {
            "Name": "Bob",
            "Age": 32,
            "Company": "IBM",
            "Engineer": true
        },
        {
            "Name": "John",
            "Age": 20,
            "Company": "Oracle",
            "Engineer": false
        },
        {
            "Name": "Henry",
            "Age": 45,
            "Company": "Microsoft",
            "Engineer": false
        }
    ]
}

// 读取参会者Henry是否工程师

var henryIsAnEngineer = conference.Members[2].Engineer;
```
## 什么是JSONP?<br>
* 先说说什么是同源策略，该策略是jsonp产生的原因:<br>

<table frame="border">
  <tr>
    <th>URL</th>
    <th>说明</th>
    <th>是否允许通讯</th>
  </tr>
  <tr>
    <td>http://www.a.com/a.js http://www.a.com/b.js </td>
    <td>同一个域名下</td>
    <td>允许</td>
  </tr>
  <tr>
    <td>http://www.a.com/lab/a.js  http://www.a.com/script/b.js </td>
    <td>同一个域名下,不同文件夹</td>
    <td>允许</td>
  </tr>
  <tr>
    <td>http://www.a.com:8000/a.js http://www.a.com/b.js </td>
    <td>同一个域名下,不同端口</td>
    <td>不允许</td>
  </tr>
  <tr>
    <td>https://www.a.com/a.js http://www.a.com/b.js </td>
    <td>同一个域名下,不同协议</td>
    <td>不允许</td>
  </tr>
  <tr>
    <td>http://www.a.com/a.js http://ip/b.js </td>
    <td>域名和域名对应ip</td>
    <td>不允许</td>
  </tr>
   <tr>
    <td>http://www.a.com/a.js http://script.a.com/b.js</td>
    <td>主域相同，子域不同</td>
    <td>不允许</td>
  </tr>
   <tr>
    <td>http://www.cnblogs.com/a.js http://www.a.com/b.js</td>
    <td>主域不同</td>
    <td>不允许</td>
  </tr>
</table>
<br>
* 特别注意两点<br>
1.如果是`协议和端口`造成的跨域问题，`前台`是无能为力来解决的。<br>
2.在跨域问题上，域仅仅是通过`URL的首部`来识别而不会尝试判断相同的ip地址，即不会判断两个域是否在同一个ip上。<br>

---
* jsonp是如何产生的：<br>
其实网上关于JSONP的讲解有很多，但却千篇一律，而且云里雾里，对于很多刚接触的人来讲理解起来有些困难，小可不才，试着用自己的方式来阐释一下这个问题，看看是否有帮助。<br>
1.众所周知的问题，Ajax直接请求普通文件存在跨域无权访问的问题，不管是静态页面、动态页面、web服务、WCF,只要是跨域请求，一律不准。<br>
2.不过我们发现，web页面上调用js文件时候不受是否跨域影响(不仅如此，还发现拥有`src`,这个属性标签都有跨域能力，比如:`<script>`,`<img>`,`ifrmae`).<br>
3.于是可以判断，当前阶段如果想通过存WEB端（ActiveX控件、服务端代理、属于未来的HTML5之Websocket等方式不算）跨域访问数据就只有一种可能，那就是在远程服务器上设法`把数据装进js格式的文件里`，供`客户端调用`和进一步处理；<br>
4.恰巧我们已经知道有一种叫做JSON的纯字符数据格式可以简洁的描述复杂数据，更妙的是JSON还被js原生支持，所以在客户端几乎可以随心所欲的处理这种格式的数据。<br>
5.这样子解决方案就呼之欲出了，web客户端通过与调用脚本一模一样的方式，来调用跨域服务器上动态生成的js格式文件（一般以JSON为后缀），显而易见，服务器之所以要动态生成JSON文件，目的就在于把客户端需要的数据装入进去。<br>
6.客户端在对JSON文件调用成功之后，也就获得了自己所需的数据，剩下的就是按照自己需求进行处理和展现了，这种获取远程数据的方式看起来非常像AJAX，但其实并不一样。<br>
7.为了便于客户端使用数据，逐渐形成了一种非正式传输协议，人们把它称作JSONP，该协议的一个要点就是允许用户传递一个callback参数给服务端，然后服务端返回数据时会将这个callback参数作为函数名来包裹住JSON数据，这样客户端就可以随意定制自己的函数来自动处理返回数据了。<br>
<br>
## JSONP客户端具体实现<br>
* 不管jQuery也好，extjs也罢，又或者是其他支持jsonp的框架，他们幕后所做的工作都是一样的，下面我来循序渐进的说明一下jsonp在客户端的实现：<br>
1、我们知道，哪怕跨域js文件中的代码（当然指符合web脚本安全策略的），web页面也是可以无条件执行的。<br>
远程服务器remoteserver.com根目录下有个remote.js文件代码如下：<br>
```javascript
alert('我是远程js文件中内容');
```
<br>
本地服务器localserver.com下有个jsonp.html页面代码如下：<br>
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
    <script type="text/javascript" src="http://remoteserver.com/remote.js"></script>
</head>
<body>
 
</body>
</html>
```
<br>毫无疑问，本地客户端页面将会弹出一个提示窗体，显示跨域调用成功。<br>
2、现在我们在jsonp.html页面定义一个函数，然后在远程remote.js中传入数据进行调用。<br>
jsonp.html页面代码如下：<br>
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
    <script type="text/javascript">
    var localHandler = function(data){
        alert('我是本地函数，可以被跨域的remote.js文件调用，远程js带来的数据是：' + data.result);
    };
    </script>
    <script type="text/javascript" src="http://remoteserver.com/remote.js"></script>
</head>
<body>

</body>
</html>
```
<br>
remote.js文件代码如下：<br>
```javascript
localHandler({"result":"我是远程js带来的数据"});
```
<br>
**运行结果：**<br>
运行之后查看结果，页面成功弹出提示窗口，显示本地函数被跨域的远程js调用成功，并且还接收到了远程js带来的数据。很欣喜，跨域远程获取数据的目的基本实现了，但是又一个问题出现了，我怎么让远程js知道它应该调用的本地函数叫什么名字呢？毕竟是jsonp的服务者都要面对很多服务对象，而这些服务对象各自的本地函数都不相同啊？我们接着往下看。<br>
3、聪明的开发者很容易想到，只要服务端提供的js脚本是动态生成的就行了呗，这样调用者可以传一个参数过去告诉服务端`“我想要一段调用XXX函数的js代码，请你返回给我”`，于是服务器就可以按照客户端的需求来生成js脚本并响应了。<br>
看jsonp.html页面的代码：<br>
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">  
<html xmlns="http://www.w3.org/1999/xhtml">  
<head>  
    <title></title>  
    <script type="text/javascript">  
    // 得到航班信息查询结果后的回调函数  
    var flightHandler = function(data){  
        alert('你查询的航班结果是：票价 ' + data.price + ' 元，' + '余票 ' + data.tickets + ' 张。');  
    };  
    // 提供jsonp服务的url地址（不管是什么类型的地址，最终生成的返回值都是一段javascript代码）  
    var url = "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998&callback=flightHandler";  
    // 创建script标签，设置其属性  
    var script = document.createElement('script');  
    script.setAttribute('src', url);  
    // 把script标签加入head，此时调用开始  
    document.getElementsByTagName('head')[0].appendChild(script);   
    </script>  
</head>  
<body>  
  
</body>  
</html>
```
<br>
我们看到调用的url中传递了一个code参数，告诉服务器我要查的是CA1998次航班的信息，而callback参数则告诉服务器，我的本地回调函数叫做flightHandler，所以请把查询结果传入这个函数中进行调用。<br>
OK，服务器很聪明，这个叫做flightResult.aspx的页面生成了一段这样的代码提供给jsonp.html（服务端的实现这里就不演示了，与你选用的语言无关，说到底就是拼接字符串）：(远程服务器后台端根据传递过来的参数，生成返回一段js代码给调用者的前端使用)<br>
```javascript
flightHandler({  
    "code": "CA1998",  
    "price": 1780,  
    "tickets": 5  
});  
```
<br>我们看到，传递给flightHandler函数的是一个json，它描述了航班的基本信息。调用端运行一下页面flightHandler函数，函数会将远程端返回的json数据进行显示，jsonp的执行全过程顺利完成！<br>

---
## JQuery实现jsonp调用<br>
* 看看jquery发送ajax跨域请求页面:<br>
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">  
 <html xmlns="http://www.w3.org/1999/xhtml" >  
 <head>  
     <title>Untitled Page</title>  
      <script type="text/javascript" src=jquery.min.js"></script>  
      <script type="text/javascript">  
     jQuery(document).ready(function(){   
        $.ajax({  
             type: "get",  
             async: false,  
             url: "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998",  
             dataType: "jsonp",  
             jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)  
             jsonpCallback:"flightHandler",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据  
             success: function(json){  
                 alert('您查询到航班信息：票价： ' + json.price + ' 元，余票： ' + json.tickets + ' 张。');  
             },  
             error: function(){  
                 alert('fail');  
             }  
         });  
     });  
     </script>  
     </head>  
  <body>  
  </body>  
 </html>  
```
<br>
## 总结<br>
* ajax和jsonp这两种技术在调用方式上“看起来”很像，目的也一样，都是请求一个url，然后把服务器返回的数据进行处理，因此jquery和ext等框架都把jsonp作为ajax的一种形式进行了封装；<br>
* 但ajax和jsonp其实本质上是不同的东西。ajax的核心是通过XmlHttpRequest获取非本页内容，而jsonp的核心则是动态添加\<script\>标签来调用服务器提供的js脚本。<br>
* 所以说，其实ajax与jsonp的区别不在于是否跨域，ajax通过服务端代理一样可以实现跨域，jsonp本身也不排斥同域的数据的获取。<br>
* 还有就是，jsonp是一种方式或者说非强制性协议，如同ajax一样，它也不一定非要用json格式来传递数据，如果你愿意，字符串都行，只不过这样不利于用jsonp提供公开服务。<br>

**转载自：** [【原创】说说JSON和JSONP，也许你会豁然开朗，含jQuery用例](http://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html)
