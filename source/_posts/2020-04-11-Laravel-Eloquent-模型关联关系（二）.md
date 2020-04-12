---
title: Laravel Eloquent 模型关联关系（二）
date: 2020-04-11 16:47:12
categories:
  - PHP
tags:
  - PHP
  - Laravel
---

## 前言

前面我们已经演示了一对一，一对多，多对多的关系这三种是日常工作中很常见的关联关系，我们会介绍更加复杂的关联关系，分别是远层一对多和多态关联。

## 远层一对多关联

远层一对多在一对多关联的基础上加上了一个修饰词【远层】，意味着这个一对多关系不是直接关联，而是【远层】关联，远层关联需要借助中间表，前面我们讨论多对多关联也是借助中间表，但是远层一对多与其区别在于还是一对多的关联。

理论和实例结合才容易理解。如果博客系统是针对全球市场的话，可能针对不同的国家推出不同的用户系统和功能，然后中国用户过来就只展示中国用户发布的文章，日本用户过来就只展示日本用户发表的文章，这里就涉及了三张表，存储国家的 `countries` 表，存储用户的 `users` 表以及存储文章的 `posts` 表。

用户与文章是一对多的关联关系，国家与用户之间是一对多的关联（一个用户只能有一个国际），那么通过用户这张中间表，国家和文章之间也建立起来一对多的关联，只是这个关联不是直接的关联，而是【远层】的关联。

针对这个情况，我们说国家和文章之间是远层的一对多关联。

<!-- more -->

### 建立远层一对多关联关系

我们要先创建 Country 模型类及其对应数据库迁移：

```shell
php artisan make:model Country -m
```

编写新生成的数据库迁移文件对应迁移类的 up 方法如下：

```php
public function up()
{
    Schema::create('countries', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name', 100)->unique();
        $table->string('slug', 100)->unique();
        $table->timestamps();
    });
}
```

然后，编写迁移文件为 users 表新增一个 country_id 字段：

```shell
php artisan make:migration alter_users_add_country_id --table=users
```

编写新生成的迁移类文件如下：

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class AlterUsersAddCountryId extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->integer('country_id')->unsigned()->default(0);
            $table->index('country_id');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('country_id');
        });
    }
}
```

接下来，运行 php artisan migrate 让迁移生效。在 countries 表和 users 表填充一些测试数据便于后续测试。

准备好数据库、模型类并填充测试数据后，接下来，我们在 Country 模型类中通过 Eloquent 提供的 hasManyThrough 方法定义其与 Post 模型类之间的远层一对多关联：

```php
public function posts()
{
    return $this->hasManyThrough(Post::class, User::class);
}
```

其中，第一个参数是关联的模型类，第二个参数是中间借助的模型类。

这样，我们就可以在代码中通过 Country 模型实例获取归属于该国家的所有文章了，查询方式和前面其它关联查询一样，可以懒惰式加载，也可以渴求式加载:

```php
$country = Country::findOrFail(1);
$posts = $country->posts;
```

### hasManyThrough 方法的签名

同样，我们在通过 `hasManyThrough` 方法定义一个远层一对多关联关系的时候，并没有指定关联字段，因为我们在定义字段数据库字段、模型类的时候遵循了 Eloquent 底层的约定：

```php
public function hasManyThrough(
    $related, $through, $firstKey = null, $secondKey = null,
    $localKey = null, $secondLocalKey = null)
