=============
Configuration
=============

任何一个框架终究免不了一些配置，Tropic同样如此。由于Tropic本身由Javascript脚本语言所开发，秉持着配置即代码，代码即配置的理念而设计。所以，
整个Tropic的配置文件实际上就是个全局Javascript变量，只不过是个比较大的Json-Object而已。整个配置中会把服务器监听端口，并行线程数，数据库
驱动配置信息，Redis连接信息，Http工具默认超时时间等等。之前，提到过由于其本身是个全局Javascript变量，这就意味着任何一个服务端小程序的代码里
都可以直接访问使用，其名字为:config。在此要说明的是，Tropic并不限制该配置变量的其他额外配置信息加入。也就是说，任何一个开发者都可以根据自己的
实际情况向config中添加自己需要的配置信息。在使用时，只需要根据Javscript的对象导航语法就可以完成设置或者读取。

配置示例
--------

.. code-block:: javascript

    var config =
    {
        db: {
            url: "jdbc:mysql://127.0.0.1:3306/test",
            user: "root",
            pass: "123qwe123",
            driver: "com.mysql.cj.jdbc.Drvier",
            poolSize:10,
        },
        redis:{
          host:"192.168.10.173",
          port:6379,
          maxIdle:10,
          maxTotal:20,
          maxTimeout:2000
        },
        http:{
            connect_timeout:3000,
            read_timeout:3000
        },
        server: {
            port: 9999,
            threads: 200,
            use_dynamic_bind:true,
            auth_bind_token:"Tropic",
        },
        endpoints: [
            {path: "/", servlet: "./bin/index.js", name: "index"},
            {path: "/@db", servlet: "./bin/dbutils.js", name: "dbutils"},
            {path: "/db/person", servlet: "./servlet/db_person.js", name: "db_person"},
            {path: "/person", servlet: "./servlet/person.js", name: "person"},
            {path: "/pxy", servlet: "./servlet/httpproxy.js", name: "httpproxy"}
        ]
    };

目前我们看到的这一份完整的配置信息，所有的一级属性字段都是不可以私自改变命名的。因为，这里所有展示的配置项都是Tropic框架本身所使用的。

* db
  配置数据库连接相关
* redis
  配置redis相关
* http
  配置Http工具超时
* server
  配置Tropic启动参数，详细属性后面会展开介绍
* endpoints
  配置所有需要注册加载的服务端小程序



