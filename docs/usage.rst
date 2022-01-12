========
Usage
========

Tropic的目录结构
--------------
* /bin
* /lib
* /log
* /patch
* /servlet
* app.js
* config.js
* start.bat
* start.sh

在/bin目录下有四个文件：

* /bin/bind.js
* /bin/dbutils.js
* /bin/server.js
* /bin/index.js

这四个文件中server.js是框架的核心，index.js是框架的默认首页响应程序脚本，等同于index.html/index.jsp之类
dbutils.js是对数据库服务化的一种模板化能力的提炼，将数据库访问SQL能力换成另一种语法。
bind.js是整个框架的一种动态绑定能力模块，在后面的章节中展开介绍。

/lib目录存放我们所需要的jar包，这些jar包将在程序启动时加载至JVM中。
/log目录是我们存放程序运行时日志的目录，所有程序产生的日志将会写入到该目录下。
/patch目录是bing.js所提供的能力向服务器打补丁时所提交的代码所存放的目录
/servlet目录是我们所有开发的小程序的存放目录
app.js 是程序的主启动程序，相当于main函数。
config.js 是我们程序的集中配置文件，所有的配置信息将在这个配置文件中进行分类配置。
start.bat和start.sh分别是对应Windows和Linux系统的启动脚本。

快速上手
--------------
现在，我们开始编写我们的一个服务端小程序。首先我们要先了解下一份标准的服务端小程序应该是什么样子。
我们重新回顾下，/bin/index.js的样子：

    var index = {
    service: function(req, resp) {
            try {
                resp.body.append("Hello!! Welcome to Tropic engine.");
                resp.msg.append("Version:1.0");
            } catch (e) {
                println(e);
            }
            return resp;
        }
    };

整体上来讲，我们定义了一个Javascript变量，这个变量的名字是index，这个变量是个Object类型的变量，并且拥有一个命名为
service的方法，这个方法有两个参数，分别为req何resp（对应request和response）。整个方法中的代码就是向resp的body中
添加了一段字符串，以及向msg属性中添加了另一段字符串。最后，直接return resp;结束。

看上去很简单，对吧？
我们可以照着它来写另一个我们可以自由发挥的服务端小程序。我们在servlet目录下新建一个test1.js，其内容如下
    var test1 = {
    service: function(req, resp) {
            resp.body.append("你看，这是我的第一个服务端小程序。");
            return resp;
        }
    };
那么，如何才能让它运行起来呢？我们还要在config.js文件中进行配置这个小程序要服务的Http路径。我们编辑下config.js
找到:
        endpoints: [
            {path: "/", servlet: "./bin/index.js", name: "index"},
添加
        {path: "/test1", servlet: "./servlet/test1.js", name: "test1"}

一切完成后，我们启动起来，就可以在访问http://localhost:9999/test1，我们将在浏览器看到我们预期的效果了。

请求和响应
--------------

在上面的小节中，我们看到一个标准的服务端小程序是一个有service函数的JS变量，这个函数具有两个参数，分别为req和resp。
当然，实际使用中框架本身不会对这两个参数的命名进行严格控制约束。如果你愿意，你写成service(request,response)也不是不可以。
但是，这两个变量所对应的就是请求和响应的处理对象。第一个参数负责携带请求相关数据和能力，第二个参数负责携带响应相关数据和能力。
那么他们的结构优势什么样呢？
    request={
        headers:
        method:
        uri:
        params:
        body:
    }

    response={
    {
        headers:
        body:
        code: 200
        msg:
    }

以上就是请求和响应参数对象的字段信息，其中headers是个Map，根据key取出的项是List，params也同样如此，但是params是标准的JS
变量类型所定义出来的，这一点不同于headers是源自Java内生的类型系统。method和uri都是字符串类型，分别是请求方法(GET/POST....)
body属性，一般情况下都会处理成JSON对象格式，当请求方法为PATCH的时候，会作为原生字符串格式。

响应参数的body是Java中StringBuffer类型，所以一般可以直接使用append方法进行添加想要输出的内容，msg同样是StringBuffer格式，其
默认值都是new StringBuffer("");属性code默认值是200，这个属性可以完全交由开发人员进行重新赋值。