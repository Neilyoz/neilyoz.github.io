---
title: Laravel Eloquent 模型关联关系（一）
date: 2020-04-11 11:47:12
categories:
  - PHP
tags:
  - PHP
  - Laravel
---

## 前言

Laravel Eloquent 模型支持的关联关系包括以下其中：

- 一对一
- 一对多
- 多对多
- 远层一对多
- 多态关联（一对一）
- 多态关联（一对多）
- 多态关联（多对多）

不着急，我们一个一个的看如何使用。

<!-- more -->

## 一对一

一对一是最简单的关联关系，一般可用于某个扩展表与主表之间的关联关系。比如大型系统中，我们的用户表通常用于最基本的信息的存储，如：邮箱、用户名、密码等放在主表，然后用户的身份证信息，个性签名放在另外一张扩展表。需要的时候才会去扩展表取数据，从而提高查询性能。

在开始之前，我们先通过数据迁移建立一张 `user_profiles` 表，并创建对应的 `UserProfile`，这可以通过 `php artisan` 命令一次完成：

```shell
php artisan make:model UserProfile -m
```

在生成的 `create_user_profiles` 迁移文件中编写迁移类的 `up` 方法如下：

```php
/**
  * Run the migrations.
  *
  * @return void
  */
public function up()
{
    Schema::create('user_profiles', function (Blueprint $table) {
        $table->id();
        $table->bigInteger("user_id")->comment("用户ID");
        $table->string("card_number")->comment("身份证号码");
        $table->timestamps();
    });
}
```

注意：我们在 `user_profiles` 表中添加了 `user_id` 字段用于指向所属用户，从而建立与 `users` 表的关联。运行 `php artisan migrate` 在数据库创建这张数据表。

准备好数据表之后，接下来我们创建 `users` 表和 `user_profiles` 表之间的关联，首先，我们在 `User` 模型类中通过 `hasOne` 方法定义它与 `UserProfile` 的的一对一关联，User 类中加入 `profile` 方法

```php
public function profile() {
    return $this->hasOne(UserProfile::class);
}
```

实际的使用时：

```php
$user = User::first();
$user->profile;
```

### hasOne 方法的签名

```php
public function hasOne($related, $foreignKey = null, $localKey = null)
```

- \$related 是关联模型的类名
- \$foreignKey 是关联模型所属表的外键
- \$localKey 是关联表的外键关联到当前模型所属表的那个字段

### 建立相对的关联关系

通常我们都是通过 `User` 模型获取 `UserProfile` 模型，但是有时候我们可以需要反过来通过 `UserProfile` 反查询所属的 `User` 模型，Eloquent 底层也为我们提供了相应的 `belongsTo` 方法来建立一对一关联关系，我们在 `UserProfile` 关联模型定义它与 `User` 模型的关联如下：

给 UserProfile.php 中加入 `user` 方法:

```php
public function user()
{
    return $this->belongsTo(User::class);
}
```

实际的使用时：

```php
$userProfile = UserProfile::first();
$userProfile->user;
```

### belongsTo 方法的签名

和 `hasOne` 方法一样，`belongsTo` 方法也是遵循了默认的约定签名如下：

```php
public function belongsTo($related, $foreignKey = null, $ownerKey = null, $relation = null);
```

- \$related 是关联模型的类名
- \$foreignKey 是当前模型所属表的外键
- \$owerKey 是关联模型类所属表的主键
- \$relation 是关联关系方法名，也是关联关系动态属性名

## 一对多

一对多关联是我们日常开发中经常碰到的一种关联关系。以博客系统为例，一个用户可以发布多篇文章，反过来，一篇文章只能归属于一个用户，那么用户和文章之间就是一对多的关系，同样，用户可以发布多条评论，一条评论只能归属于一个用户，用户与评论之间也是一对多的关系。

### 创建 posts 表以及对应迁移文件

```shell
php artisan make:model Post -m
```

对应迁移文件中的 Up 方法：

```php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->bigInteger("user_id")->comment("所属用户id");
        $table->string('title')->comment("文章标题");
        $table->longText('content')->comment("文章内容");
        $table->bigInteger('view')->comment("查看次数");
        $table->timestamps();
        $table->softDeletes();
    });
}
```

### 创建 User 模型对应的 posts 方法

要定义用户文章之间的一对多关联，可以在 `User` 模型类中通过 `Eloquent` 底层的提供的 `hasMany` 方法来实现：

```php
public function posts()
{
    return $this->hasMany(Post::class, "user_id", "id")
}
```

实际的使用时：

```php
$user = User::first();
$user->posts;
```

### hasMany 方法的签名

```php
public function hasMany($related, $foreignKey = null, $localKey = null)
```

- \$related 是关联模型的类名
- \$foreignKey 是关联模型所属表的外键
- \$localKey 是关联表的外键关联到当前模型所属表的那个字段

### 建立相对的关联的关系

我们将 `Post` 模型中的关联关系调用方法名为 `author`，这样，我们就需要手动指定更多的 `belongsTo` 方法传入参数：

Post.php 中加入 `author` 方法：

```php
public function author()
{
    return $this->belongsTo(User::class, 'user_id', 'id', 'author');
}
```

实际的使用时：

```php
$post = Post::first();
$post->author;
```

## 多对多

多对多关联在实际开发中很常见，还是以博客系统为例子，我们会为每篇文章设置标签，一篇文章往往有多个标签，反过来，一个标签可能会归属于多篇文章，这时，我们说文章和标签之间是多对多的关联关系。

多对多关联比一对一和一对多关联复杂一些，需要借助一张中间表才能建立关联关系。以文章标签为例，文章表已经存在，还需要创建一张 `tags` 表和中间表 `post_tags` 表。首先创建 `Tag` 模型类及其对应数据表 `tags` 迁移文件：

