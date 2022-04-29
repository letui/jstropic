=============
配置文件介绍
=============

任何一个框架终究免不了一些配置，Tropic同样如此。由于Tropic本身由Javascript脚本语言所开发，秉持着配置即代码，代码即配置的理念而设计。所以，
整个Tropic的配置文件实际上就是个全局Javascript变量，只不过是个比较大的Json-Object而已。整个配置中会有服务器监听端口，并行线程数，数据库
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

server
------

目前针对server的配置项只有四个，分别是port,threads,use_dynamic_bind,auth_bind_token。前面两个相信很多人很容易就理解了，所以就展开介绍
下后面两个，这两个配置具体来说是Tropic的一个高级功能，当使用了动态绑定能力时，Tropic-Server允许你向一个特殊的路径发起控制请求。这个请求是用来
实时启用或停用某一个服务端小程序的。这个功能是个很强大的功能，但同时也很危险，所以呢就需要引入一个安全令牌来保障。这就是另外一个auth_bind_token
的作用。auth_bind_token所配置的令牌内容将作为动态绑定控制请求的安全校验，也就是说发起动态绑定控制请求时，必须携带令牌，这个令牌的内容必须和配置
的令牌保持一致才可以得以成功处理。

endpoints
---------
这个配置项不难理解，这就是所有服务端小程序要注册的地方，大致上类似我们用Spring时的所有Controller的注册。这个配置项是个数组格式，其中每一项都要求
是标准的Json-Object格式，拥有三个属性，分别是path,servlet,name。

* path 服务端小程序要服务的路径
* servlet 服务端小程序源代码在服务器上存放的路径，一把建议使用相对路径./servlet/xxx.js，放在Tropic框架的servlet目录下，这里并不限制子级目录
的使用
* name 我们的任何一个服务端小程序都是个Javascript变量，Object类型，务必持有一个service(req,resp)格式的函数，返回值是resp。这个name要求就是
所定义出的变量的名字，这一点就有点类似Spring单例的概念，等同于beanName的意义。

filters
--------
过滤器和servlet的配置没有什么不同，在写法上也没有什么不同。这里启用单独配置，完全是为了分开独立管理，便于维护。



