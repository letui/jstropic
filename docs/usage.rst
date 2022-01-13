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
那么他们的结构又是什么样呢？

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

进阶之dbutils.js文件
---------------
框架中的$已经提供了简便的访问数据库的能力，那么dbutils.js又是什么鬼东西呢？不用着急，我们慢慢来看。dbutils本身也是标准化的一个服务端小程序，它作为
框架一个标准组件而附带，当然开发者是否使用，完全取决于配置。接下来，我们先来看一段代码：

.. code-block:: javascript

    {
    "table":"person",
    "select":"id,name,address,age,pet_id",
    "filter":"id > 40 and id !=52",
    "limit":"0,5",
    "order":"id desc"
    }

假设，我们设计了一套低代码化的结构性查询语言，按照上面的代码来进行解读，我们大概能够得出这样的意图。我们要查询的目标表是person，我们要查询的字段
是id,name,address,age,pet_id，我们要过滤的条件是 id > 40 and id !=52，我们限制数据返回条件是 0,5 （熟悉mysql的朋友很容易就理解），我们要
排序的字段设置是 id desc。读完以后，作为程序员的朋友应该很清楚这就是一条SQL语句的另一种表达方式了。没错，这就是用JSON的语法来重新定义SQL的能力。
当然，这个能力肯定是受限制的，不能完全等价于SQL的全部能力。

既然如此，我们的意图是当接收到一个HTTP请求，其携带的Body体是以上数据结构的时候，我们如何才能够以一种以不变应万变的来提供数据库访问能力呢？这就是我们的
dbutils.js要来解决的事情了。下面，我们来看下dbutils.js的代码：

.. code-block:: javascript

    importPackage(org.apache.commons.dbutils, org.apache.commons.dbutils.handlers, java.sql, java.util, java.time.format)
    var dbutils = {
    service: function (req, resp) {
        try {
            if (req.body) {
                var connection = $.jdbc();
                var run = $.sql();
                var sql = "select ".concat(req.body.select).concat(" from ").concat(req.body.table).concat(" where ").concat(req.body.filter);

                if(req.body.order){
                    sql=sql.concat(" order by ").concat(req.body.order);
                }
                if(req.body.limit){
                    sql=sql.concat(" limit ").concat(req.body.limit);
                }
                var result = run.query(connection, sql, $.asMapList);
                resp.body = $.format(result);
                resp.msg = "OK";
                resp.code = 200;
                $.jdbc(connection);
            }else{
                resp.code=500;
                resp.msg="request body is not provided";
            }
            return resp;
        } catch (e) {
            println(e);
            resp.msg=e;
            return resp;
        }
    }
    };

一共有三十多行代码，此时我们仍然可以看到$的身影，是的，此时的dbutils.js就是对之前的访问数据库的另一种高度抽象，添加了些参数校验逻辑，那之前的5个步骤
一个也没少。我们只需要在config.js中对其进行配置就可以启用强大的数据库服务化能力了。Tropic框架默认会将dbutils.js注册到/@db路径上，@符号有助于和常规
路径区分开。如果你作为一个开发者不需要这样的通用能力，完全可以取消其在config.js中的注册配置即可。

.. code-block:: javascript

    {path: "/@db", servlet: "./bin/dbutils.js", name: "dbutils"}

以上就是对dbutils.js的建议型配置，喜欢自定义路径的朋友可以根据自己的喜好调整即可，这里就不做测试结果的展示了。

进阶之数据库表服务化
-----------------

可能有这么一种场景，我们需要将某一张数据库表的数据暴露成http-rest服务，我们的预期要求是，简单，高效，快速，轻量，安全，定制化，热部署。看，这样的要求
很高了，如何才能够实现呢？如何才能够优雅的实现呢？其实，仔细思考下就明白了，我们完全可以依托dbutils向这些高要求高目标前进，从而达到“低代码能力”。那么，
接下来我们创建一个服务端小程序，其代码如下:

.. code-block:: javascript

    var db_person = {
        config:{
            table: "person",
            select: "id,name,address",
            filter: "id > 1 and id !=52"
        },
        service: function (req, resp) {
            load("./bin/dbutils.js");
            req.body = this.config;
            return dbutils.service(req, resp);
        }
    };

