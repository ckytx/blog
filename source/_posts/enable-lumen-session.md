---
title: Lumen中启用session
tags:
  - lumen
categories:
  - laravel
  - lumen
toc: false
date: 2017-06-02 08:55:18
---

Lumen默认没有启用了session的支持，但是我们可以自己添加session功能，这一切都得益于强大的Laravel及Composer。
<!--more-->
> Lumen5.2 的Release Notes中官方明确的指出Lumen专注于构建无状态API，JSON API 服务，移除了session和view的支持，但其实view还是存在的，session确实真的被移除了

因为项目需要所以我决定找回session，虽然官方建议需要session功能时可以使用强大的Laravel框架，但是对我小项目确实有点大材小用了。因为喜欢Laravel的优雅，所以我在小项目中都会把Lumen做为首选。

下面就来一步步找回“丢失”的session吧！

### 注册SessionServiceProvider
打开bootstrap/app.php，在相应位置添加注册SessionServiceProvider，代码如下
```
// bootstrap/app.php
// ...
/*
|--------------------------------------------------------------------------| 
Register Service Providers
|--------------------------------------------------------------------------|
...
|*/
// ...
// 注册 SessionServiceProvider
$app->register(Illuminate\Session\SessionServiceProvider::class);
// ...
```

### 添加session相关配置所需配置
同样在bootstrap/app.php中添加配置代码
```
// bootstrap/app.php
// ...
// 注册 SessionServiceProvider
$app->register(Illuminate\Session\SessionServiceProvider::class);

// 载入session相关配置
$app->configure('session');
```
configure函数会从文件中加载配置，并绑定到container中，接下来只需要在项目根目录创建config文件夹，并新建session.php配置文件
```
<?php
// config/session.php
// 这里直接从laravel中copy过来了，并且使用了file驱动，当然你也可以使用其他驱动，详细配置请查阅laravel官方文档
return [
    'driver' => env('SESSION_DRIVER', 'file'),//默认使用file驱动，你可以在.env中配置
    'lifetime' => 120,//缓存失效时间
    'expire_on_close' => false,
    'encrypt' => false,
    'files' => storage_path('framework/session'),//file缓存保存路径
    'connection' => null,
    'table' => 'sessions',
    'lottery' => [2, 100],
    'cookie' => 'laravel_session',
    'path' => '/',
    'domain' => null,
    'secure' => false,
];
```
然后在storage/framework/下建立session文件夹用来存储session缓存，别忘了修改权限
### 注册StartSession中间件
打开bootstrap/app.php，在Register Middleware位置修改
```
// bootstrap/app.php
// ...
|--------------------------------------------------------------------------
| Register Middleware
|--------------------------------------------------------------------------
// ...
$app->middleware([
    Illuminate\Session\Middleware\StartSession::class
]);
```

### 添加Session别名
因为框架中使用了session别名，所以需要添加alias，不然会报错
```
// bootstrap/app.php
// ...
// 注册 SessionServiceProvider
$app->register(Illuminate\Session\SessionServiceProvider::class);

// 载入session相关配置
$app->configure('session');

// 设置session别名
$app->alias('session', 'Illuminate\Session\SessionManager');
```

### 使用session
至此，你就可以在代码中使用session了，使用 app('session') 即可获取session实例，示例：
```
// 写入一条数据至 session 中...
app('session')->put('key','value');

// 获取session中键值未key的数据
app('session')->get('key');
```
有关session的详细用法可查阅Laravel官方文档

ps：session辅助函数
```
// session辅助函数
if (! function_exists('session')) {
    function session($key = null, $default = null)
    {
        $session = app('session');

        if (is_null($key)) {
            return $session;
        }
        if (is_array($key)) {
            return $session->put($key);
        }

        return $session->get($key, $default);
    }
}
```
辅助函数可以让你更方便的使用session

End.

本文最初发布于我的简书：[http://www.jianshu.com/p/dc33f8ab0618](http://www.jianshu.com/p/dc33f8ab0618)