```

- \$related 关联模型类
- \$through 中间模型类
- \$firstKey 表示中间模型类与当前模型类的关联外键，按照默认约定，本例中凭借出来的字段是 `country_id`，正好和我们中间表 `users` 中新增的 `country_id` 吻合，所以不需要额外指定。
- \$secondKey 指的是中间模型类与关联模型类的关联外键，按照默认约定，在本例中凭借出来的字段是`user_id`，正好和我们在关联表 `posts` 中定义的 `user_id` 吻合，所以不需要额外指定。
- \$localKey 默认是当前模型类的主键 ID，
- \$secondLocalKey 是中间模型类的主键 ID

## 一对一的多态关联

接下来讲的三个关联关系都属于多态关联，多态关联允许目标模型通过单个关联归属多种类型的模型，根据模型之间关联关系类型，又可以将多态关联分为一对一，一对多，多对多三种关联。

这里的一对一关联是【多态】的，举个例子：假设在博客系统中用户可以设置头像，文章也可以设置缩略图，我们知道每个用户只能有一个头像，一篇文章只能有一个缩略图，所以此时：

- 用户和图片是一对一关联
- 文章和图片是一对一关联

通过多态关联，我们可以让用户和文章共享与图片的一对一关联，我们只需要在图片模型通过一次定义，就可以动态建立与用户和文章的关联。

### 如何建立一对一的多态关联

开始之前我们要创建图片模型类 Image 及其对应数据库迁移文件：

```shell
php artisan make:model Image -m
```

然后编写新创建的 create_images_table 迁移文件对应迁移类的 up 方法如下：

```php
public function up()
{
    Schema::create('images', function (Blueprint $table) {
        $table->increments('id');
        $table->string('url')->comment('图片URL');
        $table->morphs('imageable');
        $table->timestamps();
    });
}
```

其中 \$table->morphs('imageable') 用于创建 imageable_id 和 imageable_type 两个字段，其中 imageable_type 用于存放 User 模型类或 Post 模型类，而 imageable_id 用于存放对应的模型实例 ID，从而方便遵循默认约定建立多态关联。

运行 php artisan migrate 让迁移生效，准备好数据表和模型类后，接下来我们在模型类中建立一对一的多态关联。首先在 Image 模型类中通过 morphTo 建立其与 User/Post 模型类之间的关联：

Image.php 创建方法 `imageable`:

```php
public function imageable()
{
    return $this->morphTo();
}
```

我们不需要指定任何字段，因为我们在创建数据表和定义关联方法的时候都遵循了 Eloquent 底层的约定，还是来看下 morphTo 方法的完整签名：

```php
public function morphTo($name = null, $type = null, $id = null, $ownerKey = null)
```

- \$name 是关联关系的 i 你改成，默认是关联的方法名，本例为`imageable`
- $type $id 是结合 \$name 用于构建的关联字段，就是 `imageable_type` 和 `imageable_id`。由于我们的数据库字段和关联方法名都遵循了默认约定，所以不需要额外指定。如果你的数据库字段名是自定义的，比如 `item_id` 和 `item_type`，那么第一个参数值为 `item.
- \$ownerKey 是当前模型的主键 id

这样，我们就可以在 images 表中填充一些测试数据进行测试了，你可以借助填充器来填充，或者手动插入，需要注意的是在 imageable_type 字段中需要插入完整的类名作为类型，比如 `App\User` 或者 `App\Post`，以便 Eloquent 在插询的时候结合 imageable_id 字段利用反射构造对应的模型实例.

### 定义相对的关联关系

当然，我们在日常开发中，更常见的是获取某个用户的头像或者某篇文章的缩略图，这样，我们就需要在 User 模型中定义其与 Image 模型的关联：

```php
public function image()
{
    return $this->morphOne(Image::class, 'imageable');
}
```

然后在 Post 模型中定义其与 Image 模型的关联：

```php
public function image()
{
    return $this->morphOne(Image::class, 'imageable');
}
```

同样，因为我们遵循了 Eloquent 底册的约定，只需要传入最少的参数即可建立关联。morphOne 方法的完整签名如下：

```php
public function morphOne($related, $name, $type = null, $id = null, $localKey = null)
```

- \$related 表示关联的模型类
- $name $type $id 和前面 `morphTo` 方法的前三个参数一样，用于在关联表中拼接关联外键，在本例中就是 `imageable_type` 和 `imageable_id`，所以第三个和第四个参数不需要额外指定，当然如果你是用的是 item_id 和 item_type 字段需要将第二个参数设置为 item，如果结尾不是以 type 和 id 作为后缀，也需要通过 $type 和 \$id 参数传入。
- \$localKey 表示当前模型类的主键 ID。

## 一对多的多态关联

理解了一对一的多态关联之后，一对多的多态关联理解起来就简单多了，其实就是模型类与关联类之间的关联变成一对多了，只不过这个一对多是多态的，如何理解这个多态，其实就是在关联表引入了类型的概念，关联表中的数据不再是与某一张表有关联，而是与多张表有关联，具体是哪张表通过关联类型来确定，具体与哪条记录关联，通过关联 ID 来确定。能理解到这个层面基本上就可以通吃多态关联了。这种逻辑和面向对象中的多态很像（面向对象三大特性：继承、封装、多态），所以将其称作「多态关联」。

博客系统避免不了评论，一篇文章和单页面我们会区分开来，我们用户可以评论文章也可以评论单页面，但是这些评论可以统一存储在一个表中。

### 在模型类中构建一对多多态关联

首先还是要创建对应数据表和模型，我们先创建评论模型类 Comment 及其数据库迁移文件：

