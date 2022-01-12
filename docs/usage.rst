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
.. code-block:: javascript

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
.. code-block:: javascript

    var test1 = {
    service: function(req, resp) {
            resp.body.append("你看，这是我的第一个服务端小程序。");
            return resp;
        }
    };


那么，如何才能让它运行起来呢？我们还要在config.js文件中进行配置这个小程序要服务的Http路径。我们编辑下config.js
找到:
.. code-block:: javascript

    endpoints: [
        {path: "/", servlet: "./bin/index.js", name: "index"}]


加入下面一行配置，为我们的请求路径/test1绑定服务端小程序。
.. code-block:: javascript

        {path: "/test1", servlet: "./servlet/test1.js", name: "test1"}


一切完成后，我们启动起来，就可以在访问http://localhost:9999/test1，我们将在浏览器看到我们预期的效果了。

请求和响应的对象结构
--------------

在上面的小节中，我们看到一个标准的服务端小程序是一个有service函数的JS变量，这个函数具有两个参数，分别为req和resp。
当然，实际使用中框架本身不会对这两个参数的命名进行严格控制约束。如果你愿意，你写成service(request,response)也不是不可以。
但是，这两个变量所对应的就是请求和响应的处理对象。第一个参数负责携带请求相关数据和能力，第二个参数负责携带响应相关数据和能力。
那么他们的结构优势什么样呢？
.. code-block:: javascript

    request={
        headers:
        method:
        uri:
        params:
        body:
    }

    response={
        headers:
        body:
        code: 200,
        msg:
    }


以上就是请求和响应参数对象的字段信息，其中headers是个Map，根据key取出的项是List，params也同样如此，但是params是标准的JS
变量类型所定义出来的，这一点不同于headers是源自Java内生的类型系统。method和uri都是字符串类型，分别是请求方法(GET/POST....)
body属性，一般情况下都会处理成JSON对象格式，当请求方法为PATCH的时候，会作为原生字符串格式。

响应参数的body是Java中StringBuffer类型，所以一般可以直接使用append方法进行添加想要输出的内容，msg同样是StringBuffer格式，其
默认值都是new StringBuffer("");属性code默认值是200，这个属性可以完全交由开发人员进行重新赋值。

进阶之访问数据库
-------------
在我们了解了服务端小程序如何开发之后，我们接下来尝试快速访问数据库，进行数据库的数据查询处理。通常，我们如果访问关系型数据库，那MySQL
来举例，大致分为以下几步。

* 注册JDBC驱动类
* 获取数据库连接
* 执行SQL，或者是参数化的SQL
* 返回结果，映射处理成POJO集合
* 关闭数据库连接

在这里，我们假设数据库中已经有了一张表名为person的数据表，表中定义了id,name,age,address,birthday,pet_id这几列。第一步我们要做的是
把数据库对应的驱动jar包放在lib目录下，同时在config.js文件中进行相关的数据库连接参数配置。一份完整的配置信息应该如下：
.. code-block:: javascript

    db: {
        url: "jdbc:mysql://192.168.10.60:3306/test",
        user: "root",
        pass: "123qwe123",
        driver: "com.mysql.cj.jdbc.Drvier",
        poolSize:10,
    }


在完成配置后，我们即可开始开发我们的数据库访问小程序了。接着请看示例代码：
.. code-block:: javascript

    importPackage(org.apache.commons.dbutils, org.apache.commons.dbutils.handlers, java.sql, java.util, java.time.format)
    var person = {
    service: function (req, resp) {
        if (req.params) {
            if (req.params.get("id")) {
                resp.body= this.queryPerson(req.params.get("id"));
            }
        }
        resp.msg.append("OK");
        return resp;
    },
    queryPerson: function (id) {
        try {
            var connection = $.jdbc();
            var run = $.sql();
            var result = run.query(connection, "select id,name,birthday from person where id = ? ", $.asMapList, parseInt(id));
            $.jdbc(connection);
            return $.format(result);
        } catch (ex) {
            //println(ex);
        }
        return "NOT——FOUND";
    }
    };


在上面的代码中，我们按照小程序规范定义了一个含有service函数的变量，当然也可以称之为对象。但是，与之前不同的是，这个对象中海油另一个函数，这个名为
queryPerson的函数只有一个参数，参数名为id，仔细看函数的实现逻辑大致为：根据传入id查询一个person表中的数据行，将id,name,birthday三列进行返回。

* 1.获取连接 定义一个变量来接收 $.jdbc(); 的返回值
* 2.准备一个SQL执行者 $.sql();
* 3.执行一段SQL语句
* 4.还回数据库连接 $.jdbc(connection);
* 5.对数据进行格式化处理，并返回.

此处我们没有看到定义Java中的POJO类，直接将数据经过格式化后返回，那么我们如果现在启动后，会看到什么结果呢？
我们在配置文件中加入路径绑定信息：
.. code-block:: javascript

    {path: "/persons", servlet: "./servlet/person.js", name: "person"}


启动后，访问http://localhost:9999/persons?id=2。即可看到以下内容：
.. code-block:: javascript

    {"code":200,"msg":"OK","body":[{"id":2,"name":"test","birthday":"2022-03-26 10:34:48"}]}


很明显，这里body的集合中有一条对应了数据库中的数据行的JSON对象数据。并且日期完成了格式化。此时，应该有很多朋友会疑问，这个$到底是个什么东西呢？
关键的5行代码里，出现了5次它的身影。其实呢，$就是我们这个框架的核心能力的体现，我们关键的能力都将集成在这个$上，这个$如果是会用Jquery的朋友看到
应该会无比亲切吧，是的，准确的说这个$就是向Jquery的一种致敬，将less is more的内涵发扬光大。需要注意的是，这里对数据库的访问上是集成了连接池的
能力的，用完记得使用$.jdbc(back_var);将数据库连接还回池中。

进阶之dbutils.js
---------------
框架中的$已经提供了简便的访问数据库的能力，那么dbutils.js又是什么鬼东西呢？不用着急，我们慢慢来看。dbutils本身也是标准化的一个服务端小程序，它作为
框架一个标准组件而附带，当然开发者是否使用，完全取决于配置。
