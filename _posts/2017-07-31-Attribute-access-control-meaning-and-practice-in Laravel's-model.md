---
layout: post
title: Laravel 的 Eloquent模型 中的属性访问控制意义及实践
---

我们在定义一个自己的 Laravel 模型时都需要继承 Illuminate\Database\Eloquent\Model 这个类，在这个类中定义了很多模型相关的操作和行为。今天我们来重点说说 Model 中包含的属性。

在框架定义属性中访问权限一般有 `protected` 和 `public` 两种，这两种属性是在我们继承基础模型后可以重写的，那为什么不统一都是`protected`，是因为有一些属性我们需要去动态改变。

`protected` 的属性我们一般只重写一次或者使用基类提供的默认值，例如：

```PHP
protected $table
protected $primaryKey = 'id';
protected $keyType = 'int';
```

以上这些都是只需要在继承基类的时候重写就可以了，分别表示模型的对应的表名、主键字段名、主键类型。

在软件的生命周期内这些属性都不需要再出改变。

而有几个特殊的属性使用的 `public` 权限：

```PHP
public $incrementing = true;
public $exists = false;
public $wasRecentlyCreated = false;
public static $snakeAttributes = true;
public static $manyMethods = [
        'belongsToMany', 'morphToMany', 'morphedByMany',
        'guessBelongsToManyRelation', 'findFirstMethodThatIsntRelation',
    ];
public $timestamps = true;
```

这些属性有些在 `Illuminate\Database\Eloquent\Model` 有些在 `Model` 使用的 `trait` 中。

现在我们就拿 `$exists`、`$timestamps` 这两个来举例说明为什么框架会将他们的访问属性定义为 `public`。

### $exists

这个属性用来指示模型中数据是否存在，典型的判断在 model 的 save() 方法中，如果 `$exists = true` 那么 save() 执行的就是数据库的 update 操作，如果是 false 执行的就是 instert 操作。

比如你有一个这样的场景，这个模型在做正常的存储的时候都是执行正常的 save 逻辑，而在特殊的时候需要全部操作都是 inster 操作就可以在调用 save 前将此属性手动赋值为 true。

这样的场景有典型应用就是用户。

```PHP

public function register(Request $rquest)
{

    $model = new User();
    
    $model->exists = false;
    
    $model->create($request->only('email', 'name', 'password'))
    
    $model->save()

}

public function updateName(Request $rquest)
{

    $model = User::find($id, $request->get('id'));
    
    $model->exists = true;
    
    $model->name = $request->get('name')
    
    $model->save()

}

``` 

以上是一个伪代码的用户注册和用户更新字段方法，我们手动去将$exists属性修改为我们操作想要其达到效果以保证其每次执行都是我们想要的效果而非产生我们不想要的副作用。

当然这个例子可能不是一个最佳实践，因为这些判断框架内部都帮我们坐在，这个属性最大作用还是用于判断，比如你调用了一些模型操作，想要查看一看一下这个属性的bool值，以帮助你做下一步的判断，以执行不同的逻辑时更有作用。

### $timestamps

这个属性是我将要着重说的属性，因为我认为它在我的代码中目前找到了最佳实践。

我现在有一个表，是文章表，我在文章表里冗余了需要文章的属性，比如阅读数，关注数，评论数，收藏数，就是为了减少获取这篇文章的时候那些关联查询，以使得文章的首次加载速度更快。对我来说此时获取一个关注的数去查询一次文章关注表是不划算的。

但是我在获取文章的是又要将阅读数自增，以反映有人点开并阅读了此文章一次。但是鉴于Laravel默认会自动管理我的文章模型 `created_at` 和 `updated_at` 这个两个字段，我每次更改阅读数，模型同时会更新记录的 `updated_at` 到最新时间，这并不是我想要的结果。

于是我将 `$timestamps` 在模型定义为 false ， 这样我的时间就不会被框架自动管理。

但是这样做又产生一个问题，虽然我不想我在更改阅读数，关注数这些冗余数据是让框架自动管理我的记录时间，但是我却想在创建和文章编辑提交时更新时间。

当然方法有，我可以手动传递当前时间给模型，也是可以实现的。但是毕竟这样不够优雅，且本来框架提供的功能我却要自己再去实现它，让我觉得这是一种资源浪费，而且手动维护这些时间是繁琐易出错的。

当我看到这个访问控制的 `public` 的时候，我觉得我可以同时实现我的这两个需求，而且不需要自己手动传递时间。

```PHP

/**
* 查看文章
*/
public function showArticle($id)
{
    $article = Article::find($id);
    
    $article->timestamps = false; // 模型已经定义，可省略
    
    $article->increment('views');
    
    $article->save();

    return $article;
}

/**
* 创建文章
*/
public function createArticle(Request $request)
{
    $article = Article::crete($request->all());
    
    $article->timestamps = true;
    
    $article->save();

    return $article;
}

/**
* 更新文章
*/
public function updateArticle($id, Request $request)
{
    $article = Article::find(id);
    
    $article->timestamps = true;
    
    $article->update($request->all());

    return $article;
}
```

这样我就不用手动传递时间给模型了，模型会用自身的功能帮我维护记录的时间。

### 总结

这些 `public` 的属性就是方便外部可以访问它们或者可以根据自己场景来动态的改变它们，以符合你的期望，这些属性我并没有完全研究


