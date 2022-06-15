<p align="center"><img width="260px" src="https://chaz6chez.cn/images/workbunny-logo.png" alt="workbunny"></p>

**<p align="center">workbunny/webman-nacos</p>**

**<p align="center">🐇  PHP implementation of Nacos OpenAPI for webman plugin. 🐇</p>**

# A PHP implementation of Nacos OpenAPI for webman plugin

[![Latest Stable Version](http://poser.pugx.org/workbunny/webman-nacos/v)](https://packagist.org/packages/workbunny/webman-nacos) 
[![Total Downloads](http://poser.pugx.org/workbunny/webman-nacos/downloads)](https://packagist.org/packages/workbunny/webman-nacos) 
[![Latest Unstable Version](http://poser.pugx.org/workbunny/webman-nacos/v/unstable)](https://packagist.org/packages/workbunny/webman-nacos) 
[![License](http://poser.pugx.org/workbunny/webman-nacos/license)](https://packagist.org/packages/workbunny/webman-nacos) 
[![PHP Version Require](http://poser.pugx.org/workbunny/webman-nacos/require/php)](https://packagist.org/packages/workbunny/webman-nacos)

---
## 简介
- **Nacos 致力于帮助您发现、配置和管理微服务；是微服务/SOA架构体系中服务治理环节的重要成员服务；**

- **Webman-naocs是基于PHP开发的Webman插件生态下的Nacos客户端；**

- **本项目来源于 [Tinywan/nacos](https://www.workerman.net/plugin/25)，对 Tinywan 表示感谢！区别于 [Tinywan/nacos](https://www.workerman.net/plugin/25)，[workbunny/webman-nacos](https://github.com/workbunny/webman-nacos)在配置监听和实例注册上有不同的实现方式，其他的使用方法与之无异；**

- **Webman-nacos使用的主要组件：**
    - workerman/http-client
    - guzzlehttp/guzzle

## 安装
~~~
composer require workbunny/webman-nacos
~~~

## 使用

### 1. Nacos文档地址

- **[Nacos Open-API文档](https://nacos.io/zh-cn/docs/open-api.html)**

### 2. 服务的使用

#### 配置相关：

- 监听配置 

webman-nacos组件默认会启动一个名为 **config-listener** 的进程，用于监听在配置文件
**plugin/workbunny/webman-nacos/app.php** 中 **config_listeners**
下的配置内容。

如果想自行掌控调用，可以使用如下服务：
```php
$client = new Workbunny\WebmanNacos\Client();

# 异步非阻塞监听
$response = $client->config->listenerAsyncUseEventLoop();

# 异步阻塞监听
$response = $client->config->listenerAsync();

# 同步阻塞监听
$response = $client->config->listener();
```
**listenerAsyncUseEventLoop()** 在webman中是异步非阻塞的，不会阻塞当前进程；

**listenerAsync()** 在webman中是异步阻塞的，返回的是promise，多条请求可以同时触发，
但需要调用 **wait()** 执行，请求会阻塞在 **wait()** 直到执行完毕；详见 **ConfigListernerProcess.php** ；

- 获取配置

```php
$client = new Workbunny\WebmanNacos\Client();
$response = $client->config->get('database', 'DEFAULT_GROUP');
if (false === $response) {
    var_dump($nacos->config->getMessage());
}
```

- 提交配置

```php
$client = new Workbunny\WebmanNacos\Client();
$response = $client->config->publish('database', 'DEFAULT_GROUP', file_get_contents('.env'));
if (false === $response) {
    var_dump($nacos->config->getMessage());
}
```

- 移除配置

```php
$client = new Workbunny\WebmanNacos\Client();
$response = $client->config->delete('database', 'DEFAULT_GROUP');;
if (false === $response) {
    var_dump($nacos->config->getMessage());
}
```

#### 服务相关：

- 实例注册

webman-nacos组件默认会启动一个名为 **instance-registrar** 的进程，用于注册在配置文件
**plugin/workbunny/webman-nacos/app.php** 中 **instance-registrar**
下的配置内容。

如需动态注册实例，请使用：

```php
$client = new Workbunny\WebmanNacos\Client();
$response = $client->instance->register('127.0.0.1', 8848, '猜猜我是谁', [
    'groupName' => 'DEFAULT_GROUP',
]);
if (false === $response) {
    var_dump($nacos->config->getMessage());
}
```

- 移除实例

```php
$client = new Workbunny\WebmanNacos\Client();
$response = $client->instance->delete('猜猜我是谁', 'DEFAULT_GROUP', '127.0.0.1', 8848, []);
if (false === $response) {
    var_dump($nacos->config->getMessage());
}
```

- 实例列表

```php
$client = new Workbunny\WebmanNacos\Client();
$response = $client->instance->list('猜猜我是谁', []);
if (false === $response) {
    var_dump($nacos->config->getMessage());
}
```

**注：实例与服务的区别请参看Nacos文档；**

#### 其他：

- **具体使用参数都在源码内已标注，使用方法很简单，参考Nacos官方文档即可；**

- **后缀为Async的方法是Guzzle异步请求，在workerman的on回调中依旧是阻塞，只是多个请求可以并发执行；**

- **后缀为AsyncUseEventLoop的方法是workerman/http-client异步请求，在workerman的on回调中是非阻塞的；**

```php
$client = new Workbunny\WebmanNacos\Client();

# 配置相关接口
$client->config;

# 鉴权相关接口
$client->auth;

# 实例相关接口
$client->instance;

# 系统相关接口
$client->operator;

# 服务相关接口
$client->service;
```


## 说明

- 目前这套代码在我司生产环境运行，我会做及时的维护，**欢迎 issue 和 PR**；

- 对于不知道Nacos有什么用的/在什么时候用，可以参考这篇文章 [Nacos在我司的应用及SOA初尝](https://www.workerman.net/a/1339);

- nacos的配置监听项采用了服务端长轮询，有点类似于stream_select，当配置没有改变的时候，会阻塞至请求结束；但当配置有变化时候，会立即返回其配置dataId；这里我的做法是开启一个Timer对配置进行监听，定时器间隔与长轮询最大阻塞时间一致:

    1. ConfigListenerProcess使用Guzzle的异步请求对配置监听器进行请求处理，
       onWorkerStart中的Guzzle客户端会阻塞请求，workerman status中会显示BUSY状态；

    2. AsyncConfigListenerProcess使用wokerman/http-client异步请求对配置监听
       器进行请求，workerman/http-client使用了workerman的event-loop进行I/O处理，
       不会阻塞当前进程，推荐使用；

- 所有的配置同步后会触发 **workerman reload** 对所有进程进行重载，保证了config的刷新，包括已经在内存中的各种单例，如 数据库连接、Redis连接等，保证即时将配置传达至需要的业务点；

- 使用配置方式不必改变，使用webman的config()即可，降低封装组件的心智负担;

