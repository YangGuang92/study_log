# laravel学习记录

### 1. 查询构造器

> DB::第一个使用的是静态的，返回的是一个查询对象，然后使用->where()等链式请求，返回的也是一个查询对象，所以后面可以继续使用链式操作，但是当最后遇到比如get()返回结果等方法的时候，停止连缀。**所以返回结果必须放在最后**
>
> tips: 打印sql有三种方法: toSql(),dump(),dd()
>
> toSql()之后还需要打印出来
>
> dump()和dd()直接打印出来

##### 1.1 构造器的查询、分块、聚合

- 查询

  `table()`方法引入相应的表，

   ```
  常用的方法有:
  get()  获取所有数据
  first() 获取第一条数据
  value() 获取第一条数据的指定字段
  find() 获取指定id的一条数据
  pluck() 获取某一列的数据
  例如：
  $user = DB::table('user')->get();
   ```

- 分块

  如果一次性处理成千上万条数据，防止读取出错，可以使用`chunk()`方法对数据进行分批操作，**该方法需要先对数据进行排序，该方法不需要return 直接再闭包函数中执行逻辑**

  ```php
  //切块分割执行，每次读取3条，对id进行排序
  DB::table('user')->orderBy('id')->chunk(3,function ($users){
      foreach ($users as $user) {
          echo $user->id;
          echo '<br>';
      }
  });
  ```

- 聚合

  构造器提供了：`count()`,`max()`,`min()`,`avg()`,`sum()`

  ```php
  DB::table('user')->count();
  DB::table('user')->max('age');
  ....
  ```

##### 1.2 构造器的查询表达式

1. `select()`方法可以设置你想要的列，也可以给列设置别名

   ```php
   $user = DB::table('user')->select('username as name','age')->get();
   dump($user->toArray());
   ```

2. `addSelect()`方法，可以在已经构造好的查询构造器上再添加想要的字段

   ```PHP
   $user = DB::table('user')->select('name as n', 'age');
   $user->addSelect('id')->get();
   ```

3. `DB::raw()`方法可以在`select()`内部实现原生表达式,否则解析错误

   ```php
   //因为有其他的字段，所以必须要加上groupBy()分组
   $user = DB::table('user')->select(DB::raw('count(id) as count,age'))->groupBy('id')->get();
   ```

4. 也可以用`selectRaw()`方法实现原生语句

   ```php
   $user = DB::table('user')->selectRaw('count(id) as count,age')->groupBy('id')->get();
   ```

5. `havingRaw()`来实现having的功能，其用法和`selectRaw()`类似

   ```php
   $user = DB::table('user')->selectRaw('count(id) as count,age')->groupBy('id')->havingRaw('count>=1')->get();
   ```