我们来解读下这一份小程序，其db_person对象，拥有一个名为config的属性，这个属性也是个Object结构，并且刚好符合我上一小节当中对Http请求体的JSON格式要求。
这里就不在过度解读这个config的语义。我们来到service函数内部，仔细看后，会发现只有三行代码。这里出现了一个load函数，这里需要重点说明，这个函数是native函数
是引擎自带的，其作用就是帮我们加载另一个小程序的代码，加载后，下面的代码就可以直接使用被加载的代码中所有定义的能力。紧接着我们将req.body直接赋值为this.config
,然后返回dbutils.service调用结果。到此为止，我们发现，这个小程序中没有任何代码对HTTP请求进行处理，只是简单的将数据库以不透明的定制化将其服务出去。并且，其
service函数中的代码是不需要做任何的改变的，某种意义上来讲，这部分代码就是固化的，是专门针对特定数据库访问做的场景化固化能力范式代码。如果这么理解的话，我们真正的
对于数据库表服务化的要求就变成了对config属性的设置了。只要能够理解上一小节中对SQL的JSON话语义表达定义，那么我们开发数据库服务化就简直易如反掌。

当然，此时，我们仍然是可以对service函数添加自己的业务逻辑的，无非是写参数校验， 响应结果换一种格式等等。

进阶之调用REST服务
---------------

随着现在很多API设计WEB化，很多微服务暴露出来的接口已经完全是承载JSON数据格式的HTTP服务。那么，这就带来更多的调用HTTP接口的场景，做Java开发的朋友
肯定很熟悉一些类似Spring-RestTemplate，Apapche-HttpClient，Ok-Http等等开源框架。这里不一一点评各框架的优劣，Tropic框架本身也是希望集成进来
这种能力，从而方便开发。

既然，要访问Http-rest服务，那么我们可以直接利用之前查询dbutils.js所提供的Http-rest服务。总结下，现在的场景就变成了：我们要提供个Http接口服务，
这个接口服务的实际实现逻辑是当你请求它的时候，它去请求另一个Http-rest服务，将请求回来的数据经过处理（或者不处理）再次返回浏览器（或者其他Http-Client）。这么
听上去，有点类似Http代理的意思。是的，本质上我们写WEB程序，大多数是代理了数据库的能力。所以，此时我们就新创建个小程序文件命名为httpproxy.js。

.. code-block:: javascript

    var httpproxy = {
        config: {
            table: "person",
            select: "id,name,address",
            filter: "id > 50 and id !=52"
        },
        service: function (req, resp) {
            resp = $.post("http://127.0.0.1:9999/@db", null, this.config, true);
            resp.body.push("Hello!!");
            return resp;
        }
    };

同样，我们仍然需要为这个小程序配置下http路径，我们在config.js的endpoints里加入以下代码:

.. code-block:: javascript

    {path: "/pxy", servlet: "./servlet/httpproxy.js", name: "httpproxy"}

接下来，启动我们的程序。在浏览器里访问http://127.0.0.1:9999/pxy，我们将看到以下结果：

.. code-block:: javascript

    {"code":200,"msg":"OK","body":[{"id":50,"name":"P990.6751635416179556","address":"北京市海淀区中关村22号0.8318787196947994"},{"id":51,"name":"P990.6449720409186297","address":"北京市海淀区中关村22号0.7112042891897301"},"Hello!!"]}

以上数据是测试数据，不同使用者并不相同，但是这里我们仔细观察，发现响应结果里面，其中body这个集合中多出来一条字符串数据，内容为"Hello!!"。没错，这一条内容恰恰是我们的代理小程序加进去的。
其关键的代码就是上文中的resp.body.push("Hello!!");这是一段典型JS数组的操作，就不用再做过多解释。我们把关注点转移到神奇的$上，我们又一次发现了这个$对象，用过Jquery的朋友
肯定很熟悉Jquery的Ajax请求有两个很常用的$.get和$.post。没错，这里也是同样的API设计，但是稍微有些不同。我们来仔细看下这一行代码：

.. code-block:: javascript

    resp = $.post("http://127.0.0.1:9999/@db", null, this.config, true);

