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