6. ``toSql()`可以显示查询构造器的sql语句,**注意：其条件的值框架都会用`?`代替**

   ```php
   //toSql()之后不能跟其他的方法，因为这个返回的是一个字符串
   $user = DB::table('user')->select('name as n', 'age')->where('name','like','张%')->toSql();
   dump($user);
   //显示的是：select `name` as `n`, `age` from `user` where `name` like ?
   ```

##### 1.3 where条件

1. `where()`条件，完整性是需要三个字段表达式或者值，但是如果是`=`的情况，可以省略第二个参数，第二个参数可以为`like`,`!=`,`>`....等系统支持的运算符，`where()`中可以添加数组，表示多个条件以`and`连接

   > 注意：where也可以查询json数据类型，具体写法如下
   >
   > mysql 5.7 之后开始支持json格式，具体用法可以看: https://segmentfault.com/a/1190000019582491 

   ```php
   //等于
   $user = DB::table('user')->select('name as n', 'age')->where('age',12)->get();
   //大于
   $user = DB::table('user')->select('name as n', 'age')->where('age','>',12)->get();
   //like
   $user = DB::table('user')->select('name as n', 'age')->where('name','like','张%')->get();
   //where
   $user = DB::table('user')->select('name as n', 'age')->where([['name','like','张%'],['age','>',12]])->get();
   //where是一个json里面的属性
   //该条sql打印出来是：select * from `user` where json_unquote(json_extract(`list`, '$."info"')) = ?  所以说也就相当于sql中的 list->>'$.info'
   $user = DB::table('user')->where('list->info','{"info1": "aaaa", "info2": "bbbb"}')->get();
   ```

2. `orWhere()`连接再`where()`条件之后来表示`or`，`orWhere()`中可以添加闭包来添加更加复杂的条件，比如`or`里面的也有and或者or

   ```php
   //普通的orwhere条件
   $user = DB::table('user')->select('name as n', 'age')
               ->where('age',12)
               ->orWhere('age','>',20)
               ->get();
   //复杂的orwhere条件
   $user = DB::table('user')->select('name as n', 'age')
               ->where('age',12)
               ->orWhere(function ($query){
                   $query->where('age',30)
                       ->orWhere('name','like','郑%');
               })
               ->get();
   ```

3. `whereBetween()`可以添加`between`条件,同类型的还有`whereNotBetween()`,`orWhereBetween()`,`orWhereNotBetween()`,不过or开头的两个方法前面必须要加上`where()`条件，才能表示`or`

   ```php
   $user = DB::table('user')->select('name as n', 'age')
               ->whereBetween('age',[12,30])
               ->get();
   ```

4. `whereIn()`表示`in`的用法，其参数格式跟`whereBetween()`相同，同类型的方法还有`whereNotIn()`,`orWhereIn()`,`orWhereNotIn()`，不过or开头的两个方法前面必须要加上`where()`条件，才能表示`or`

5. `whereNull()`表示`is null`用法,同类型的方法还有`whereNotNull()`,`orWhereNull()`,`orWhereNotNull()`，不过or开头的两个方法前面必须要加上`where()`条件，才能表示`or`

   ```php
   $user = DB::table('user')->whereNull('age')->get();
   ```

6. `whereDate()`获取时间字段的条件，用法同`where()`类似，还有`whereYear()`，不过打印sql语句发现，**该方法其实是where条件中使用函数，这样是不好的，因为用不到索引了**，所以最好还是用`where()`方法

   ```php
   $user = DB::table('user')->whereDate('update_time',date('Y-m-d'))->get();
   //其sql是：select * from `user` where date(`update_time`) > ?
   ```


##### 1.4 构造器的排序、子查询

1. `orderBy()`排序，同sql中的`orderby`

2. `offect()`和`limit()`分页

3. `when()`可以设置条件选择，根据true或false来执行不同的闭包,true的话执行第一个闭包，false执行第二个 

   ```php
   $user = DB::table('user')->when(true,function ($query){
       $query->where('id',1);
   }, function ($query) {
       $query->where('id',2);
   })->get();
   ```

4.  使用`where()`和`whereExists`来实现一个子查询,当然，其他的where查询也是可以实现的

   ```php
   //这条是查在user表中有数据的score表的信息
   //whereColumn()方法用来连接两张表的关联关系
   $user = DB::table('score')->where('user_id',function ($query) {
       $query->select('id')
           ->from('user')
           ->whereColumn('user.id', 'score.user_id');
       //                ->where('user.id',1);
   })->get();
   //这条是查在score表中有数据的user表信息
   //使用whereExists查询
   //selectRaw(1) 就是select 1 这是让查询效率更高的一种用法
   $user = DB::table('user')->whereExists(function ($query) {
       $query->selectRaw(1)
           ->from('score')
           ->whereColumn('user.id', 'score.user_id');
   })->get();
   ```

##### 1.5 构造器的join查询

1. `join()`，`leftJoin()`，`rightJoin()`

   ```php
   //因为有些字段重复，所以需要selectRaw来写一些原生的查询字段
   $user = DB::table('user')
               ->selectRaw('user.id,user.name,user.age,score.id as score_id,score.score')
               ->leftJoin('score','user.id','=','score.user_id')
               ->where('user.id',1)
               ->get();
   ```

2.  你可以使用 `joinSub`，`leftJoinSub` 和 `rightJoinSub` 方法关联一个**查询作为子查询** 

   ```php
   $latestPosts = DB::table('posts')
                      ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                      ->where('is_published', true)
                      ->groupBy('user_id');
   
   $users = DB::table('users')
           ->joinSub($latestPosts, 'latest_posts', function ($join) {
               $join->on('users.id', '=', 'latest_posts.user_id');
           })->get();
   ```

3. `union()`代表联合查询

   ```php
   $first = DB::table('users')
               ->whereNull('first_name');
   
   $users = DB::table('users')
               ->whereNull('last_name')
               ->union($first)
               ->get();
   ```

##### 1.5 构造器的增、删、改

1. `insert()`新增，返回插入成功或者失败，`insertGetId()`返回的是主键id

   ```php
   //插入单条数据
   DB::table('users')->insert(
       [
           'email' => 'john@example.com',
           'votes' => 0
       ]
   );
   //插入多条数据
   DB::table('users')->insert(
       [
           'email' => 'john@example.com',
           'votes' => 0
       ],
       [
           'email' => 'example.com',
           'votes' => 2
       ],
   );
   ```

2. `update()`更新

   `updateOrInsert()`：存在就更新，不错在就插入

   `increment()`，`decrement`: 自增或者自减， 这两种方法都至少接收一个参数：需要修改的列。第二个参数是可选的，用于控制列递增或递减的量 

   ```php
   //update更新 
   $return = DB::table('users')
                 ->where('id', 1)
                 ->update(['votes' => 1]);
   //updateOrInsert更新,注意，必填字段要写满
   //第一个参数的键和值对来查找匹配的数据库记录。 如果记录存在，则使用第二个参数中的值去更新记录。 如果找不到记录，将插入一个新记录，更新的数据是两个数组的集合
   DB::table('users')
       ->updateOrInsert(
           ['email' => 'john@example.com', 'name' => 'John'],
           ['votes' => '2']
       );
   //注意！！！更新json数据，options是json类型数据，enabled是里面的属性
   $affected = DB::table('users')
                 ->where('id', 1)
                 ->update(['options->enabled' => true]);
   ```

3. `delete()`删除数据,这个函数可以传参数为表的主键，也可以用`where()`条件作为前置条件

### 2. ORM模型

> 超级好用的代码提示插件,请务必安装！！！
>
> composer require barryvdh/laravel-ide-helper
>
> php artisan ide-helper:generate - 为 Facades 生成注释
> php artisan ide-helper:models - 为数据模型生成注释
> php artisan ide-helper:meta - 生成 PhpStorm Meta file
>
>  由于使用此扩展包会生成相应的代码结构文件，这些文件可能只有当前的开发者的 IDE 需要，因此需要添加对应配置到 `.gitignore` 文件中： 
>
>  .idea _ide_helper
>
> .php _ide_helper_models
>
> .php .phpstorm
>
> .meta.php 

**查询构造器的方法在ORM中都是可以用的，不管是增删改查**

##### 2.1 模型的定义

常用的定义会有:

1. 设置表名：通过设置table属性来设置表名

   ```php
   protected $table = 'my_flights';
   ```

2. 更改创建时间和更新时间的字段名

   ```php
   const CREATED_AT = 'creation_time';
   const UPDATED_AT = 'update_time';
   ```

3. 不使用自动生成的时间戳

   ```php
   public $timestamps = false;
   ```

   

##### 2.2 模型的增删改

在模型中除了下面的方法，也可以使用查询构造器的增删改操作

1. **新增**，可以使用`save()`方法，返回的`true`或者`false`

   还有一种方法是通过`create()`，该方法返回一个对象模型，使用该方法需要在模型内设置`fillable`,该属性表示可以被批量复制的属性, `guarded`属性跟`fillabel`正好相反,表示 不允许批量赋值的数组

   ```php
   //通过save新增
   $user = new User();
   $user->name = '孙悟空';
   $user->age = '550';
   $user->list = '{"id": 1223, "info": {"info1": "aaaa", "info2": "bbbb"}, "name": "孙悟空"}';
   $user->save();
   //通过create新增
   //首先需要在model中设置fillable
    protected $fillable = [
           'id',
           'name',
           'age',
           'list'
       ];
   //然后再调用处使用create
   
   
   ```

2. **更新**，更新单条数据可以用`save()`，但是更新多条数据的话就不能用save了，就需要用查询构造器的相关用法

   ```php
   //修改单条数据
   $user = User::find(6);
   $user->name = '孙行者';
   $user->save();
   //批量修改,就是查询构造器的用法
   User::where(....)->update(...);
   ```

3. **删除**,返回1或者0

   ```php
   //通过条件删除
   User::where('name','沙和尚')->delete();
   //通过主键删除,也可以批量删除
    User::destroy(5);
   User::destroy([1,2,3]);
   ```


##### 2.2 软删除

首先需要在模型中添加`use SoftDeletes;`，然后在该表中加入`deleted_at`来应用软删除，然后在实际的代码中使用`delete`和`destroy`方法，就会进行软删除操作

通过软删除的删除的数据，正常搜索是搜不到软删除的数据的

如果需要查到软删除的数据，需要用`withTradshed()`方法

```php
//使用withTrashed方法，可以查到所有的数据，包括软删除的
$users = User::withTrashed()->get();
```

如果只想搜到软删除的数据，需要用到`onlyTrashed()`方法

判断某些数据是否被软删除，需要用到`trashed()`方法

```php
$users = User::withTrashed()->find(7);
dd($users->trashed());
```

`restore()`方法：可以用来恢复软删除

```php
App\Flight::withTrashed()
    ->where('airline_id', 1)
    ->restore();
