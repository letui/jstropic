===========
服务实例
===========

boot()
----------

无参数，启动服务实例

shutdown()
----------

无参数，停止服务实例

afterBoot(fn)
-------------

* fn 服务实例启动后执行的函数

绑定一个函数，在服务实例启动后执行，使用方法：app.js代码中加入。示范如下：

.. code-block:: javascript

    load("nashorn:mozilla_compat.js");
    load("./config.js");
    load("./bin/server.js");
    //绑定启动后执行的函数
    $.afterBoot(function(){
        $.logger().info("服务实例启动后，将会输出这行内容");
    });
    $.boot();

afterShutdown(fn)
------------------

* fn 服务实例启动后执行的函数

绑定一个函数，在服务实例停止后执行，使用方法：app.js代码中加入。示范如下：

.. code-block:: javascript

    load("nashorn:mozilla_compat.js");
    load("./config.js");
    load("./bin/server.js");
    //绑定启动后执行的函数
    $.afterShutdown(function(){
        $.logger().info("服务实例停止后，将会输出这行内容");
    });
    $.boot();

status()
----------------

无参数，返回服务实例当前状态，以下三种

* RUNNING
* SHUTDOWN
* NOT_BOOT

bind(entry)
----------------

entry格式和config.js中endpoints配置数组中的项格式相同，需要三个字段

* path  要提供服务的http路径
* servlet 提供服务的小程序文件路径
* name 提供服务的对象名

该函数一般不显式使用，配合/bin/bind.js组件使用时由引擎自己使用

unbind(entry)
----------------

entry格式和config.js中endpoints配置数组中的项格式相同，需要一个字段

* path  要停止提供服务的http路径

该函数一般不显式使用，配合/bin/bind.js组件使用时由引擎自己使用

=================
数据访问
=================

jdbc(back)
----------

* back 提供该参数时，视为归还JDBC Connection实例，不提供则视为获取一个JDBC Connection。

使用该函数默认使用了连接池技术，繁忙时，该函数将会阻塞等待，所以使用完毕请及时归还。

db()
----------

无参数，不使用连接池创建一个JDBC Connection对象，一般不推荐使用。

sql()
----------

无参数，返回一个Apache DBUtils组件的QueryRunner实例，具体使用方式可参加官方文档。

asMapList
----------

非函数，默认提供一个Apache DBUtils组件的 MapListHandler实例，负责将查询出的数据转换为一个填充了HashMap的List，Map中的Key为数据库列名
值为数据库列的实际内容。

format(mapList)
---------------

* mapList 为QueryRunner的查询结果集

该函数提供默认的日期或者时间类型字段的自动格式化。

redis(back)
------------

* back 提供该参数时，视为归还Jedis实例，不提供则视为获取一个Jedis。

使用该函数默认使用了连接池技术，繁忙时，该函数将会阻塞等待，所以使用完毕请及时归还。

mongo(db,collection)
--------------------

* db 要访问的db名称 ，字符串格式
* collection  要访问的collection名称 ， 字符串格式。该collection视为隶属于 db 参数值下。

返回一个MongoDatabase 或者 MongoCollection 实例，只提供db参数时返回MongoDatabase实例，两者都提供时返回MongoCollection实例


asDoc(obj)
-------------------

* obj 常规Javascript 对象。

由于MongoDB要求查询条件务必是Document类型，所以该方法会将普通Javascript对象转换成为MongoDB所支持的Document类型，以便用于查询或其他数据
访问交互需要。

neo4j(session)
-------------------

* session 提供值true 则表示获取Session 不提供或者false则返回 Driver实例

代码示例辅助说明:

.. code-block:: java

    public class DriverLifecycleExample implements AutoCloseable
    {
    private final Driver driver;

    public DriverLifecycleExample( String uri, String user, String password )
    {
        //默认驱动 Driver实例
        driver = GraphDatabase.driver( uri, AuthTokens.basic( user, password ) );
    }

    @Override
    public void close() throws Exception
    {
        driver.close();
    }
    }

    private Driver createDriver( String virtualUri, String user, String password, ServerAddress... addresses )
    {
        Config config = Config.builder()
                .withResolver( address -> new HashSet<>( Arrays.asList( addresses ) ) )
                .build();

        return GraphDatabase.driver( virtualUri, AuthTokens.basic( user, password ), config );
    }

    private void addPerson( String name )
    {
        String username = "neo4j";
        String password = "some password";

        try ( Driver driver = createDriver( "neo4j://x.example.com", username, password, ServerAddress.of( "a.example.com", 7676 ),
                ServerAddress.of( "b.example.com", 8787 ), ServerAddress.of( "c.example.com", 9898 ) ) )
        {
            //使用会话 Session
            try ( Session session = driver.session( builder().withDefaultAccessMode( AccessMode.WRITE ).build() ) )
            {
                session.run( "CREATE (a:Person {name: $name})", parameters( "name", name ) );
            }
        }
    }

