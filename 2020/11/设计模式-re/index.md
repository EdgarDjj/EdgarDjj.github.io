# 设计模式(re)


# 设计模式-re

## 创建者模式

**该模式提供创建对象的机制，能够提升已有代码的灵活性和复用性**

创建型模式包括：

1. 工厂方法 Factory Method
2. 抽象工厂 Abstract Factory
3. 建造者 Builder
4. 原型 Prototype
5. 单例 SingleTon

### 工厂方法模式

工厂方法主要是在父类中提供一个创建对象的方法，由子类决定实例化对象的类型。主要意图是定义一个创建对象的接口，由子类去自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。

### 抽象工厂模式

抽象工厂是一个中心工厂，创建其它的工厂模式。

总的来说，抽象工厂模式所要解决的问题就是在一个产品族，存在多个不同类型的产品（Redis集群、操作系统）情况下，接口选择的问题。

### 建造者模式

建造者模式所完成的内容就是通过将多个简单对象通过一步步的组装建造出一个复杂的对象的过程。

将一个复杂的构建与其相分离，使得同样的构建过程可以创建不同的表示。

#### 案例

1. java.lang.StringBuilder

Appendable 接口定义了多个append方法（抽象方法），即Appendable为抽象建造者，定义了抽象方法，AbstractStringBuilder已经是建造者，只是不能实例化，StringBuilder即充当了指挥者角色，同时充当了具体点的建造者，建造方法的实现是由AbstractStringBuilder完成，而StringBuilder继承了AbstractStringBuilder

### 原型模式

主要解决的问题就是创建重复对象，而这部分对象内容比较复杂，生成过程可能从库或者RPC接口中获取数据的耗时较长，因此采用克隆的方式节省时间。

原型模式的优点在于便于通过克隆方式创建复杂对象，也可以避免重复做初始化操作，不需要与类中所属的其它类耦合等，但也有一些缺点，如果对象中包括了循环引用的克隆，以及深度使用对象的克隆，都会使此对象变的异常复杂。

#### 案例

例如考试出卷，有两种类型题：问答题和选择题

想要让问题进行随机，如果不使用原型，很难完成拓展和随机的使用。

而使用原型模式的最重要的手段就是克隆，在需要用到的类都需要实现 `implements Cloneable` 接口。

Spring bean.xml 中的配置 scope：prototype

与单例不同的是，实例出来的对象不是同一个对象了，Bean工厂默认配置是单例。、

#### 深拷贝和浅拷贝

**浅拷贝：**对于基本数据类型的成员变量，浅拷贝会直接进行值传递，即将该属性值复制一份给新的对象。对于引用数据类型，浅拷贝会进行引用传递，将该成员变量的引用值（内存地址）复制一份给新的对象。浅拷贝是使用默认的clone（）方法来实现的

**深拷贝：**复制对象的所有基本数据类型的成员变量。为所有引用数据类型的成员变量申请存储空间，直到该对象可达所有对象。也就是说，对象进行深拷贝要对整个对象进行拷贝。重写clone方法来实现深拷贝，通过对象序列化实现深拷贝。

### 单例模式

保证一个类只有一个实例哪怕再多线程访问也要提供一个全局访问此实例的点。单例模式主要解决的是，一个全局使用的类频繁的创建和消费，从而提升整体代码的性能。

保证了系统中只存在一个对象，节省了系统资源。

想要实例化一个单例类的时候，必须记住使用相应获取对象的方法而不是new。

8个实现方法：

**饿汉模式-2**

懒汉模式-3

**静态内部类**

**双重锁检测**

**枚举**

#### 案例

1. 数据库的连接池不会反复创建
2. spring中一个单例模式bean的生成和使用
3. 在我们平常的代码中需要设置全局的一些属性保存

4. Runtime源码中，使用了饿汉模式。
5. 工具类、Session工厂的等等

## 结构型模式

**该模式介绍如何将对象和类组装成较大的结构，并同时保持结构的灵活和高效**

结构型模式包括：

1. 适配器模式 Adapter
2. 桥接 Bridge
3. 组合 Composite
4. 装饰器 Decorator
5. 外观 Facade
6. 享元 Flyweight
7. 代理

### 适配器模式

一个简答的例子：3.5mm耳机接口无法在iphone8以上的手机类型上使用，因此需要3.5mm耳机口转接lighting口。而适配器模式的主要作用也是如此，把原本不兼容的接口，通过适配器修改做到统一。

适配器模式解决的主要问题就是多种差异化类型的接口做统一输出。

### 桥接模式

> 为什么你的代码那么多ifelse

桥接模式的主要作用就是通过抽象部分与实现部分分离，把多种可匹配的使用进行组合。**说白了核心实现就是在A类中含有B类接口，通过构造函数传递B类实现，这个B类就是设计的桥**

像JDBC多种驱动程序的实现、同品牌类型的台式机和笔记本平板、业务实现中的多类接⼝同组过滤服务等。这些场景都⽐较适合使⽤桥接模式进⾏实现，因为在⼀些组合中如果有如果每⼀个类都实现不同的 服务可能会出现笛卡尔积，⽽使⽤桥接模式就可以⾮常简单。

