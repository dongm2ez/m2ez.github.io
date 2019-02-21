---
layout: post
title: Laravel 5.5 以下版本中自定义日志行为
---

在 Laravel 5.6 版本中日志行为可以很容易的进行自定义，而在5.5以下版本中日志行为自定义自由度并不是很高，但是项目有需求不能因为这个就强行将项目升级为5.6吧，况且作为一个稳定的项目升级框架大版本有可能会有很多坑，基于这些原因我尝试了对 Laravel 5.5 的日志进行改造以适应我的需求。

Laravel 的日志行为大部分是在 `Illuminate\Log\LogServiceProvider` 中，我们可以看一下其中的代码片段：

```PHP
/**
 * Configure the Monolog handlers for the application.
 *
 * @param  \Illuminate\Log\Writer  $log
 * @return void
 */
protected function configureDailyHandler(Writer $log)
{
    $log->useDailyFiles(
        $this->app->storagePath().'/logs/laravel.log', $this->maxFiles(),
        $this->logLevel()
    );
}
```

这是我最常在项目中使用的日志存储方式，可以看到日志的存储路径几近与写死的状态，无法通过外部参数轻易的更改。

最开始我想的是重写这个 `Provider` 然后将其注册到 `app.php` 的 `providers` 数组中，但是这种行为并不可行，因为通过查看源码，`LogServiceProvider` 是在框架启动时就注册。

在 中有这样一个方法控制了这个注册行为：

```php
protected function registerBaseServiceProviders()
{
    $this->register(new EventServiceProvider($this));

    $this->register(new LogServiceProvider($this));

    $this->register(new RoutingServiceProvider($this));
}
```

既然我们知道了它们是如何生效的，那么我们将这两个类继承并修改其中我们需要改变的行为进行改造，我的改造方式如下。在`app\Providers`中新建`LogServiceProvider`类继承`Illuminate\Log\LogServiceProvider`，代码如下：

```PHP
<?php


namespace App\Providers;

use Illuminate\Log\LogServiceProvider as BaseLogServiceProvider;
use Illuminate\Log\Writer;

class LogServiceProvider extends BaseLogServiceProvider
{
    /**
     * Configure the Monolog handlers for the application.
     *
     * @param  \Illuminate\Log\Writer  $log
     * @return void
     */
    protected function configureDailyHandler(Writer $log)
    {
        $path = config('app.log_path');
        $log->useDailyFiles(
            $path, $this->maxFiles(),
            $this->logLevel()
        );
    }
}
```

在`config/app.php`目录中添加配置：

```
'log_path' => env('APP_LOG_PATH', storage_path('/logs/laravel.log')),
```

`app`目录中新建`Foundation`目录，新建`Application`类继承`Illuminate\Foundation\Application`类，重写`registerBaseServiceProviders`方法。

```PHP
<?php
/**
 * Created by PhpStorm.
 * User: dongyuxiang
 * Date: 2018/7/31
 * Time: 16:53
 */

namespace App\Foundation;

use App\Providers\LogServiceProvider;
use Illuminate\Events\EventServiceProvider;
use Illuminate\Routing\RoutingServiceProvider;
use Illuminate\Foundation\Application as BaseApplication;


class Application extends BaseApplication
{

    /**
     * Register all of the base service providers.
     *
     * @return void
     */
    protected function registerBaseServiceProviders()
    {
        $this->register(new EventServiceProvider($this));

        $this->register(new LogServiceProvider($this));

        $this->register(new RoutingServiceProvider($this));
    }

}
```

说是重写其实只是将use类换从了我们自己创建的`LogServiceProvider`。

然后在`bootstrap\app.php`中将变量`$app`的`new`对象换成我们继承重写后的。

```
$app = new App\Foundation\Application(
    realpath(__DIR__.'/../')
);
```

这样我就成功的将日志路径可以随便定义了，而且来说有了这次经验我对于框架不符合我需求的地方可以做更进一步的优化以符合我的要求，而且我没有更改框架底层的代码，当框架有bug修复的时候我也可以放心的进行框架更新。

