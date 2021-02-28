# PageHelper的使用


# PageHelper

在实际项目开发中，常常需要使用到分页，分页的方式分为两种：

* 前端分页
* 后端分页

**前端分页：**

一次 `ajax` 请求数据的所有记录，然后在前端缓存并且计算 `count` 和分页逻辑，一般前端组件会提供分页动作。

特点：简单，很适合小规模的web平台；当数据量大的时候会产生性能问题，在查询和网络传输的时间会很长。

**后端分页：**

在 `ajax` 请求中指定页码 `pageNum` 和每页的大小 `pageSize` ，后端查询出当页的数据返回，前端只负责渲染。

特点：复杂一点；性能瓶颈来自于对数据库的压力。

### 不使用分页插件的分页操作

查询 count 的 select 语句，然后再写一个真正分页查询语句，MySQL中有对分页的支持，是通过limit子句

`limit`关键字的用法是：`LIMIT [offset], rows`

`offset`是相对首行的偏移量（首行是0），`rows`是返回条数

例如

```mysql
// 查询的是前5条记录
select * from tableA limit 0, 5;
// 查询的是第6条到第10条的记录
select * from tableA limit 5, 5
```

## MyBatis分页插件PageHelper使用

可在yml中配置：

* dialect：默认情况会使用PageHelper方式进行分页，如果想要实现自己的分页逻辑，可以实现 `Dialect`(`com.github.pagehelper.Dialect`) 接口，然后配置该属性为实现类的全限定名称。

```yml
pagehelper:
  auto-dialect: false
```

将自动配置关闭即可

使用自定义dialect实现时，下面参数没有任何作用：

1. `helperDialect`：分页插件会自动检测当前的数据库链接，自动选择合适的分页方式。
    你也可以实现 `AbstractHelperDialect`，然后配置该属性为实现类的全限定名称即可使用自定义的实现方法。
2. `offsetAsPageNum`：默认值为 `false`，该参数对使用 `RowBounds` 作为分页参数时有效。 当该参数设置为 `true` 时，会将 `RowBounds` 中的 `offset` 参数当成 `pageNum` 使用，可以用页码和页面大小两个参数进行分页。
3. `rowBoundsWithCount`：默认值为`false`，该参数对使用 `RowBounds` 作为分页参数时有效。 当该参数设置为`true`时，使用 `RowBounds` 分页会进行 count 查询。
4. `pageSizeZero`：默认值为 `false`，当该参数设置为 `true` 时，如果 `pageSize=0` 或者 `RowBounds.limit = 0` 就会查询出全部的结果（相当于没有执行分页查询，但是返回结果仍然是 `Page` 类型）。
5. `reasonable`：分页合理化参数，默认值为`false`。当该参数设置为 `true` 时，`pageNum<=0` 时会查询第一页， `pageNum>pages`（超过总数时），会查询最后一页。默认`false` 时，直接根据参数进行查询。
6. `params`：为了支持`startPage(Object params)`方法，增加了该参数来配置参数映射，用于从对象中根据属性名取值， 可以配置 `pageNum,pageSize,count,pageSizeZero,reasonable`，不配置映射的用默认值， 默认值为`pageNum=pageNum;pageSize=pageSize;count=countSql;reasonable=reasonable;pageSizeZero=pageSizeZero`。
7. `supportMethodsArguments`：支持通过 Mapper 接口参数来传递分页参数，默认值`false`，分页插件会从查询方法的参数值中，自动根据上面 `params` 配置的字段中取值，查找到合适的值时就会自动分页。 使用方法可以参考测试代码中的 `com.github.pagehelper.test.basic` 包下的 `ArgumentsMapTest` 和 `ArgumentsObjTest`。
8. `autoRuntimeDialect`：默认值为 `false`。设置为 `true` 时，允许在运行时根据多数据源自动识别对应方言的分页 （不支持自动选择`sqlserver2012`，只能使用`sqlserver`），用法和注意事项参考下面的**场景五**。
9. `closeConn`：默认值为 `true`。当使用运行时动态数据源或没有设置 `helperDialect` 属性自动获取数据库类型时，会自动获取一个数据库连接， 通过该属性来设置是否关闭获取的这个连接，默认`true`关闭，设置为 `false` 后，不会关闭获取的连接，这个参数的设置要根据自己选择的数据源来决定。
10. `aggregateFunctions`(5.1.5+)：默认为所有常见数据库的聚合函数，允许手动添加聚合函数（影响行数），所有以聚合函数开头的函数，在进行 count 转换时，会套一层。其他函数和列会被替换为 count(0)，其中count列可以自己配置。

**重要提示：**

当 `offsetAsPageNum=false` 的时候，由于 `PageNum` 问题，`RowBounds`查询的时候 `reasonable` 会强制为 `false`。使用 `PageHelper.startPage` 方法不受影响。

### PageHelper.startPage 静态方法调用

除了 `PageHelper.startPage` 方法外，还提供了类似用法的 `PageHelper.offsetPage` 方法。

在你需要进行分页的 MyBatis 查询方法前调用 `PageHelper.startPage` 静态方法即可，紧跟在这个方法后的第一个**MyBatis 查询方法**会被进行分页。

> //获取第1页，10条内容，默认查询总数
>
> count PageHelper.startPage(1, 10); 
>
> //紧跟着的第一个select方法会被分页 
>
> List<Country> list = countryMapper.selectIf(1); assertEquals(2, list.get(0).getId()); 
>
> assertEquals(10, list.size());
>
>  //分页时，实际返回的结果list类型是Page<E>，如果想取出分页信息，需要强制转换为Page<E> assertEquals(182, ((Page) list).getTotal());
>