```

`forceDelete()`:强制删除软删除的数据

```php
$users = User::onlyTrashed()->find(7)->forceDelete();
```

##### 2.3 模型的作用域

- 本地作用域(局部作用域)：

  很多情况下，我们在搜索数据的时候，有一大部分的搜索条件是重复的，而且这些搜索条件只适用于某一张表，这种情况我们就可以用**本地作用域**的方式把这个条件封装且来

  在模型中设置本地作用域时，方法名必须要以`scope`开头，在具体逻辑代码中使用，就需要`scope`开头

  当然，使用本地作用域也可以传递参数,详情请看例子

  ```php
  //在user模型中使用本地作用域，查询年龄大于100的
  //在user模型中添加,注意！以scope开头
  public function scopeAge100($query) {
      $query -> where('age', '>=', 100);
  }
  //在业务代码中使用,不需要scope开头
  $users = User::age100()->get();
  dd($users->toArray());
  
  //传递参数的本地作用域,当然也可以传更多的参数
  public function scopeAge($query,$age = 100) {
      $query -> where('age', '>=', $age);
  }
  
  $users = User::age100(150)->get();
  dd($users->toArray());
  ```

  

- 全局作用域、

  全局作用域适用于所有模型的查询操作，**软删除**就是利用全局作用域来实现的

  全局作用域的跟本地作用域的区别是：**适用于所有的模型，而且不需要再显式的调用**

  全局作用域有两种实现方法，第一种是:

  ```php
  //1.写一个实现Illuminate\Database\Eloquent\Scope接口的类,并且实现apply方法
  <?php
  
  
  namespace App\Scope;
  
  
  use Illuminate\Database\Eloquent\Builder;
  use Illuminate\Database\Eloquent\Model;
  use Illuminate\Database\Eloquent\Scope;
  
  class AgeScope implements Scope
  {
  
      /**
       * @param Builder $builder
       * @param Model $model
       */
      public function apply(Builder $builder, Model $model)
      {
          // TODO: Implement apply() method.
          $builder -> where('age', '>=', 100);
      }
  }
  //2.在模型中添加方法来应用全局作用域,该方法是固定的 booted(),并且要使用addGlobalScope方法
  //在addGlobalScope方法中只需要new 一下刚才创建的类即可
  protected static function booted()
      {
          parent::booted(); // TODO: Change the autogenerated stub
          static::addGlobalScope(new AgeScope());
      }
  
  //如果想要取消全局作用域的话，需要在具体的查询中使用
  User::withoutGlobalScope(AgeScope::class)->get();
  ```

  第二种方法,匿名全局作用域：

  ```php
  //1.不需要创建scope类，只需要在模型的booted方法中使用匿名函数即可
  use Illuminate\Database\Eloquent\Builder;
  
  protected static function booted()
  {
      parent::booted(); // TODO: Change the autogenerated stub
      //第一个参数是给这个匿名函数取一个名字，主要是用作取消的时候
      static::addGlobalScope('age', function (Builder $builder){
          $builder->where('age', '>=', 100);
      });
  }
  
  //如果想要取消全局作用域的话，需要在具体的查询中使用
  User::withoutGlobalScope('age')->get();
  ```

##### 2.4 模型的访问器和修改器

- 访问器：

  **当我们要查询某个字段的时候，可能想要对这个字段做一些格式化的处理，比如：姓名需要加中括号，或者英文名称需要首字母大写等，这些情况就可以用到访问器**

  在模型中定义一个访问器，需要创建一个`getXxxAttribute()`的方法，中间的`Xxx`就是字段名且需要使用**驼峰式**命名法

  如果字段名是`firstname`这样的，`Xxx`为`Firstname`，如果是`first_name`，那么方法名是`FirstName`这样的

  ```php
  //访问器，需要注意传递参数，然后返回格式化后的参数
  public function getNameAttribute($name){
      return '【'.$name.'】';
  }
  //在具体的业务代码中访问name字段，就是格式化后的数据
  ```

  **虚拟访问器**：我们可以设置一个虚拟字段来对其他字段进行整合`getXxxAttribute()`,其中`xxx`就是需要追加的字段(首字母是小写的),不过要景行数据追加，也就是设置`protected $appends = [...]`

  ```php
  //设置数据追加
  protected $appends = ['name_age'];
  //虚拟访问器
  public function getNameAgeAttribute(){
      return $this->name.'-'.$this->age;
      //return $this->attributes['name'].'--'.$this->attributes['age'];
  }
  ```

  注意！！！如果其中的某些字段已经通过访问器格式化过，那么通过`return $this->name.'-'.$this->age;`拼接的是格式化后的数据

  如果想要原始数据，需要使用`return $this->attributes['name'].'--'.$this->attributes['age'];`

- 修改器：

  修改器的意思是：**在数据插入或者修改时对数据格式化后在存入数据库**

  具体的方法是：`setXxxAttribute()`，其字段的用法跟访问器类似，这里就不写了, 不过需要注意的是 设置字段只能用`$this->attributes['xxx']`这种方式，而且需要传递参数

  ```php
  public function setFirstNameAttribute($value)
  {
      $this->attributes['first_name'] = strtolower($value);
  }
  
  ```

- 转换器（不常用）

  日期转换器：就是把某些字段设置成日期格式，也就是说，在存入数据的时候，这个字段会以日期的格式写入(如果字段设置的不是日期类型，那么写入的数据会有问题)，展示的时候也会以日期的格式展示，是在模型中设置的

  ```php
  //就是在插入或者修改时，这个字段会以日期的格式插入，
  protected $dates = [
      'test_date',
  ];
  ```

  属性类型转换：使用`$casts`属性,会把字段的格式转换成设置的格式，支持转换的数据类型有： integer, real, float, double, decimal:<digits>, string, boolean, object, array, collection, date, datetime, 和 timestamp。 当需要转换为 decimal 类型时，你需要定义小数位的个数，如： decimal:2。
  
  ```php
  protected $casts = [
      'is_admin' => 'boolean',
  ];
  ```

##### 2.5 集合的使用

集合是什么：它是一种更具读取性和处理能力的数组封装

我们通过数据库得到的数据列表就是一种集合，当然我们也可以自己通过`collect`方法来创建集合

数据集合提供了很多强大的方法供我们进行各种操作，当没有我们想要的方法时，我们也可以自定义方法

常用方法有(一共有119个，其他的可以看文档)：

1. `all()`,把集合数据类型转换成数组类型，注意：如果通过`all()`只是把外层的数据类型转换成数组，所以并不能用来转换数据库数据列表产生的集合

2. `avg()`，返回平均值，还可以用来分组的平均值

   ```php
   //显示分组的平均值
   $average = collect([['foo' => 10,'a'=>20], ['foo' => 10,'a'=>20], ['foo' => 20,'a'=>20], ['foo' => 40,'a'=>20]])->avg('foo');
   dd($average);
   ```

3. `count()` 返回集合的数量

4. `countBy()` 计算集合中每个值出现的次数，该方法也可以传递一个回调函数来计算自定义的值出现的次数

   ```php
   //计算值出现的次数
   $collection = collect([1, 2, 2, 2, 3]);
   $counted = $collection->countBy();
   dd($counted->all());
   // [1 => 1, 2 => 3, 3 => 1]
   
   //自定义,计算各中类型的邮件出现的次数
   $collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);
   $counted = $collection->countBy(function ($value) {
       return substr(strrchr($value, "@"), 1);
   });
   dd($counted->all());
   // ['gmail.com' => 2, 'yahoo.com' => 1]
   ```

5. `diff()` 返回集合中不相同的部分,以第一个集合为准

   ```php
   $collection = collect([1, 2, 3, 4, 5]);
   $diff = $collection->diff([2, 4, 6, 8]);
   dd($diff->all());
   // [1, 3, 5]
   ```

6. `duplicates()` 返回重复的值

7. `first()`，返回集合中的第一个元素，如果传一个回调函数，返回指定条件成立后的第一条数据，返回的是具体数据，需要传一个匿名函数

   ```php
   collect([1, 2, 3, 4])->first(function ($value, $key) {
       return $value > 2;
   });
   // 3
   ```

8. `flatten()`将多维数组转换成一维数组

9. `get()`根据key找到具体的值

10. `has()`返回是否有具体的值

11. `pop()`弹出最后一个值

12. `slice()` 返回集合中给定索引开始后面的部分

13. `sort()`排序，需要链式请求`values()`方法来显示

14. `where()` 方法通过给定的键 / 值对查询过滤集合的结果 

    ```php
    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);
    
    $filtered = $collection->where('price', 100);
    
    $filtered->all();
    
    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */
    ```

15. `map()`类似于访问器，可以对集合内的数据进行格式化

    ```php
    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    });
    dd($collection->all());
    ```

16. ` macro ()`可以自定义一些集合的方法，最好是写在服务提供者内

17. `filter()` 使用给定的回调函数过滤集合，只保留那些通过指定条件测试的集合项 

    ```php
    $collection = collect([1, 2, 3, 4]);
    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });
    $filtered->all();
    // [3, 4]
    ```

18. `reject()`跟`filter()`相反, 使用指定的回调函数过滤集合。如果回调函数返回 true 就会把对应的集合项从集合中移除 

    ```php
    $collection = collect([1, 2, 3, 4]);
    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });
    $filtered->all();
    // [1, 2]
    ```

    

##### 2.6 模型的数据集合

在模型的数据集合中，有些方法的用法和集合有些不同，比如：`map()`

```php
//我们想要实现访问器的功能
$user = User::get();
//在集合的map方法,我们会直接返回修改后的数据，但在数据集合中，就需要给对应的属性重新赋值，然后返回整个对象
$collection = $user->map(function ($user){
    $user->email = strtoupper($user->email);
    return $user;
});
```

**还有一些其他的模型集合方法，可以查看文档 https://learnku.com/docs/laravel/7.x/eloquent-collections/7501#method-contains** 

##### 2.7 模型的一对一关联

> 关联的概念：
>
> 1.一对一：比如，一个学生只有一张个人信息卡，一个信息卡也只有一个学生，那么这就是一对一的概念
>
> 2.一对多：比如，一篇文章有多条评论，一条评论只能属于一篇文章，那么这就是一对多的关系
>
> 3.多对多：比如，一个用户可能有多个职称，反过来一个职称也可以有多个人，这就是多对多的关系
>
> 还有更多的....

既然是关联，当然也会有绑定的概念，当有数据库操作时，关联表也会跟着变动，这就是关联模型的意义

> 主键设置为id，关联主键默认就是id，可以默认不屑
>
> 附属表的外键设置为user_id，即：主表名_主键，吻合可默认不写

现在我们又两张表做一对一关联，`user`和`score`（这里score假设每个user都只有一个score）

首先要在user模型中添加一个用于关联的方法，方法名可以随意取，最好是关联表的名称，通过`hasOne()`方法进行一对一关联

```php
//hasOne方法，还可以传第二个参数和第三个参数
//第二个参数是关联表的外键id，默认的是主表明_主键
//第三个参数是主键名称
public function score(){
    return $this->hasOne(Score::class);
    //return $this->hasOne(Score::class, 'user_id', 'id');
}
```

这样我们就可以在业务逻辑中使用 Eloquent 的动态属性获得相关的记录 。

```php
//返回的是一个score对象
$data = User::find(1)->score;
```

**反向关联**：跟正向关联类似，只不过方法名变成了`belongsTo()`

```php
public function user(){
    return $this->belongsTo(User::class);
    //return $this->belongsTo(User::class,'user_id','id');
}
```

获取

```php
//返回的是user对象
$data = Score::find(1)->user;
```

##### 2.8 模型的一对多关联

现在我们又两张表做一对多关联，`user`和`book`（这里book假设每个user都有多本书）

在user模型中，通过`hasMany()`来表示一对多关系，参数和`hasOne()`相同

```php
//一对多关联 正向
public function book(){
    return $this->hasMany(Book::class);
}
```

这样我们就可以在业务逻辑中使用 Eloquent 的动态属性获得相关的记录 。

```php
//因为是一对多关系，返回的是一个集合
$data = User::find(1)->book;
dd($data);
//我们只想要其中的某一本书，那么可以这样写
$data = User::find(1)->book()->where('book_name','西游记')->get();
dd($data->toArray());
```

**一对多反向**:

在book模型中，跟正向关联类似，只不过方法名变成了`belongsTo()`

```php
public function user(){
    return $this->belongsTo(User::class);
}
```

获取

```php
$data = Book::find(1)->user;
dd($data);
```

##### 2.9 模型的多对多

现在我们有三张表来实现多对多关联，user表，group表(兴趣小组表),user_group表(关联表)，一个用户可以在多个小组中，一个小组可以有多个用户

在user模型中使用`belongsToMany`，需要注意的是框架默认的关系表是根据表名首字母顺序来排先后的，比如，user 和group ，其关联表默认是group_user，所以就需要自己指定关联表

 默认情况下，`pivot` 对象只包含两个关联模型的主键，如果你的中间表里还有其他额外字段，你必须在定义关联时明确指出 ，通过在`belongsToMany()`后面链式请求`withPivot()`

也可以在模型中，通过关联表的条件进行过滤,通过在`belongsToMany()`后面链式请求`wherePivot()`或者`wherePivotIn()`,一个是相等的意思，一个是in的意思

如果觉得`pivot`这个名称比较难记的话，可以通过在`belongsToMany()`后面链式请求`as('xxx')`的方式改变`pivot`属性的名称

```php
//多对多
public function group()
{
    //需要设置关联表的名称
    return $this->belongsToMany(Group::class,'user_group');
    //第三个参数是：对于group表的外键在关联表中的字段
    //第四个参数是：对于group表的主键在关联表中的字段
    //第五个参数是：关联表的主键id
    //return $this->belongsToMany(Group::class,'user_group','user_id','group_id','id');
    
   //获取关联表的其他字段
    return $this->belongsToMany(Group::class,'user_group')->withPivot('id');
    
    //通过关联表的条件进行过滤
    return $this->belongsToMany(Group::class,'user_group')->withPivot('id')->wherePivotIn('id',[1,2]);
}
```

**多对多反向**：



```php
public function user()
{
    return $this->belongsToMany(User::class,'user_group');
    //第三个参数是：对于group表的外键在关联表中的字段
    //第四个参数是：对于group表的主键在关联表中的字段
    //第五个参数是：关联表的主键id
    //return $this->belongsToMany(User::class,'user_group','group_id','user_id','id');
    
    
    //获取关联表的其他字段
    return $this->belongsToMany(User::class,'user_group')->withPivot('id');
}
```

获取跟其他的类似，但是会多一个`pivot`的字段，其下是`user_id`和`group_id`，表示其关联关系

```php
//如果要获取某一个小组的信息，有两种方法
//1.User::find(1)->group() 返回的是关联表的对象，然后再where，所以where中的条件是group_id,返回的数据既有group的数据，也有关联数据
$data = User::find(1)->group()->where('group_id',2)->get();
//2.User::find(1)->group 返回的一个集合，所以后续的where是集合的操作方法
$data = User::find(1)->group->where('group_name','数学小组');
```

##### 2.10 模型的关联查询(重要)

> 注意：通过调用属性返回的是一个集合，而不是数据集合，所以不能使用toSql()，如果想要看sql，就需要通过请求方法的方式，这样请求返回的是数据集合，后面可以链式请求任何数据集合或者查询构造器的方法

1. 通过请求方法的方式，这样请求返回的是数据集合，后面可以链式请求任何数据集合或者查询构造器的方法，当然，在where等能够写闭包的方法内，也可以传一个闭包

   ```php
   use Illuminate\Database\Eloquent\Builder;
   
   //最好在闭包参数前加上类型，这样在闭包中请求方法，可以使ide给出提示
   $data = User::find(1)->group()->where(function (Builder $query) {
       $query->where('group_id',1);
   })->dd();
   ```

2. `has()`，在上面关联模型中举例子中，有一个问题，都是通过`find()`确定到某一条数据，然后再根据关联模型找数据，如果我现在想通过一个条件来找到大于一条的数据，然后再通过关联模型找数据呢？`has()`就是解决这个问题的

   > 如果通过has查询的数据还想再获取另一个模型的数据，可以foreach 返回的数据集合，然后在各自的模型对象中调用另一个模型的方法或者属性

   ```php
   //我想找到所有选好兴趣小组的用户信息
   //打印出的sql:select * from `user` where exists (select * from `group` inner join `user_group` on `group`.`id` = `user_group`.`group_id` where `user`.`id` = `user_group`.`user_id`)
   //注意！！！！这里的group代表的是group模型类
   $data = User::has('group')->get();
   dd($data->toArray());
   
   //参加兴趣小组大于等于3个用户信息
   //打印的sql是：select * from `user` where (select count(*) from `group` inner join `user_group` on `group`.`id` = `user_group`.`group_id` where `user`.`id` = `user_group`.`user_id`) >= 3 
   $data = User::has('group','>=','3')->get();
   dd($data->toArray());
   ```

3. `whereHas`和`orWhereHas`方法，就相当于通过闭包来自定义has条件

   ```php
   $posts = App\Post::whereHas('comments', function (Builder $query) {
       $query->where('content', 'like', 'foo%');
   })->get();
   ```

4. `doesntHave()`和`has()`相反的操作

5. `withCount()`这个方法可以用来统计关联模型的某个字段某个值的数量，这个信息会以添加到返回结果中，其字段名称是`{关联模型的名称}_count`

   ```php
   //统计一个关联模型的数量
   $data = User::withCount('book')->get()->toArray();
   //统计多个关联模型的数量
   $data = User::withCount(['book','group'])->get()->toArray();
   //统计每个人书有几本，只统计user_id=1的参加了多少个兴趣小组
   $data = User::withCount(['book','group' => function (Builder $query) {
               $query->where('user_id', 1);
           }])->get()->toArray();
   ```

##### 2.11  DebugBar调试器

- 首先通过composer 安装 `composer require barryvdh/laravel-debugbar` 如果是只想在开发环境使用，后面要加上`--dev`

- 生成配置文件 `php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"`
- 写一个模板文件，在resource/view目录下创建一个模板文件，然后生成一段html代码
- 在`data`方法最开头 写 `return view('data');`，这个调试工具可以显示了
- 在controller中 需要 `use Barryvdh\Debugbar\Facade as Debugbar;` ，然后就可以用`Debugbar::info('aaaaa');`等方法了