桥接模式的关键是选择桥接拆分点，是否可以找到这样类似的相互组合。

#### 案例

例如：支付app与支付方式的桥接

我们需要判断

* 微信支付
  * 人脸支付
  * 指纹支付
  * 密码支付
* 支付宝支付
  * 人脸支付
  * 指纹支付
  * 密码支付

因此产生了重复方式的判断。

```java
System.out.println("\r\n模拟测试场景；微信支付、人脸方式。");
Pay wxPay = new WxPay(new PayFaceMode());
wxPay.transfer("weixin_1092033111", "100000109893", new BigDecimal(100));

System.out.println("\r\n模拟测试场景；支付宝支付、指纹方式。");
Pay zfbPay = new ZfbPay(new PayFingerprintMode());
zfbPay.transfer("jlu19dlxo111","100000109894",new BigDecimal(100));
```

### 组合模式

通过吧相似的对象组合成一组可被调用的结构树对象的设计思路。

这种设计⽅式可以让你的服务组节点进⾏⾃由组合对外提供服务，例如你有三个原⼦校验功能( A：身份 证 、 B：银⾏卡 、 C：⼿机号 )服务并对外提供调⽤使⽤。有些调⽤⽅需要使⽤AB组合，有些调⽤⽅需要 使⽤到CBA组合，还有⼀些可能只使⽤三者中的⼀个。那么这个时候你就可以使⽤组合模式进⾏构建服 务，对于不同类型的调⽤⽅配置不同的组织关系树，⽽这个树结构你可以配置到数据库中也可以不断的 通过图形界⾯来控制树结构。

#### 案例

一些决策树问题，例如优惠券的分发，如果购物不多，则对其会经常增加促销力度，增加用户粘性。和ifelse的作用，但方便以后的拓展和修改。

### 装饰器模式

装饰器的核心就是不再修改原有类的基础上给新类新增功能，**不改变原有类**。例如继承、AOP切面都可以实现，但装饰器模式能避免继承子类过多的情况，避免AOP带来的复杂性。

### 外观模式

主要解决的是降低调用方的使用接口的复杂逻辑组合。这样调用方与实际的接口提供方提供了一个中间件层，对服务中的通用性复杂逻辑进行中间件包装，让使用方只关心业务开发。

这样的模式十分常见，例如以前注册一个网站的时候需要补全很多信息，但现在一步就可以注册，需要的信息在要使用到的时候再让用户进行补充。

### 享元模式

享元模式主要在于共享通用对象，减少内存的使用，提升系统的访问资源效率。而这部分共享对象通常比较占用内存或需要大量的接口或者使用数据库资源，因此统一抽离作为共享对象使用。

在享元模式下，需要享元工厂来进行管理这部分独立的对象和共享的对象，避免出现线程安全的问题。

#### 案例

例如数据库连接池的使用、线程池的使用等等

### 代理模式

代理模式类似于分销商，主要解决某些资源的访问、对象的类的易用操作上提供方便使用的代理服务。。⽽这种设计思想的模式经常会出现在我们的系统中，或者你⽤到过 的组件中，它们都提供给你⼀种⾮常简单易⽤的⽅式控制原本你需要编写很多代码的进⾏使⽤的服务类。

类似这样的场景可以想到；

1. 你的数据库访问层⾯经常会提供⼀个较为基础的应⽤，以此来减少应⽤服务扩容时不⾄于数据库连 接数暴增。
2. 使⽤过的⼀些中间件例如；RPC框架，在拿到jar包对接⼝的描述后，中间件会在服务启动的时候⽣ 成对应的代理类，当调⽤接⼝的时候，实际是通过代理类发出的socket信息进⾏通过。
3. 另外像我们常⽤的 MyBatis ，基本是定义接⼝但是不需要写实现类，就可以对 xml 或者⾃定义注 解⾥的 sql 语句进⾏增删改查操作。

## 行为型模式

**该模式负责对象间的高效沟通和职责委派**

行为型模式包括：

1. 责任链 Chain of Responsesiability
2. 命令 Command
3. 迭代器 Iterator
4. 中介者 Mediator
5. 备忘录 Memento
6. 观察者 Observer
7. 状态 State
8. 策略 Strategy
9. 模板 Template
10. 访问者 Visitor

### 责任链模式

责任链模式的核心就是解决一组服务中的先后执行处理关系，例如学校请假，普通的请假班长审批辅导员审批，长假就需要班长辅导员院长审批。

### 命令模式

命令模式相对于互联网开发来说相对教授，但日常中很常见，就是cmd+c和cmd+v，这就是把逻辑实现与操作请求进行分离，降低耦合方面方便拓展。

命令方式以数据驱动的方式命令对象，可以使用构造函数的方式传递给调用者。调用者再提供相应的实现命令执行提供操作方法。

