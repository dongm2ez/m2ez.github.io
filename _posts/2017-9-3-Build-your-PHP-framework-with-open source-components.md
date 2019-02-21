---
layout: post
title: 用开源组件构建属于你的 PHP 框架
---

## 为什么要构建自己的 PHP 框架？

现在的 PHP 框架很多，当然不止 PHP ，即使是其他编程语言也有很多框架，这篇文章讲 PHP 框架构建是因为我对 PHP 的生态最为熟悉，但这个方法同样也适用于其他编程语言框架的构建。

框架是为了提升我们的应用开发效率，市面上有很多开源免费的框架给我们使用，我们尽可以拿来用，为什么还要自己构建一个自己的框架呢？原因就在于市面上的开源框架，是给大部分人用的，给通用项目用的，作为框架的开发者是不知道自己的框架使用者的具体业务的，所以开源框架一定是满足大部分人的需求，而且力求能够为开发者提供所有可能用到的功能。

但是对于一个商业项目或者是一个你自己要做的项目也许只能用到框架的很少一部分功能，或者是框架给你提供的东西并不是最符合你自己的需求的，你使用了框架的一部分功能，另一部分根本没用，这样使用框架首先是性能上的损失，一些你根本用不到的功能却要降低你应用的性能显然不合适的。再就是也许框架提供的功能不是你想要的，或者这个功能这个框架提供的并不是符合你需求的，又或者要使用这部分功能必须按照框架开发者制定的规范来使用，这个规范并符合你的开发哲学。

## 从哪开始？

各种现代编程语言都有自己的包管理工具，PHP 就是 composer ，利用它我们就可以构建属于自己的框架了，并能很好的组织我们的框架。

## 怎么开始？

我们该怎么开始构建我们自己的框架呢？从零开始吗？这个问题没有标准答案，如果你要做的项目要求很严格，从底层开始就要保证项目架构的最稳定可控，那么建议你从零开始。如果要求不是非常严格那么我们就从那些开发一个应用最基本需要的功能开始，这样的功能谁提供呢？PHP 有很多微框架，这些框架提供开发一个应用最基础的功能，我们可以从这里开始。

首先我们通过 `composer init` 初始化一个项目：

```
{
    "name": "dongm2ez/m2ez-framework",
    "description": "a dongm2ez's framework",
    "keywords": ["framework", "m2ez-framework"],
    "license": "MIT",
    "authors": [
        {
            "name": "dongm2ez",
            "email": "dongm2ez@163.com"
        }
    ],
    "require": {}
}
```

这就是我得到的一个 `composer.json` 的描述文件，现在我们就从这里开始。我的目标是构建一个最符合我开发习惯的框架，让我的开发效率最高。

我选择 Slim 框架作为我的框架基础框架，这是一个微框架，我喜欢它，它足够简单，提供了 web 开发 和 API 开发最基础的功能，而且还有一个原因开发这个框架的作者写了一本名为《Modern PHP》的书，这本书颠覆了我对 PHP 这个语言的认知，开始喜欢并乐于使用它。

在引入这个框架之前我还要对 PHP 版本做个限制，从我使用 PHP 从 5.2 开始到现在，PHP 已经发展到 PHP 7.2 了，但是我不想再去使用低版本的 PHP，一个是 PHP 低版本马上将失去官方的支付，另一个是一些 PHP 的新特性我不能使用，而且低版本的性能也是不好的。所以我要将我的框架限制在 PHP 7.0 以上，同时我希望我的框架对中文有更好的支持。

那么我将更新我的框架 `composer.json` 文件:

```
{
    "name": "dongm2ez/m2ez-framework",
    "description": "a dongm2ez's framework",
    "keywords": ["framework", "m2ez-framework"],
    "license": "MIT",
    "authors": [
        {
            "name": "dongm2ez",
            "email": "dongm2ez@163.com"
        }
    ],
    "require": {
        "php": ">=7.0.0",
        "ext-mbstring": "*",
        "slim/slim": "^3.0"
    }
}

```

这样我就获得了一个最基础的我的框架版本，但是我还没完成，因为我们没有定义我的框架目录结构。我觉得 laravel 框架的目录划分是挺让我喜欢的，但我又不完全喜欢 laravel 的目录结构，我需要对它进行改造。

```shell
├── app
│   ├── Helpers.php
│   ├── Http
│   │   └── Controllers
│   └── Models
├── composer.json
└── tests
```

我这样设置我的目录结构，并更新我的 `composer.json`:

