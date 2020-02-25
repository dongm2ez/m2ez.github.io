---
layout: post
title: Laravel/Lumen项目接入Sentry异常监控
categories: development
tags: [laravel]
---

1. 首先安装sentry SDK包：

`$ composer require sentry/sentry-laravel:1.5.0`

如果使用的5.5以上的框架会自动发现包并加载（Lumen需手动注册），我们目前项目都是5.5以上所以无需单独配置。

Lumen注册方法是在bootstrap/app.php中添加：

```php

$app->register('Sentry\Laravel\ServiceProvider');

# Sentry must be registered before routes are included

# 必须在路由加载前注册

require __DIR__ . '/../app/Http/routes.php';

```

2. 然后将报告方法加入 App/Exceptions/Handler.php ​中：

```php

public function report(Exception $exception){
    if (app()->bound('sentry') && $this->shouldReport($exception)) {
        app('sentry')->captureException($exception);
    }

    parent::report($exception);

}

```

​3. 执行 `php artisan vendor:publish --provider="Sentry\Laravel\ServiceProvider"​` 将配置文件加入config文件夹中（Lumen需要手动复制或创建），将sentry中获得的DSN链接配置进文件：

`'dsn' => env('SENTRY_LARAVEL_DSN', null)，`

如果是null，sentry SDK将不触发上报。

需在`.env`中添加 `SENTRY_LARAVEL_DSN=https://<key>@sentry.io/<project>` 配置，具体地址是Sentry中项目分配。

4. 测试是否接入成功可以使用在路由文件中新建一个路由主动抛出异常

```php
Route::get('/debug-sentry', function () {
    throw new Exception('My first Sentry error!');

});
```