1. 抽象命令类：声明执行命令的接口和方法
2. 具体的命令实现类：接口类的具体实现，可以是一组相似的行为逻辑
3. 实现者：也就是为命令做实现的具体实现类
4. 调用者：处理命令，实现的具体操作者，负责对外提供命令服务

### 迭代器模式

迭代器模式，常见的就是日常使用的iterator遍历。虽然在实际业务开发中不常见，但每天都要使用JDK提供的list集合遍历。

> 注意：增强for循环不是迭代器模式，迭代器模式的特点是实现Iterable接口，通过next方式获取集合，同时具备删除等操作。而增强for循环可以做到。

这种设计模式的优点就是让我以相同的方式可以去遍历不同的数据结构元素。

#### 迭代器模式的遍历组织结构

1. Collection，集合方法部分对于自定义的数据结构添加通用方法：add、remove、Iterator等核心方法
2. Iterable，提供获取迭代器，这个接口类会被Collection继承
3. Iterator，提供两个方法定义：hasNext、next会在具体的数据结构中写实现方法

### 中介者模式

中介者模式要解决的就是复杂功能之间的重复调用，在这中间添加一层中介者包装服务，对外提供简单、通用、易拓展的服务能力。

例如ORM框架——MyBatis等

### 备忘录模式

备忘录模式是以可以恢复或者说回滚、配置、版本、悔棋为核心功能的设计模式。在功能上是以不破坏原对象为基础增加备忘录操作类，记录原对象的行为从而实现备忘录模式。

如果cmd+z，返回上一个操作，游戏回档等。

备忘录的设计模式实现⽅式，重点在于不更改原有类的基础上，增加备忘录类存放记录。可能平时虽然 不⼀定⾮得按照这个设计模式的代码结构来实现⾃⼰的需求，但是对于功能上可能也完成过类似的功 能，记录系统的信息。

### 观察者模式

简单来说，就是当一个行为发生时传递消息给另外一个用户接收做出相应的处理，两者之间没有直接的耦合关联。例如经常使用的MQ服务，虽然MQ服务是有个一个通知中心并不是每个类进行服务，但整体上也算是观察者模式的设计思路。

即一个对象发生改变的时候自动通知其他对象，其他对象将相应做出反应。在此，发生改变的对象称为观察目标，而被通知的对象称为观察者。

### 状态模式

状态模式描述的是⼀个⾏为下的多种状态变更，⽐如我们最常⻅见的⼀个⽹站的⻚页⾯，在你登录与不登录 下展示的内容是略有差异的( 不登录不能展示个⼈信息 )，⽽这种 登录 与 不登录 就是我们通过改变状态， ⽽让整个⾏为发⽣了变化。

一个对象的行为取决于一个或多个动态变化的属性，这样的属性叫做状态，这样的对象叫做有状态的(stateful)对象，这样的对象状态是从事先定义好的一系列值中取出的。当一个这样的对象与外部事件产生互动时，其内部状态就会改变，从而使得系统的行为也随之发生变化。

### 策略模式

策略模式是⼀种⾏为模式，也是替代⼤量ifelse的利器。它所能帮你解决的是场景，⼀般是具有同类可替代的⾏为逻辑算法场景。⽐如；不同类型的交易⽅式(信⽤卡、⽀付宝、微信)、⽣成唯⼀ID策略 (UUID、DB⾃增、DB+Redis、雪花算法、Leaf算法)等，都可以使⽤策略模式进⾏⾏为包装，供给外部 使⽤。定义一系列算法，将每一个算法封装起来，并让它们可以相互替换。策略模式让算法独立于使用它的客户而变化。

- 完成一项任务，往往可以有多种不同的方式，每一种方式称为一个策略，我们可以根据环境或者条件的不同选择不同的策略来完成该项任务。
- 在软件开发中也常常遇到类似的情况，实现某一个功能有多个途径，此时可以使用一种设计模式来使得系统可以灵活地选择解决途径，也能够方便地增加新的解决途径。

### 模板模式

模板模式的核⼼设计思路是通过在，抽象类中定义抽象⽅法的执⾏顺序，并将抽象⽅法设定为只有⼦类 实现，但不设计 独⽴访问 的⽅法。简单说也就是把你安排的明明⽩⽩的。

就像⻄西游记的99⼋⼗⼀难，基本每⼀关都是；师傅被掳⾛、打妖怪、妖怪被收⾛，具体什么妖怪你⾃⼰ 定义，怎么打你想办法，最后收⾛还是弄死看你本事，我只定义执⾏顺序和基本策略，具体的每⼀难由 观⾳来安排。

即你的人生从出生起就被安排好了如何过完，出生-》死亡，但你的经历随便你。

### 访问者模式

访问者要解决的核⼼事项是，在⼀个稳定的数据结构下，例如⽤户信息、雇员信息等，增加易变的业务 访问逻辑。为了增强扩展性，将这两部分的业务解耦的⼀种设计模式。

说⽩了访问者模式的核⼼在于同⼀个事物不同视⻆角下的访问信息不同。


