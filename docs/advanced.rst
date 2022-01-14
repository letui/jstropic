=============
Advanced
=============

这个章节里我们要展开什么内容呢？准确的说，这部分内容不同于之前的简单使用帮助说明。

* 假如你的程序已经部署上线了，那怎么样才能不停止服务器的情况下来进行升级个别服务端小程序呢？或者说，这种行为叫做打补丁。

* 假如你想要临时停掉某个Http-Path的服务端小程序这又该怎么办呢？

* 假如你想恢复停掉的那个Http-Path的服务端小程序，怎么办？

* 假如你不想用Tropic来开发Web程序，只想用它来一些控制台小程序，又该如何呢？

我们有这么多假如，那么就来看Tropic是如何做到的吧。

热部署能力
--------
无需编译，热部署，总会有无数程序员讨厌编译的繁琐，对着这两大特性垂涎三尺。其实呢，JVM作为一个中间代码解释器，本身就是支持热部署的，有很多字节码
处理框架完全可以在运行时来改变某个Java类的结构和方法，这一点在面向切面技术领域已经屡试不爽了。由于Tropic本身不使用Java字节码技术，而是简单的
使用Java的Javascript引擎能力，所以热部署这个能力就浑然天成了。

接下来演示下，热部署的效果。我们创建个服务端小程序hot.js，其内容如下:

.. code-block:: javascript

    //path: /hot
    var hot = {
        service: function (req, resp) {
          resp.body.append("我喜欢简单。");
          return resp;
        }
    };

我们完成配置后，启动服务器，访问http://localhost:9999/hot。将会在浏览器看到：

.. code-block:: javascript

    {"code":200,"msg":"","body":"我喜欢简单。"}

现在，我们将小程序的代码改以下内容:

.. code-block:: javascript

    //path: /hot
    var hot = {
        service: function (req, resp) {
          resp.body.append("我喜欢热部署能力。");
          return resp;
        }
    };

此时，只需要保存以上代码，在不重新启动服务器的同时，我们重新刷新浏览器，我们将在浏览器看到以下内容:

.. code-block:: javascript

    {"code":200,"msg":"","body":"我喜欢热部署能力。"}

不用为之惊叹，就是这么任性。此时此刻，是不是有点PHP的意思了？原来Java给我们留着这么大的一个惊喜，可是却鲜有人去挖掘。这么爽的特性，显然要比频繁
重启好使多了。

动态绑定能力
----------
动态绑定能力是指允许程序运行期间，动态的将服务端小程序绑定到某个预期的Http-path上来提供服务。我们这里拿之前的hot.js接着举例示范。当然，首先要记得
在我们的config.js中打开这个黑科技。具体配置如下：

.. code-block:: javascript

    server:{
        use_dynamic_bind:true,
        auth_bind_token:"Tropic"
    }

另外，我们还需要准备一个Http-Client测试工具，比如Postman。一切准备就绪后，我们打开Postman。假设，我们需要另一个/sohot路径提供和/hot同样的能力
那么此时，我们准备好以下内容：

动态绑定功能的服务地址是 http://localhost:9999/@bind

我们要发送的报文内容是:

.. code-block:: javascript

    {
    "path":"/sohot",
    "servlet":"./servlet/hot.js",
    "name":"hot"
    }

准备好这些还不够，因为处于安全考虑，我们必须携带token才可以成功请求。token是携带在http请求头里的，其名称为js$auth_bind_token，我们在Postman
设置js$auth_bind_token对应的值为:Tropic。最后，还有一点需要注意，否则是无法成功的。处于安全考虑，由于POST请求太过普通，所以这个动态绑定的功能
使用了PUT请求作为准入限制，请一定记得设置HTTP请求方法为PUT。一切都准备好后，我们用Postman发起请求，不出意外将返回以下内容:

.. code-block::javascript

    {
    "code": 200,
    "msg": "bind for path: /sohot",
    "body": ""
    }

当我们收到这样的返回结果时就代表我们已经绑定成功了，此时，我们访问浏览器地址http://localhost:9999/sohot，将看到以下内容:

.. code-block::javascript

    {"code":200,"msg":"","body":"我喜欢热部署能力。"}

那么如何解绑定呢？

其实解绑定和绑定的动作很相似，地址都是/@bind路径来提供服务，只是解绑定的时候我们需要使用HTTP的DELETE请求方法，请求头里依然要携带令牌，但是
请求体里可以只携带一个path属性即可。

.. code-block:: javascript

    {
    "path":"/sohot"
    }

特别需要注意的是，所有动态绑定的小程序路径，在服务器重启后自动失效。

动态打补丁能力
------------
动态绑定能力已经很强大了，对吧？但其实，更强大的是动态打补丁的能力。这个功能准确的描述来说，是指当你已经上线了一套服务端应用，此时你无法到到服务器上
更换所有的源代码了，这时候就该动态打补丁的功能闪亮登场了。这个功能允许你上传一个服务端程序源代码，并且完成绑定到一个固定的Http路径上来提供服务。
警告，这个功能已经和黑客所熟知的WebShell有些类似了，是个强大但危险的功能。

那我们接下来具体介绍如何使用这个强大的打补丁功能吧。假设你有以下源代码想要提交到服务器上提供Http服务。

.. code-block:: javascript

    //path: /door.jsp
    var hot = {
        service: function (req, resp) {
          resp.body.append("我是个补丁。我更像个后门。");
          return resp;
        }
    };

源代码准备好后，我们打开Postman，键入之前的动态绑定地址http://localhost:9999/@bind。将源代码黏贴在body输入区域中。此外，我们还有好多Http
-header要进行设置，因为我们要高速服务端这个源代码存放的文件名，服务的路径等等。那么Http-header里要填写哪些内容呢？

