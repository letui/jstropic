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


这么一段代码，启动我们的Tropic后，访问 http://127.0.0.1:9999/ 。即可看到 {"code":200,"msg":"Version:1.0","body":"Hello!! Welcome to Tropic engine."}