```JSON
{
    "name": "dongm2ez/m2ez-framework",
    "description": "a dongm2ez's framework",
    "keywords": ["framework", "m2ez-framework"],
    "license": "MIT",
    "type": "project",
    "authors": [
        {
            "name": "dongm2ez",
            "email": "dongm2ez@163.com"
        }
    ],
    "require": {
        "php": ">=7.0.0",
        "slim/slim": "^3.0"
    },
    "require-dev": {
        "phpunit/phpunit": "~6.0"
    },
    "autoload": {
        "classmap": [
            "app/Models"
        ],
        "psr-4": {
            "App\\": "app/"
        },
        "files": [
            "app/Helpers.php"
        ]
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    }
}

```

这样的框架可以访问吗，显然是不行的，我们还要加一些东西让我们的框架真正可以跑起来，然后在来迭代它。

```shell
├── app
│   ├── Helpers.php
│   ├── Http
│   │   └── Controllers
│   └── Models
├── bootstrap
│   ├── app.php
│   └── autoload.php
├── composer.json
├── config
├── public
│   └── index.php
├── routers
└── tests

```

我们将目录结构改造成这样，并编写一些启动框架的代码到相应的文件

```PHP

// public/index.php
<?php

require __DIR__.'/../bootstrap/autoload.php';

$app = require_once __DIR__.'/../bootstrap/app.php';

$app->run();

// bootstrap/app.php

<?php

$app = new \Slim\App;

return $app;

// bootstrap/autoload.php

<?php

define('M2EZ_START', microtime(true));

require __DIR__.'/../vendor/autoload.php';



```

然后运行 `composer install` 安装框架的依赖包，安装完成后我们的目录中就会多出一个 `vendor` 的目录和 `composer.lock` 的文件，此时运行 `php -S 0.0.0.0:8080 -t public public/index.php` 利用 PHP 自带的 web 服务器进行测试，为了这个命令更简单使用，我们可以将这个命令加到 `composer.json` 的 `script` 中。

此时访问 `127.0.0.1:8080` 或 `localhost:8080` 就可以看到如下的页面：