在这一行代码中，post函数总共有四个参数，第一个参数不用解释也很清楚是个Http地址，第二个参数是null，这个可能不太好理解，没关系我们接着看第三个参数，第三个参数是一个JSON-Object，
同时呢这个对象刚好符合了我们数据库服务化的格式要求，第四个参数是个bool类型的变量，我们这里传入了true。那么让我们解开这里的get和post的庐山真面目吧。

.. code-block:: javascript

    get: function (url, headers, asJson) {
    }

    post: function (url, headers, data, asJson) {
    }

看到上面的代码声明时，所有的疑惑都解开了。post函数比get函数多了个数据对象的传入要求，但是这个data并不限定是严格JSON对象，其他类型也是可以的，因为无论如何它都要被
塞进Http请求报文体当中。后面的asJson是说响应回来的结果要不要完成JSON反序列化，使其成为一个JSON对象。至此，我想各位朋友应该没有疑问了。我们可以这样简单的使用
框架自带的能力来完成Http请求，并且是如此少的代码。

进阶之访问Redis
---------------

考虑到大家在实际开发中，很大部分场景是要使用Redis的，所以框架本身也将Redis的访问能力集成了进来，当然只是集成，其依赖的核心jar包是我们熟知的Jedis。那么我们来看下，
如何访问Redis吧。

.. code-block:: javascript

    var redis = $.redis();
    redis.set("Hi","I'm Tropic");
    redis.get("Hi");
    $.redis(redis);

相信看了上面的代码，所有的朋友应该很容易理解了。这里的$.redis();采用了和$.jdbc();同样的设计，如果你调用函数没有传递任何参数那么就是获取一个Jedis对象，
当你用完了以后，你需要把它还回去，使用方式就是同样的$.redis(back);这里传入刚才的返回值就可以了。细心的朋友可能会疑惑，那么我们的Redis要如何配置呢？
是的，我们当然需要完成初始化配置。我们来看下Redis的连接配置长什么样子。

.. code-block:: javascript

        redis:{
          host:"192.168.10.173",
          port:6379,
          maxIdle:10,
          maxTotal:20,
          maxTimeout:2000
        }

以上就是在config.js中配置redis连接完整信息，可能稍有些不足，比如密码啊，集群啊，等等。目前呢，暂时没有加入这些功能，后续会逐渐完善。
至于详细的Redis访问API我们就不在这里详细展示了，完全可以参考Jedis的API。另外，这里也就不在做完整的Redis访问能力的服务端小程序的示例代码了。

进阶之JSON序列化工具
-----------------

目前，业内已经普遍认可JSON语法来表达数据结构，我们很多朋友也会用很多像fastxml-json，gson，fastjson等等一些框架，这些框架功能很强大帮我们省去
了很多Json序列化和反序列化的工作，我们经常会使用这些框架来完成Entity或者VO之类的序列化要求。但是，其实这都是在Java这种强类型编程语言上的一种
妥协方案而已。然而，如果我们转移到JS领域，我们会发现一切都变了，我们发现声明式的JSON数据结构可以随时使用，熟练掌握JS的朋友肯定都知道默认JSON.parse
和JSON.stringify这两个函数。不过，为了统一API风格，简便开发，框架的核心对象$也提供了JSON序列化和反序列化的函数。其分别为$.toJson和$.fromJson;
这两个函数名，用过谷歌Gson的应该再熟悉不过了。

那么，我们就来不厌其烦的演示下如何使用这两个方法吧。

.. code-block:: javascript

    var input="{\n" +
        "    \"table\":\"person\",\n" +
        "    \"select\":\"id,name,address,age,pet_id\",\n" +
        "    \"filter\":\"id > 40 and id !=52\",\n" +
        "    \"limit\":\"0,5\",\n" +
        "    \"order\":\"id desc\"\n" +
        "}";

    var obj=$.fromJson(input);
    println(obj.table);
    obj.table="NPerson";
    var after=$.toJson(obj);
    println(after);

不出意外的话我们将在控制台看到两行输出：

.. code-block:: shell

    person
    {"table":"NPerson","select":"id,name,address,age,pet_id","filter":"id > 40 and id !=52","limit":"0,5","order":"id desc"}

进阶之日志工具
-----------------

为了方便程序的开发和调试，很多时候需要用到日志框架，在Tropic里，默认是集成了logback作为日志工具框架的。那么，应该如何使用呢？配置文件又在哪里呢？
