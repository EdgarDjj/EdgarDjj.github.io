# RESTfulAPI简明


# RESTfulAPI简明

**RESTful API 可以看到url + http method 就知道这个url是干什么的，让你看到啦http状态码（status code）就知道请求结果如何。**

是一种架构的风格。

## 重要的概念

REST，即REpresentational State Transfer的缩写。实际全称：**Resource Representational State Tranfe**“资源”在网络传输中以某种“表现形式”进行“状态转移“。

* **资源（Resource）：**真实的对象数据（资源）。一个资源可以是一个集合，也可以是单个个体。
* **表现形式（Representational）：**”资源“是一种信息实体，它可以有多种外在表现形式。
* **状态转移（State Transfer）：**通过增删改查引起的资源状态的改变。ps：互联网通信协议HTTP协议，是一个无状态协议，所有资源都保存在服务端。

RESTful架构：

1. 每一个URL代表一种资源；
2. 客户端和服务端之间，传递这种资源的某种表现形式：json、xml、image、txt等
3. 客户端通过特定的HTTP动词，对服务器端资源进行操作，实现”表现层状态转换“

## REST接口规范

1. #### 动作

* GET：请求从服务器获取特定资源
* POST：在服务器创建一个新的资源
* PUT：更新服务器的资源（客户端提供更新后的整个资源）
* DELETE：从服务器删除特定的资源
* PATCH：更新服务器上的资源（客户端提供的更新的属性，可以看做是部分更新）

2. #### 路径（接口命名）

路径又称”终点“（endpoint），表示API的具体网址。实际开发中的规范如下：

* **网址中不能有动词，只能有名词。API中的名词也应该是使用复数。**因为 REST 中的资源往往和数据库中的表对应，而数据库中的表都是同种记录的"集合"（collection）。**如果 API 调用并不涉及资源（如计算，翻译等操作）的话，可以用动词。** 比如：`GET /calculate?param1=11&param2=33`
* 不用大写字母，建议用中杠 - 不用下杠 _ 比如邀请码写成 `invitation-code`而不是 ~~invitation_code~~

**接口尽量使用名词，禁止使用动词。** 下面是一些例子：

```
GET    /classes：列出所有班级
POST   /classes：新建一个班级
GET    /classes/classId：获取某个指定班级的信息
PUT    /classes/classId：更新某个指定班级的信息（一般倾向整体更新）
PATCH  /classes/classId：更新某个指定班级的信息（一般倾向部分更新）
DELETE /classes/classId：删除某个班级
GET    /classes/classId/teachers：列出某个指定班级的所有老师的信息
GET    /classes/classId/students：列出某个指定班级的所有学生的信息
DELETE classes/classId/teachers/ID：删除某个指定班级下的指定的老师的信息
```

反例：

```
/getAllclasses
/createNewclass
/deleteAllActiveclasses
```

理清资源的层次结构，比如业务针对的范围是学校，那么学校会是一级资源:`/schools`，老师: `/schools/teachers`，学生: `/schools/students` 就是二级资源。

3. #### 过滤信息（Filtering）

如果我们在查询的时候需要添加特定条件的话，建议使用 url 参数的形式。比如我们要查询 state 状态为 active 并且 name 为 guidegege 的班级：

```
GET    /classes?state=active&name=guidegege
```

比如我们要实现分页查询：

```
GET    /classes?page=1&size=10 //指定第1页，每页10个数据
```

4. #### 状态码（Status Codes）

| 2xx：成功 | 3xx：重定向    | 4xx：客户端错误  | 5xx：服务器错误 |
| --------- | -------------- | ---------------- | --------------- |
| 200 成功  | 301 永久重定向 | 400 错误请求     | 500 服务器错误  |
| 201 创建  | 304 资源未修改 | 401 未授权       | 502 网关错误    |
|           |                | 403 禁止访问     | 504 网关超时    |
|           |                | 404 未找到       |                 |
|           |                | 405 请求方法不对 |                 |