![](http://img.m2ez.com/15043663356684.jpg)

这说明框架正确启动了，那么我们怎么确定框架工作正常呢，这里有个简单方法：

```PHP
<?php

use Slim\Http\Request;
use Slim\Http\Response;

require __DIR__.'/../bootstrap/autoload.php';

$app = require_once __DIR__.'/../bootstrap/app.php';

$app->get('/hello/{name}', function (Request $request, Response $response) {
    $name = $request->getAttribute('name');
    $response->getBody()->write("Hello, $name");

    return $response;
});

$app->run();
```

对 `public/index.php` 的代码进行修改，此时访问 `http://localhost:8080/hello/dongm2ez`，那么我们就会看到：

![](http://img.m2ez.com/15043666561898.jpg)

测试是成功了，但是我们不能把路由和逻辑都写到 `index.php` 文件里，因此我们需要代码更好的组织。要让我们的目录规划发挥正在的作用。

为了单独管理路由，我将路由单独写在 `routers` 文件夹中，在文件夹中我们新建两个 PHP 脚本文件，然后在 `public/index.php` 中加入两行代码：

```PHP
<?php
require __DIR__ . '/../bootstrap/autoload.php';

$app = require_once __DIR__ . '/../bootstrap/app.php';

require __DIR__ . '/../routers/web.php';
require __DIR__ . '/../routers/api.php';

$app->run();
```

变成这样，这样我就可以单独管理 API 和 WEB 项目的路由了，如果有其他路由就也可以 require 更多路由。

```PHP
<?php

$app->get('/', '\App\Http\Controllers\WelcomeController:index');

```

而我们的控制器长什么样呢，是这个样子的：

```PHP
<?php

namespace App\Http\Controllers;


use Slim\Http\Request;
use Slim\Http\Response;

class WelcomeController extends Controller
{

    public function index(Request $request, Response $response)
    {
        $response->getBody()->write("Hello, world");

        return $response;

    }

}
```

我们知道在现代化的框架中，容器会让我们很方便，我们的基础框架 Slim 提供一个容器的实现，当然你也可以使用其他的第三方的，那么这显然是我们想要的结果，不是只能使用框架提供的，我们可以随时换掉框架的功能，换成我们想要的同样功能组件。

使用也很简单，在 `app.php` 文件中初始化 Slim 框架时将容器实例传递给它就可以了。

```PHP
<?php

$container = new \Slim\Container;
$app = new \Slim\App($container);



return $app;
```

还记得上面那个控制器继承的基类控制器吗，那也是我自己写的，里面可以做一些所有控制器都有可能用的的操作封装。比如我为了更方便的使用容器，我在基类里初始化了一个容器实例。

```PHP
<?php


namespace App\Http\Controllers;

use Interop\Container\ContainerInterface;

abstract class Controller
{
    protected $ci;

    public function __construct(ContainerInterface $ci)
    {
        $this->ci = $ci;
    }

}
```

现在我已经有了 路由功能，有了控制器功能，还有请求响应的操作，那么作为一个完整的框架那么必须有访问数据库的方法。

我很喜欢 Laravel 提供的数据库 ORM 组件，那么我就决定使用它了，执行 `composer require illuminate/database "~5.5"`，我选择了最新的 Laravel 长期支持版 ORM 。

我们需要此时在 `config` 文件夹中新添加一个文件 `databases.php` 文件：

```PHP
<?php
return [
    'settings' => [
        'db' => [
            'driver' => 'mysql',
            'host' => 'localhost',
            'database' => 'database',
            'username' => 'user',
            'password' => 'password',
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
        ]
    ],
];
```

然后修改 `public/index.php`：

```PHP
<?php
require __DIR__ . '/../bootstrap/autoload.php';

$config = require __DIR__ . '/../config/databases.php';

$app = require_once __DIR__ . '/../bootstrap/app.php';

require __DIR__ . '/../routers/web.php';
require __DIR__ . '/../routers/api.php';

$app->run();
```

修改 `bootstrap/app.php`：

```PHP
<?php

$container = new \Slim\Container($config);
$app = new \Slim\App($container);

// Service factory for the ORM
$container['db'] = function ($container) {
    $capsule = new \Illuminate\Database\Capsule\Manager;
    $capsule->addConnection($container['settings']['db']);

    $capsule->setAsGlobal();
    $capsule->bootEloquent();

    return $capsule;
};

return $app;
```

然后我们就可以在控制器中使用 ORM 功能了。

```PHP
<?php

namespace App\Http\Controllers;


use App\Models\User;
use Illuminate\Database\DatabaseManager;
use Illuminate\Support\Facades\DB;
use Interop\Container\ContainerInterface;
use Slim\Http\Request;
use Slim\Http\Response;

class WelcomeController extends Controller
{

    public function index(Request $request, Response $response)
    {
        /** @var DatabaseManager $db */
        $db = $this->ci->get('db');
        $user = $db->table("user")->first();
        var_dump($user);
        $response->getBody()->write("Hello, world");

        return $response;

    }

}
```

那么此时这个框架已经是一个可以开发 API 功能的框架了，如果要开发 Web 站我可能还需要加入渲染模板组件，无论是 `twig`、`Smarty`、`Haml` 还是 `Blade`，全都看你的喜好了。当然我觉得我做到这里就可以了，够我用了，因为对于前端我更喜欢用 `React` 或者是 `Vue` 去实现它。

需要更简便的操作 `session`、`cookie` 那么我们也可以添加相应的组件，各种已经有的框架了都提供这样的组件，看看你更喜欢哪一个了，现在你的框架你做主，你想添加什么就可以添加什么组件，经过这样的定制的框架一定是最符合你开发需求的。

我这里只是对已有的组件进行了配置组装，一旦哪天你发现所有的开源组件都满足不了你的需求的时候，因为你对你的框架了解，你可以自己造个轮子给自己的框架用，如果你写的好那么你也会创造出一个极好用的框架，现在最流行的 PHP 框架，你可以看看它的 `composer.json` 文件，它就是在前人的基础上进行开发维护的，已经有的功能他拿来直接用，觉得别人做的不完善的地方自己造一个轮子给大家用。

而且我这里也没有用到太多的设计模式，你还可以改造你的框架，利用PHP的魔术方法，反射，SPL 等等让你的框架更好，更容易扩展，更容易配置。

## 总结

框架很神秘吗？看过这篇文章我相信你不会这样觉得了。

造一个框架很难吗，是的很难，因为从 0 到 1 任何事都难，但是我们现在还需要从 0 到 1 吗，基本不需要了！站在巨人身上做事更容易，而且要记住，任何事只有行动起来你就会发现尝试比踌躇不前更好，从小处开始，做小事，有一天这个小事就变成了大事。不积跬步无以至千里。

Laravel 为什么流行，因为作者本是一名 .net 开发者，在使用 CI 框架时萌生了想法要做一个更简洁、灵活的框架，他的思想真的很先进吗，不一定的，其他开发语言早就有了 Laravel 中的功能，它只是在 PHP 中实现了它们。

以上例子其实告诉我们，不要给自己贴标签，人生不设限，你不是 PHP 程序员，你就是开发者，任何开发相关的东西我们都该去了解和掌握，标签只能别人给你贴，不要自己给自己贴。

最后，这份代码我已经上传到 GitHub ，如果你有兴趣可以 fork 并完善它，开发配置一个自己的框架，以你的需求为出发，选择你最喜欢的技术和组件。

地址：[](https://github.com/dongm2ez/m2ez-framework)



