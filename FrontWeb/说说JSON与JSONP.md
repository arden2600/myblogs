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

---
* Json格式(规则)
JSON能够以非常简单的方式来描述数据,XML可以做的，json都可以做到。<br>
1.Json只能有两种数据类型描述符，大括号`{}`和方括号`[]`,其余英文冒号`:`是映射符,英文逗号`,`是分隔符，英文双引号""是定义符。<br>
2.大括号{}用来描述一组"不同类型的无序键值对集合"(每个键值对可以理解为OOP的属性)，方括号[]用来描述一组"相同类型的有序数据集合"(类似OOP数组)<br>
3.上述两种集合若有多个子项，则可以通过英文逗号`,`进行分隔。<br>
4.键值对以英文冒号:进行分隔，并且建议键名都要加上英文双引号"",以便于不同语言的解析。<br>
5.Json内部数据类型无非就是字符串，数字，布尔，日期，null这么几个，字符串必须用双引号引起来，其余的都可以不用，日期类型比较特殊，有兴趣可以自己去看看。<br>
只是建议如果客户端没有按照日期排序功能需求，那么可以把日期座位字符串传递就好，可以省去很多麻烦。<br>

---
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
## 什么是JSONP?