```shell
php artisan make:model Tag -m
```

编写 `create_tags_table` 迁移文件对应的 `up` 方法如下：

```php
public function up()
{
    Schema::create('tags', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name', 100)->unique()->comment('标签名');
        $table->timestamps();
    });
}
```

然后创建 `post_tags` 数据表迁移文件：

```shell
php artisan make:migration create_post_tags_table --create=post_tags
```

编写对应迁移类的 `up` 方法如下：

```php
public function up()
{
    Schema::create('post_tags', function (Blueprint $table) {
        $table->increments('id');
        $table->integer('post_id')->unsigned()->default(0);
        $table->integer('tag_id')->unsigned()->default(0);
        $table->unique(['post_id', 'tag_id']);
        $table->timestamps();
    });
}
```

运行 `php artisan migrate` 让迁移生效。

接下来，我们在 `Post` 模型类中定义其与 `Tags` 模型类的关联关系，通过 Eloquent 提供的 `belongsToMany` 方法来实现：

```php
public function tags()
{
    return $this->belongsToMany(Tag::class, 'post_tags');
}
```

通过数据填充器填充一些数据到 `tags` 表和 `post_tags` 表，这样我们可以通过关联查询查询指定的 `Post` 模型上的标签信息了：

```php
$post = Post::with('tags')->first();
$tags = $post->tags;
```

### belongsToMany 方法的签名

可以看到我们在定义多对多关联的时候，也没有指定哪些字段进行关联，这样是遵循 Eloquent 底层默认约定的功劳， `belongsToMany` 方法签名如下：

```php
public function belongsToMany($related, $table = null, $foreignPivotKey = null, $relatedPivotKey = null, $parentKey = null, $relatedKey = null, $relation = null)
```

- \$related 是关联模型的类名
- \$table 是建立多对多关联的中间表名，该表名默认凭借规则如下：
  - 当前模型+下划线+关联模型，这里是自定义的，默认转化是根据两个表名首字母排序之后，用下划线分割，如果默认的话，这里应该是`post_tag`，但是为了遵循 Laravel 表明为复数规定，我们自行定义一回。
- \$foreignPivotKey 是中间表中当前模型类的外键，默认拼接规则和前面一对一，一对多一样，所以本例中是 `posts` 表的 `post_id` 字段。
- \$relatedPivotKey 是中间表中当前模型类的外键，只不过作用于关联模型，所以本例中是`tags`表的`tag_id`字段。
- \$parentKey 表示对应当前模型哪个字段（即 \$foreignPivotKey 映射到当前模型所属表的哪个字段），默认是主键 ID，即 `posts` 表的 `id` 字段，所以这里不需要额外指定。
- $relatedKey 表示对应关联模型的哪个字段（即 $relatedPivotKey 映射到关联模型所属表的哪个字段），默认是关联模型的主键 ID，即 tags 表的 id 字段，所以这里也不需要额外指定。
- \$relation 表示关联关系名称，用于设置查询结果中的关联属性，默认是关联方法名。

### 建立相对的关联联系

与之前的关联关系一样，多对多关联也支持建立相对的关联关系，而且由于多对多的双方是平等的，不存在谁归属谁的问题，所以建立相对关联的方法都是一样的，我们可以在 Tag 模型中通过 belongsToMany 方法建立其与 Post 模型的关联关系：

```php
public function posts()
{
    return $this->belongsToMany(Post::class, 'post_tags');
}
```

实际的使用时：

```php
$tag = Tag::with('posts')->first();
$posts = $tag->posts;
```

### 获取中间表字段

```php
$post = Post::first();
$post->tags;
```

Eloquent 还提供了方法允许你获取中间表的字段，你仔细查询结果字段，会发现 `relations` 字段中有一个 `pivot` 属性，中间表字段就存放在这个属性对象上。

中间表只有对应的中间表关联 id 没有时间戳，我们可以通过这个方式添加：

```php
public function tags()
{
    return $this->belongsToMany(Tag::class, 'post_tags')->withTimestamps();
}
```

除了这样，你还在中间表定义了额外的字段信息，比如 `user_id`，通过 `with` 方法传入字段就将其返回了：

```php
public function tags()
{
    return $this->belongsToMany(Tag::class, 'post_tags')->withPivot('user_id')->withTimestamps();
}
```

### 自定义中间表模型类

你可以通过自定义中间表对应模型类实现更多自定义操作，中间表模型继承自 `Illuminate\Database\Eloquent\Relations\Pivot`，Pivot 也是 Eloquent Model 类的子类，只不过为中间表操作定义了很多方法和属性，比如我们创建一个自定义的中间表模型类 PostTag：

```php
namespace App;

use Illuminate\Database\Eloquent\Relations\Pivot;

class PostTag extends Pivot
{
    protected $table = 'post_tags';
}
```

这样，我们在定义多对多关联关系的时候指定自定义的模型类了：

```php
public function tags()
{
    return $this->belongsToMany(Tag::class, 'post_tags')->using(PostTag::class);
}
```

### 更多中间表操作

此外，如果你觉得 pivot 可读性不好，你还可以自定义中间表实例属性名称：

```php
$this->belongsToMany(Tag::class, 'post_tags')->as('taggable')->withTimestamps();
```

这样，就可以通过 \$tag->taggable->created_at 访问中间表字段值了。

还可以通过中间表字段值过滤关联数据（支持 where 和 in 查询）：

```php
return $this->belongsToMany(Tag::class, 'post_tags')->wherePivot('user_id', 1);
return $this->belongsToMany(Tag::class, 'post_tags')->wherePivotIn('user_id', [1, 2]);
```

## 总结

这里只是说了一个基础，更多高级的在后面呈现给大家。