```shell
php artisan make:model Comment -m
```

编写新生成的 create_comments_table 迁移文件对应迁移类的 up 方法如下：

```php
public function up()
{
    Schema::create('comments', function (Blueprint $table) {
        $table->increments('id');
        $table->text('content')->comment('评论内容');
        $table->integer('user_id')->unsigned()->default(0);
        $table->morphs('commentable');
        $table->index('user_id');
        $table->softDeletes();
        $table->timestamps();
    });
}
```

然后创建一个 Page 模型类及其对应数据库迁移文件用于存放页面内容：

```shell
php artisan make:model Page -m
```

编写新生成的 create_pages_table 迁移文件对应迁移类的 up 方法如下：

```php
public function up()
{
    Schema::create('pages', function (Blueprint $table) {
        $table->increments('id');
        $table->string('title');
        $table->string('slug')->unique();
        $table->text('content');
        $table->integer('user_id')->unsigned()->default(0);
        $table->index('user_id');
        $table->softDeletes();
        $table->timestamps();
    });
}
```

运行 php artisan migrate 让迁移生效。

准备好数据库之后，我们通过填充器填充一些数据到刚创建的两张数据表。然后在 Comment 模型类中通过 Eloquent 提供的 morphTo 方法定义其与 Post 模型和 Page 之间的一对多多态关联：

```php
public function commentable()
{
    return $this->morphTo();
}
```

因为一个评论只会对应一篇文章/页面，所以，通过和一对一的多态关联同样的 morphTo 方法定义其与文章和页面的关联关系即可。和前面的一对一多态关联一样，因为我们的数据表字段和关联方法名都遵循了 Eloquent 底层的默认约定，所以不需要指定任何额外参数，即可完成关联关系的构建。这些默认约定我们在上面一对一多态关联中已经详细列出，这里就不再赘述了。

这样，我们就可以通过 Comment 实例查询其归属的文章或页面了：

```php
$comment = Comment::findOrFail(1);
$item = $comment->commentable;
```

### 定义相对的关联关系

同样，我们在日常开发中，更多的是通过文章或页面实例获取对应的评论信息，比如在文章页或页面页获取该文章或页面的所有评论。为此，我们需要在 Post 模型类和 Page 模型类中定义其与 Comment 模型的关联关系，这需要通过 morphMany 方法来实现：

```php
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}
```

和 morphOne 方法一样，因为我们遵循了 Eloquent 底层的默认约定，所以只需要传递很少的必要参数就可以定义关联关系了，morphMany 方法的完整签名如下：

```php
public function morphMany($related, $name, $type = null, $id = null, $localKey = null)
```

这些参数的含义和 morphOne 方法完全一样，这里就不再赘述了。如果想要在 Post 模型下获取对应的所有评论，可以这么做：

```php
$post = Post::with('comments')->findOrFail(23);
$comments = $post->comments;
```

对应的关联查询底层 SQL 语句是：

```mysql
select
  *
from
  `comments`
where
  `comments`.`commentable_id` in (23)
  and `comments`.`commentable_type` = "App\Post"
  and `comments`.`deleted_at` is null
```

## 多对多的多态关联

多对多的多态关联比前面的一对一和一对多更加复杂，但是有了前面讲解的基础，理解起来也很简单。你可以类比下常规的多对多关联，现在加入了「多态」的概念，常规的多对多需要借助中间表，多态的也是，只不过此时不仅仅是两张表之间的关联，而是也要引入类型字段。

还是以文章和标签的关联为例，在常规的多对多关联中，中间表只需要一个标签 ID 和文章 ID 即可建立它们之间的关联，但当我们添加新的内容类型，比如页面、视频、音频，它们也有标签，而且完全可以共享一张标签表，此时仅仅一个文章 ID 已经满足不了定义内容与标签之间的关联了，所以此时引入多对多的多态关联，和前面两种多态关联思路一样，只是在多对多关联中，我们需要在中间表中引入类型字段来标识内容类型，将原来的文章 ID 调整为内容 ID，这样就可以从数据库层面满足不同内容类型与标签之间的关联了。

所以你可以看到从一对一、一对多（远层一对多）、多对多、一对一多态关联、一对多多态关联、多对多多态关联，它们之间是层层递进的，理解了前面的，后面的也就更好理解。

下面我们以标签与文章、页面关联关系为例，演示如何定义和使用多对多的多态关联。

### 在模型类中定义多对多的多态关联

首先我们要废弃原来的 post_tags 数据表，创建一个新的 taggables 数据表来构建不同内容类型与标签之间的关联：