* js$path 要服务的Http路径 此处示例应该填写/door.jsp
* js$servlet 源代码在服务器上的文件名 此处示例应该填写 hot.js
* js$name 源代码中声明的变量名称 此处示例应该填写 hot
* js$auth_bind_token  安全令牌  此处示例应该填写 Tropic

以上信息都设置好后，我们将HTTP请求方法调整为PATCH，然后点击Postman发送按钮。一切万事大吉后，我们将收到：

.. code-block:: javascript

    {
        "code": 200,
        "msg": "patch for path: [/door.jsp]",
        "body": ""
    }

此时，我们的补丁源码就会发送到服务器上，并且开始为Http路径为 /door.jsp的地址提供服务。

特别需要注意的是，所有的补丁程序在服务器重启后将会全部失效，但是/patch目录下将会保留所有的源代码文件。因为补丁终究是补丁，补丁的服务都应该是临时的
当服务器需要重启的时候，应该已经达到了人工介入更新整体服务应用的时机。

关于$.format和$.asMapList
------------------------

这两个能力在之前的章节中的示例源代码里出现过，那么到底是什么意思呢？

这里，就展开解释下Tropic框架集成的查询关系数据库的依赖jar包commons-dbutils。commons-dbutils是Apache的一个开源数据库访问处理工具包，提供了
简单易用的一些API封装，感兴趣的可以访问:https://commons.apache.org/proper/commons-dbutils/
其核心工具类主要是两个，一个是QueryRunner，另一个是ResultSetHandler，这两个一个负责执行SQL，另一个负责将查询出的数据进行处理。在官方提供的ResultSetHandler
里有一个MapListHander实现，作用是将查出的每一行数据处理成一个Map，列名作key，列值做value，多行数据经过转换后放进一个ArraList里。$.asMapList就是一个
语法糖，免去了写代码时new MapListHandler()的操作。

那$.format是干啥呢？为什么要用$.format呢？$.format是想做一个通用格式化的封装，目前呢主要是用来格式化MapListHandler返回的数据结构，因为数据库中难免
会有些Date和Datetime类型的字段，这些类型是没有办法直接映射成Js变量类型的，在进行toJson的时候会有些问题，所以就需要对齐进行一个预处理。这个函数会
默认将数据库Date列格式化成为yyyy-MM-dd，将数据库Datetime列格式化为yyyy-MM-dd HH:mm:ss格式。

.. code-block:: javascript

    function (maplist) {
        var list = [];
        for (var item in maplist) {
            var row = {};
            for (var key in maplist.get(item)) {
                var val = maplist.get(item).get(key);
                if (val != null && typeof val == "function") {
                    if (val instanceof LocalDateTime) {
                        row[key] = val.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
                    } else if (val instanceof LocalDate) {
                        row[key] = val.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));
                    }
                } else {
                    row[key] = val;
                }
            }
            list.push(row);
        }
        return list;
    }

看了上面的实现代码，很容易就理解了。经过这一系列的转换之后呢，Java的类型就被抹掉了取而代之的是一个填满了Json-Object的数组。在后面的数据使用时
我们就可以使用对象导航的方式了。

最后，补充要说的是，commons-dbutils的功能很强大，有很多ResultSetHandler的默认实现，也提供了POJO类到查询结果集的自动化ORM处理工具类，我们来看下
官网的示例代码。

.. code-block:: java

    QueryRunner run = new QueryRunner(dataSource);

    // Use the BeanListHandler implementation to convert all
    // ResultSet rows into a List of Person JavaBeans.
    ResultSetHandler<List<Person>> h = new BeanListHandler<Person>(Person.class);

    // Execute the SQL statement and return the results in a List of
    // Person objects generated by the BeanListHandler.
    List<Person> persons = run.query("SELECT * FROM Person", h);

上面的代码是映射成POJO类的集合，可是在Tropic框架的使用背景下，我们需要思考个问题，用JS也要强制按照Java实体类那样去写实体类吗？包括Getter和Setter？
这是个问题，没有答案，没有标准，只有适合不适合，我们完全可以根据自己的实际情况来做出开发规范。


关于三层架构
----------
以往，我们用Spring开发JavaWeb应用，基本上清一色的Controller->Service->Dao。那么用Tropic开发，还需要吗？其实，这里完全可以沿用之前的分层架构去写代码。

* Controller

.. code-block:: javascript

    var person_ctrl={
      service:function (req,resp){
          println($.toJson(resp));
          load("./servlet/demo/person_service.js");
          if(req.params){
              var id=req.params.get("id");
              if(id==null){
                  resp.code=500;
                  resp.msg.append("id不可以为null");
              }else{
                 var rst= person_service.queryOneById(id);
                 resp.body=rst;
              }
              return resp;
          }else{
              resp.code=500;
              resp.msg.append("请携带id参数查询");
              return resp;
          }
      }
    };

* Service

.. code-block:: javascript

    var person_service = {
        queryOneById: function (id) {
            load("./servlet/demo/person_dao.js");
            var sql="select * from person where id = "+id;
            var rst=person_dao.query(sql);
            return rst;
        }
    };

* DAO

.. code-block:: javascript

    var person_dao = {
        query: function (sql) {
            var conn=$.jdbc();
            var rn=$.sql();
            var obj=rn.query(conn,sql,$.asMapList);
            obj=$.format(obj);
            $.jdbc(conn);
            return obj;
        }
    };

上面我们展示了三层架构的方式来写代码，当然这些示例代码都很简陋。不过，我们需要注意load方法，这个方法是将我们三个代码源文件串起来的函数，由于我们
每个源文件都是声明式的对象变量，所以我们想使用就需要加载进来。另外，必须要从应用的根级目录来进行加载./就是指当前的框架home目录。



