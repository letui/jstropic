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

无参数，返回服务实例当前状态

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

mongo(db,collection)
--------------------

asDoc(obj)
-------------------

neo4j(session)
-------------------


================
HTTP-Client
================

get(url,headers,asJson)
-----------------------

post(url,headers,data,asJson)
-----------------------------


==============
其他工具函数
==============

empyt(str)
-------------

at(index,string|array)
-------------------

each(object|array,fn[i,n])
---------------------------

servlet(name,fn[req,resp])
--------------------------

redirect(resp,url)
-------------------

filter(name,fn[req,resp])
--------------------------

toJson(obj)
--------------

fromJson(jsonStr)
-----------------

setTimeout(fn,time)
-------------------

setInterval(fn,time)
--------------------

=========
日志
=========

logger(name)
---------------

==========
秘钥证书
==========

genkey(cfg)
------------


==========
代码生成
==========

gencrud(tables,asCamel)
-----------------------