```shell
php artisan make:migration create_taggables_table --create=taggables
```

编写新生成的 create_taggables_table 迁移文件对应迁移类的 up 方法如下：

```php
Schema::create('taggables', function (Blueprint $table) {
    $table->increments('id');
    $table->integer('tag_id');
    $table->morphs('taggable');
    $table->index('tag_id');
    $table->unique(['tag_id', 'taggable_id', 'taggable_type']);
    $table->timestamps();
});
```

运行 php artisan migrate 让迁移生效。然后通过填充器填充一些测试数据到新生成的 taggables 数据表。

接下来我们在 Tag 模型类中通过 Eloquent 提供的 morphedByMany 方法定义其与其他模型类的多对多多态关联：

```php
public function posts()
{
    return $this->morphedByMany(Post::class, 'taggable');
}

public function pages()
{
    return $this->morphedByMany(Page::class, 'taggable');
}
```

和之前一样，因为我们遵循了 Eloquent 底层的默认约定，所以我们只需传递必需参数，无需额外配置即可定义关联关系，我们来看下 morphedByMany 方法的完整签名：

```php
public function morphedByMany($related, $name, $table = null, $foreignPivotKey = null, $relatedPivotKey = null, $parentKey = null, $relatedKey = null)
```

- \$related 表示关联的模型类
- \$name 表示关联的名称，和定义中间表数据迁移的时候 `morphs` 方法中指定的值一致，也就是 `taggable`.
- \$table 表示中间表名称，默认是第二个参数 `$name` 的复数形式，这里就是 `taggables` 了，因为我们创建数据表的时候遵循了这个约定，所以不需要额外指定
- \$foreignPivotKey 表示当前模型类在中间表中的外键，默认拼接结果是 tag_id，和我们在数据表中定义的一样，所以这里不需要额外指定。
- \$relatedPivotKey 表示默认是通过 `$name` 和 `_id` 组合而来，表示中间表中的关联 ID 字段，这里组合结果是 taggable_id，和我们定义的一致，也不需要额外指定。
- \$parentKey 默认表示当前模型类的主键 ID，即与中间表中 `tag_id` 关联的字段。
- \$relatedKey 表示关联模型类的主键 ID，这个因 \$related 指定的模型而定。

如果你不是按照默认约定的规则定义的数据库字段，需要明确每一个参数的含义，然后传入对应的参数值，和之前一样，对新手来说，还是按照默认约定来比较好，免得出错。

定义好上述关联关系后，就可以查询指定标签模型上关联的文章/页面了：

```php
$tag = Tag::with('posts', 'pages')->findOrFail(53);
$posts = $tag->posts;
$pages = $tag->pages;
```

### 定义相对的关联关系

最后，我们还可以在 Post 模型类或 Page 模型类中通过 Eloquent 提供的 morphToMany 方法定义该模型与 Tag 模型的关联关系（两个模型类中定义的方法完全一样）：

```php
public function tags()
{
    return $this->morphToMany(Tag::class, 'taggable');
}
```

因为我们遵循和 Eloquent 底层默认的约定，所以指定很少的参数就可以定义多对多的多态关联，morphToMany 方法的完整签名如下：

```php
public function morphToMany($related, $name, $table = null, $foreignPivotKey = null, $relatedPivotKey = null, $parentKey = null, $relatedKey = null, $inverse = false)
```

其中前七个参数和 morphedByMany 方法含义一致，只不过针对的关联模型对调过来，最后一个参数 \$inverse 表示定义的是否是相对的关联关系，默认是 false。如果你是不按套路出牌自定义的字段，需要搞清楚以上参数的含义并传入自定义的参数值。

定义好上述关联关系后，就可以通过 Post 模型或 Page 模型获取对应的标签信息了：

```php
$post = Post::with('tags')->findOrFail(6);
$tags = $post->tags;
```

对应的底层查询 SQL 语句是：

```mysql
select
  `tags`.*,
  `taggables`.`taggable_id` as `pivot_taggable_id`,
  `taggables`.`tag_id` as `pivot_tag_id`,
  `taggables`.`taggable_type` as `pivot_taggable_type`
from
  `tags`
  inner join `taggables` on `tags`.`id` = `taggables`.`tag_id`
where
  `taggables`.`taggable_id` in (6)
  and `taggables`.`taggable_type` = "App\Post"
```

## 总结

至此，关于 Eloquent 模型内置支持的七种关联关系定义我们就已经全部介绍完了。还是要多动手练习才能慢慢熟悉。
