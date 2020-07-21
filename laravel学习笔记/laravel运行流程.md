### 1. 流程

1. require 自动加载

2. require bootstrap/app.php

3. 启动application

   1. 设置基础路径信息

   2. 注册基础绑定：把application实例绑定到$app->instances数组中的app和容器字段

   3. 注册基础服务provider

      注册事件容器，注册日志容器，注册路由容器

   4. 注册容器核心别名

   5. 设置项目根目录

4. make kernel并且启动起来

   