```php
//具体用法，这样就可以子debugbar中显示，而且还能看到sql,路由，模板等等信息
$user = User::all();
Debugbar::info($user->toArray());
return view('data');
```

##### 2.12 模型的预加载

如果不使用模型的预加载会有什么问题？

```php
//如果我们想要获取每个用户book下的信息，这样就会查询N+1次，因为每遍历一次，就执行一条sql
$user = User::all();
foreach ($user as $item) {
    Debugbar::info($item->book);
}
```

 这种情况就需要使用`with()`来预加载模板，可以大大减少执行的sql数，预加载还有很多用法，可以看： https://learnku.com/docs/laravel/7.x/eloquent-relationships/7500#has-one-through 

```php
//这样就只有两条sql 
$user = User::with('book')->get();
foreach ($user as $item) {
    Debugbar::info($item->book);
}
```

**延迟预加载：**

有时， 有可能你还希望在模型加载完成后在进行渴求式加载。举例来说，**如果你想要根据某个条件动态决定是否加载关联数据** ， 那么 `load` 方法对你来说会非常有用 

##### 2.13 模型的关联插入和更新

### 3. 其他方面

##### 3.1 请求参数相关

一般情况都是通过注入`Request $request`来获取参数，也可以通过助手函数`request()`来获取参数，后续的方法都一样

##### 3.2 生成url/获取url

在工作中，我们有时可能需要获取当前的url或者是生成一个url，这时候就可以使用`url()`助手函数