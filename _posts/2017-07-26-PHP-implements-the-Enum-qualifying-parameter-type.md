---
layout: post
title: PHP 用 Enum 限定参数类型
categories: development
tags: [PHP]
---

### 痛点

PHP 是一门若类型语言，这是大家都知道的，弱类型让我们在编写代码时很舒服，但是维护它却变得不那么舒服，一个小型的 PHP 项目只有有限的几个程序员去维护的话，其实这个问题并不明显，也不会成为困难，但是当项目变大，协作的人数变多的时候这就是一个需要去正式的问题。

### PHP 7 和 PHP < 7

在PHP 7 之前我们对参数没有太多的限定。

```php

function test(array $arr, App\Model\User $user)
{

}
```

PHP 7 之前我们只能对形参做这样的限制，无法限定基础类型。而 PHP7 之后我们可以对参数进行基础类型限制了。

```php
function test(array $arr, App\Model\User $user, string $string, int $int, bool $bool, float $float) : int 
{

}
```

可以对形参进行基础类型进行限制，并且方法和函数的返回值也可以提前告知。

这样的好处就是多少协作时，不需要他人调用一个他不熟悉或者从来没有用过的方法或者函数的时候去找到定义的人进行沟通，代码已经告诉他需要传递什么样的参数给函数，减少了多余的协作沟通成本。

当然无论是 PHP 7 还是 php 5.* 其实都有一个库可以做到这些功能，那就是 [SPL Type](http://php.net/manual/en/book.spl-types.php) ，但是这个库虽然以 SPL 命名但并不和 SPL 一样默认集成 PHP 里，它不是 PHP 的内核组件一部份，因此需要独立去安装。

当你使用 PHP 5.* 而不是 PHP 7 的时候，想要达到参数限定可以去安装它。若你使用 PHP 7 这个库的功能其实已经没什么用了，唯一还用的就是 SplEnum 这个类。

SplEnum 这个类实现了其他编程语言的枚举功能，那么枚举有什么用呢，我们可以假设这样一个场景，我们有一个很大的项目，有百人的开发团队或者更多，作为最初项目的文章模块的设计开发者你定义了一个创建文章的方法，这个方法需要两个参数。一个参数是外部表单提交或者其他途径获得的数据，一个是将要创建的文章的类型。

你可以通过文档、数据库注释、方法注释等一系列方法告诉调用者该如何传参，但是如果有的人这些都没读或者懒得去看那么你该如何限定他们用不正确的方法调用这个方法呢？

似乎好像没有办法了。

但是这个类就可以帮助我们，这里我使用了一个和扩展同样功能的PHP代码实现的包 `https://github.com/myclabs/php-enum`。

```PHP
<?php

---- # 1.定义 Enum 部分
namespace Type;

require './vendor/autoload.php';



class ArticleTypeEnum extends \MyCLabs\Enum\Enum
{
    const GENERAL = 1;
    const JOB = 2;
    const DISCUSS = 3;
    const SHARE = 4;
    const COURSE = 5;
    const LIFE = 6;
    const SPECIAL_COLUMN  = 7;
}

---- # 2.定义方法部分

function createArticle(array $data, ArticleTypeEnum $articleTypeEnum){
    // 创建对应的类型的文章
}

---- # 3.调用部分

try{
    testTypeEnum($data, new ArticleTypeEnum(ArticleTypeEnum::COURSE));
}catch (\Exception $exception){
    echo $exception->getMessage();
}
```

我在第一部分定义了枚举类，集成枚举基类 `\MyCLabs\Enum\Enum` ，在类中定义了几个文字类型常量。

第二部分是一个伪代码的一个创建文章的方法。

第三部分是调用部分，可以看见因为第二个参数已经被限定为枚举类，所以我必须传递一个枚举类型实例给创建方法，如果传递的不是枚举类的实例，那么程序就会抛出异常。

很简单的我就可以将外部调用时要创建的文章类型限定在规定范围内而不会出现未知的参数，或者一些脏数据进入到数据库中去，同时代码层面就告诉调用者应该如何去传递参数。

在传递第二个参数的时候你可能觉得麻烦，其实这个库作者已经想到了。


```PHP
<?php

---- # 3.调用部分

try{
    testTypeEnum($data, ArticleTypeEnum::GENERAL());
}catch (\Exception $exception){
    echo $exception->getMessage();
}
```


这样就可以了，这是利用 PHP 的魔术方法 `__callStatic()` 实现的调用相应常量。

这样我们就可以在代码层面控制参数类型以提高我们的程序可维护性了。