该函数获取neo4j的Driver实例或者Session实例。上文代码中演示了二者的使用方式。


================
HTTP-Client
================


get(url,headers,asJson)
-----------------------

* url 要请求的目标url地址，字符串格式
* headers 要携带的http headers，对象格式，key为header-name， value为header-value。
* asJson 响应结果是否要JSON对象化处理，默认为false，请根据实际情况自行设置。

使用get方式请求一个指定的http地址，并返回响应结果，该函数的超时配置在config.js中设置。



post(url,headers,data,asJson)
-----------------------------

* url 要请求的目标url地址，字符串格式
* headers 要携带的http headers，对象格式，key为header-name， value为header-value。
* data 要提交的http-body数据，支持对象格式
* asJson 响应结果是否要JSON对象化处理，默认为false，请根据实际情况自行设置。

使用post方式请求一个指定的http地址，并返回响应结果，该函数的超时配置在config.js中设置。


==============
其他工具函数
==============

empty(str)
-------------

* str 要检查的字符串

检查一个字符串是不是长度位0，不对null值做处理。

at(index,string|array)
-----------------------

* index 下标
* string|array 目标字符串或者数组

返回指定下标的内容，如果第二个参数是字符串则返回字符串指定index下标的内容，如果是数组则返回指定位置的数组项。

each(object|array,fn[i,n])
---------------------------

* object|array 一个Javascript对象或者数组
* fn[i,n] 一个带有两个参数的函数，i表示下标，n表示当前项。

该函数为for(var i in obj)的语法糖，像Jquery $.each致敬。

servlet(name,fn[req,resp])
--------------------------

* name 小程序的名字
* fn[req,resp] 小程序的服务提供函数，定义两个形参，一个是request一个是response。

语法糖，等价于定义一个包含了service函数的小程序对象。

redirect(resp,url)
-------------------

* resp http响应对象
* url 指定要重定向到的url地址，支持绝对路径和相对路径。

重定向函数，将请求指向另一个http-url地址

filter(name,fn[req,resp])
--------------------------

* name 小程序的名字
* fn[req,resp] 小程序的服务提供函数，定义两个形参，一个是request一个是response。

语法糖，等价于定义一个包含了service函数的小程序对象。


toJson(obj)
--------------

* obj 指定要JSON格式化的对象，支持数组和对象

将传入参数格式化为JSONString。

fromJson(jsonStr)
-----------------

* jsonStr 指定要反序列化的JSON字符串，支持数组和对象

将传入参数反序列化为对象或数组。

setTimeout(fn,time)
-------------------

* fn 无参函数
* time 延迟的时间，单位：毫秒

.. code-block:: javascript

    $.setTimeout(function(){
        println("Hello ,i'm in 'Timeout'");
    },2000);

指定一个时间，延迟执行一个函数，和浏览器中window.setTimeout等价。

setInterval(fn,time)
--------------------

* fn 无参函数
* time 间隔的时间，单位：毫秒

.. code-block:: javascript

    $.setInterval(function(){
        println("Hello ,i'm in 'Interval'");
    },2000);

指定一个时间，以该时间为固定间隔执行一个函数，和浏览器中window.setInterval等价。

=========
日志
=========

logger(name)
---------------

* name 日志对象的名字，不传入时默认为:default

获取一个日志对象，用于记录日志


==========
秘钥证书
==========

genkey(cfg)
------------

生成秘钥证书，建议配合组件/bin/keytool.js使用。如果手动调用使用，请按照参数格式要求。cfg为对象格式，包含以下属性字段

* alias 秘钥别名
* keypasswd 秘钥密码
* alg 加密算法
* keysize 长度
* expire 有效期
* keystorename 秘钥库文件名
* keystorepass 秘钥库密码
* cnname CN 名字与姓氏
* ouname OU 组织单位名称
* oname O 组织名称
* lname L 城市或区域名称
* stname ST 州或省份名称
* cname C 单位的两字母国家代码


==========
代码生成
==========

gencrud(tables,asCamel)
-----------------------

* tables 表名数组，数组格式
* asCamel 列名是否Camel化，当列名中有 _ 时，此参数传true将自动去掉 _ 并将其后第一个字母大写。

生成crud模板化代码，示例:

.. code-block:: javascript

    load("nashorn:mozilla_compat.js");
    load("./config.js");
    load("./bin/server.js");
    load("./bin/crud.js");
    $.gencrud(["person","city"],true);

执行完毕后，即可在/servlet目录下找到对应代码，并会产生一个endpoints.js的文件，其中包含了所有的默认配置。
