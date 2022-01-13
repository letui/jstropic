============
Introduce
============

Jstropic是一套支持跑在JVM上的Web开发框架，允许开发人员以JS的语法来开发后台程序。这一点有点类似
NodeJs，但是本质上又不同于NodeJs。

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
    }


这么一段代码，启动我们的Tropic后，访问 http://127.0.0.1:9999/ 。即可在浏览器里看到：

.. code-block:: javascript

{"code":200,"msg":"Version:1.0","body":"Hello!! Welcome to Tropic engine."}

框架本身并不预期解决所有用Java语言来开发程序的能力，但是提供了一种无需编译即可运行的能力，这一点不同OSGI技术，更加不同于JSP技术。准确的说运行JS
的能力本身就是JVM自带的特性之一，本框架只是稍微往前走了那么一小步，使其更容易开发一些小微服务，动态灵活热部署，同时呢又可以完全拥抱强大的Java开源
生态。整个框架的核心代码不超过300行，做到超小体积。并且，辅助核心工具库的API设计也向Jquery致敬，尽可能语义化，秉持少就是多的理念。

在这里，也希望有更多志同道合的朋友一起加入，将其打造得更优秀。
