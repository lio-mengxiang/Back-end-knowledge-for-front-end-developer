大家好，我是孟祥同学，本小册主要介绍 mysql 入门知识，参考了大量付费课程、书籍和 mysql 技术文档，并结合笔者本身使用经验结合而成，帮助大家快速入门 mysql。<br />强烈，强烈，强烈建议，对于初学者，一定要自己敲一遍代码，无论是从本章找到的示例代码或者网上你看的视频、纸质教程，不动手，纯看，效果减半。<br />mysql 更深入的内容，详见【前端的后端体系小册】mysql 高级篇 和 【前端的后端体系小册】mysql 面试题篇 。如果要学习整个【前端的后端体系】（还在更新中，微信交流群请加：a2298613245）系列，详见[我的 github](https://github.com/lio-mengxiang)，会实时更新。<br />注： mysql 使用版本是 8.0+ 。

目录：

[TOC]

# Mysql 学习环境搭建 - docker

使用 docker 很容易的就能启动一个 mysql 服务，所以我们选用 docker 来搭建学习环境。以下是具体步骤（确保已经安装 docker）：

- 拉取 docker 镜像

```
docker pull mysql:latest
```

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052596287-b4c2537c-1bd9-406f-853f-5cff5ae621dc.png#averageHue=%23f0f0ef&clientId=u589f0ee3-d396-4&from=paste&id=u620346e7&originHeight=90&originWidth=1060&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uafcc32cd-9555-48b4-b2c9-9d84f5be88a&title=)

- 启动 mysql docker 容器

```
docker run -itd --name mysql-test2 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```

- 进入 mysql docker 容器

```
docker exec -it mysql-test2 bash
```

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052596311-5a809f00-10c8-4df7-a832-7e57ce0ea74a.png#averageHue=%23d4d4d3&clientId=u589f0ee3-d396-4&from=paste&id=u1f1bb3cc&originHeight=86&originWidth=1018&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u1fe01b41-877f-45dd-9464-b436e99d29c&title=)<br />这些 docker 命令具体什么含义，请看我的【前端的后端体系小册】docker 篇。

# 小试牛刀 - 简单使用一下 mysql

好的，现在 MySQL 已经装到了我们的计算机上，在我们使用 mysql 之前， 我们非常有必要从宏观上了解一下这个软件是怎样运行的。<br />MySQL 采用了客户端/服务器（C/S）架构，这是一种常见的数据库架构模式。web 开发中，浏览器和后端服务也是一种典型的 C/S 架构，浏览器是客户端（client），后端服务是服务端（service）。如下图：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052596323-dad1cb2b-0aa6-40f1-8b5b-a36c51031729.png#averageHue=%23fafafa&clientId=u589f0ee3-d396-4&from=paste&id=u8ad09825&originHeight=398&originWidth=1067&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u1304f0e0-f00f-40ed-a138-c1a5457d8e7&title=)<br />上图的 mysqld 就是 mysql 服务端的服务名，客户端通过 mysql 命令来连接 server 端。所以我们来连接一下 mysql

```
mysql -uroot -p123456
```

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052596405-8c803b89-57b7-4719-8782-e1c07b957309.png#averageHue=%23f3f3f2&clientId=u589f0ee3-d396-4&from=paste&id=u22ae50ff&originHeight=430&originWidth=1124&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u3505fad0-fa9c-4499-b969-448fa6ac424&title=)<br />如果加上了空白字符就是错误的，比如这样：

```
mysql -u root -p 123456
```

各个参数的意义如下：

| **参数名** | **含义**                                                                                                                                                                            |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -h         | 表示启动服务器程序的计算机的域名或者 IP 地址，如果服务器程序就运行在本机的话，可以省略这个参数，也可以填[localhost](http://localhost)或者 127.0.0.1。也可以写作 --host=主机的形式。 |
| -u         | 表示用户名，我们刚刚安装完，作为超级管理员的我们的用户名是 root。也可以写作 --user=用户名的形式。                                                                                   |
| -p         | 表示密码。也可以写作 --password=密码的形式。                                                                                                                                        |

如果我们想断开客户端与服务器的连接并且关闭客户端的话，可以在 mysql>提示符后输入下边任意一个命令：

1. quit
2. exit
3. \q

比如我们输入 quit 试试：

```
mysql> quit
Bye
```

我们简单查询一下 mysql 的版本：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052596334-0f1c4da4-59e0-420c-beea-d452bf9b8a20.png#averageHue=%23f6f6f5&clientId=u589f0ee3-d396-4&from=paste&id=u263b876b&originHeight=252&originWidth=526&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u98ab48ad-0795-4bf0-a295-f69b98273d3&title=)<br />以及看看我们当前的 mysql 里有什么哪些数据库：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052597408-e8622598-3944-42d8-a2f2-b6440fe56c02.png#averageHue=%23f3f3f2&clientId=u589f0ee3-d396-4&from=paste&id=u383a97ab&originHeight=308&originWidth=500&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uf9476acb-ab69-4c57-acd7-47032447d50&title=)<br />以下是对各个数据库的说明：

1. information_schema：这个数据库包含了关于 MySQL 服务器上所有数据库、表、列和其他数据库对象的元数据信息。它提供了访问和查询数据库结构的系统视图，例如 TABLES、COLUMNS、SCHEMATA 等。通过查询 information_schema 数据库，可以获取有关数据库、表、列、索引、用户权限等方面的详细信息。
2. performance_schema：这个数据库提供了 MySQL 服务器的性能监控和诊断功能。它包含了各种性能相关的数据表和视图，可以用于分析数据库服务器的性能瓶颈、查询执行统计、锁定情况、I/O 活动等。通过查询 performance_schema 数据库，可以获取有关 MySQL 服务器性能方面的详细信息。
3. sys：sys 是一个 MySQL 5.7 版本及以上引入的数据库，它是对 information_schema 和 performance_schema 的高级封装。sys 数据库提供了更简洁和易用的视图和存储过程，用于监控和诊断 MySQL 服务器的性能。它为开发人员和管理员提供了更直观和方便的方式来获取和分析性能数据。
4. mysql：mysql 数据库是 MySQL 服务器的系统数据库，它包含了用户账户、权限信息和其他系统级配置数据。它存储了 MySQL 服务器的用户认证信息，例如用户名、密码和访问权限。通过查询 mysql 数据库，可以管理用户账户、授权、修改服务器配置等。

# 关系型数据库和非关系型数据库的关系

这一章的目的是帮助我们在选择数据库类型时有依据。<br />首先，最最关键的一点，如果你们的业务对数据的安全性和一致性要求很高，此时肯定是要选择关系型数据库的。还有如果对事务支持很好，非关系型数据库只能支持基本的事务功能。<br />其二，关系型数据库是结构化存储，例如下图，在关系型数据库中，每一张表都有严格的约束信息：字段名、字段数据类型、字段约束等等信息，插入的数据必须遵守这些约束<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052597430-98960633-5b28-4fe7-b139-9139db4b221c.png#averageHue=%23d3d2d2&clientId=u589f0ee3-d396-4&from=paste&id=u77c634a1&originHeight=372&originWidth=707&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u4cb04773-aea4-40a3-ba1c-32d9c473d90&title=)<br />而非关系型数据库，存储形式就比较自由了，比如 redis 以是键值型存储数据，如下图，value 可以是 json 字符串，这样存储起来：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052597474-6cb7c3b2-3306-47ae-b795-a25da298efe6.png#averageHue=%23baaca1&clientId=u589f0ee3-d396-4&from=paste&id=u3ef154ba&originHeight=244&originWidth=267&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u8b32a93c-7ef1-473f-b2fb-b89c4571fb4&title=)<br />也可以是文档型，例如 mongodb，如下图，存储的字段也是没有限制的，比如你第一个数据是{ id: 1， name: '张三'}，第二条数据可以是{ id: 2， name: '李四'， age: 20 }，多了一个 age 字段。<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052597484-576a91bb-ab3a-40bc-9359-42ac4139536f.png#averageHue=%23baada1&clientId=u589f0ee3-d396-4&from=paste&id=ub0fa23fa&originHeight=238&originWidth=265&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u42225ab8-763d-4400-a9b0-bb1934ffda9&title=)<br />其三，关联和非关联，传统数据库的表与表之间往往存在关联，如下图：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052597736-65b960a3-5cde-4f3c-a3b1-f6e4797f0c22.png#averageHue=%23c6d0d3&clientId=u589f0ee3-d396-4&from=paste&id=uc3a03156&originHeight=405&originWidth=757&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ub348377a-c3b5-42c3-a8f3-7d2c5b87c52&title=)<br />例如有一个 user 表，存储用户信息，有一个产品表存储产品，也就是手机信息的表，此时如果要查询订单数据，也就是第三章表，order 表，我们就需要把用户和产品表用彼此的 id 关联在一起，也就是表与表之间往往是有关联的。<br />而非关系型数据库不存在关联关系，要维护关系要么靠代码中的业务逻辑，要么靠数据之间的耦合，如下，张三的订单有两条：

```
{
  id: 1,
  name: "张三",
  orders: [
    {
       id: 1,
       item: {
     id: 10, title: "荣耀6", price: 4999
       }
    },
    {
       id: 2,
       item: {
     id: 20, title: "小米11", price: 3999
       }
    }
  ]
}
```

此时的问题在于，如果李四也买了小米 11，那么李四也会有跟张三类似的产品信息，并没有把产品信息抽象出来，这样就产生了耦合，也产生了冗余的数据。<br />其四，查询方式，关系数据库为代表的 sql 数据库，是有着严格的 sql 查询标准的，而非关系数据库查询语法则。

# 数据类型

mysql 在建表的时候，需要描述你存储的字段是哪种类型，比如名字是字符串类型，年龄是数值类型。

## 数值类型

### MySQL 的整数类型

很显然，使用的字节数越多，意味着能表示的数值范围就越大，但是也就越耗费存储空间。根据表示一个数占用字节数的不同，MySQL 把整数划分成如下所示的类型：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052598031-23e8bb19-785a-4926-a36c-4bf3e08e0c91.png#averageHue=%23f6f6f6&clientId=u589f0ee3-d396-4&from=paste&id=u3d56778d&originHeight=516&originWidth=1964&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u271b7fd7-c2c2-40d0-b2b4-b67eebe961f&title=)<br />以 TINYINT 为例，用 1 个字节，也就是 8 个位表示有符号数的话，就是既可以表示正数，也可以表示负数的话，需要有一个比特位表示正负号。但是如果表示无符号数的话，也就是只表示非负数的话，就不需要表示正负号，这是有符号数和无符号数的区别。

### 整数类型的可选属性介绍

- M

M: 表示显示宽度，M 的取值范围是（0， 255），比如 int（M），表示宽度为 M 的 int 类型。例如，int（5）：当数据宽度小于 5 位的时候在数字前面需要用字符填满宽度。该项功能需要配合“ZEROFILL`”使用，表示用“0”填满宽度，否则指定显示宽度无效。<br />我们举个例子：

```
CREATE TABLE users (
    id INT(5) ZEROFILL,
    name VARCHAR(50)
);
```

在上面的示例中，我们创建了一个名为 users 的表，其中包含两个列：id 和 name。id 列被定义为 INT(5) ZEROFILL，这意味着它是一个带有宽度为 5 的整数，并且使用 ZEROFILL 属性。<br />现在，当我们向 users 表中插入数据时，MySQL 会自动在 id 列中填充零，以保持宽度为 5。例如：

```
INSERT INTO users (id, name) VALUES (1, 'John');
```

在执行上述插入语句后，id 列的值将为 00001，而不是普通的 1。这确保了在查询和显示数据时，数字的宽度保持一致。

- 无符号类型（UNSIGNED）： UNSIGNED 是 MySQL 中用于表示无符号类型（非负数）的属性。所有整数类型都可以选择添加 UNSIGNED 属性，无符号整数类型的最小取值为 0。因此，如果需要在 MySQL 数据库中保存非负整数值，可以将整数类型设置为无符号类型。
- 显示宽度和默认宽度：对于 int 类型，默认的显示宽度为 int(11)，而无符号的 int 类型默认的显示宽度为 int(10)。
- 0 填充（ZEROFILL）： ZEROFILL 表示在 MySQL 中使用零填充。如果某一列指定了 ZEROFILL，那么 MySQL 会自动为该列添加 UNSIGNED 属性。使用 ZEROFILL 只是表示当数值的位数不够时，在左侧用零进行填充，如果超过指定的位数，只要不超过数据存储范围即可。

### 浮点类型

- FLOAT 表示单精度浮点数；
- DOUBLE 表示双精度浮点数；

在 javascript 中，数字都是 DOUBLE 类型的浮点数，浮点数有个非常明显的特点就是很容易在 10 进制和 2 进制转换中丢失精度。至于为什么，我们必须要了解一下如何将 10 进制转换为 2 进制，而且 10 进制整数转换为 2 进制和 10 进制小数转换为 2 进制的方法还不一样。

- 十进制整数转为任意进制

方法是除商取余法：比如 10 进制转 2 进制<br />例如： 把 89 化为二进制的数<br />把 89 化为二进制的数<br />89÷2=44 余 1<br />44÷2=22 余 0<br />22÷2=11 余 0<br />11÷2=5 余 1<br />5÷2=2 余 1<br />2÷2=1 余 0<br />1÷2=0 余 1<br />然后把余数由下往上排序<br />1011001<br />这样就把 89 化为二进制的数了<br />![](https://cdn.nlark.com/yuque/0/2023/webp/432603/1703052598082-29a5024c-7b1a-4ffc-8832-8cd9b29d39ab.webp#averageHue=%23f19f79&clientId=u589f0ee3-d396-4&from=paste&id=u9e8cc09c&originHeight=438&originWidth=440&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u65adfa0e-1aec-4fcc-a1f5-0226135f89d&title=)

- 十进制小数转为 n 进制
  - 我们还是以 2 进制为例，方式是采用“乘 2 取整，顺序排列”法。具体做法是：
    - 用 2 乘十进制小数，可以得到积，将积的整数部分取出-
    - 再用 2 乘余下的小数部分，又得到一个积，再将积的整数部分取出-
    - 如此进行，直到积中的小数部分为零，或者达到所要求的精度为止

**所以 n 进制是一个道理**<br />我们具体举一个例子<br />如： 十进制 0.25 转为二进制

- 0.25 \* 2 = 0.5 取出整数部分：0
- 0.5 \* 2 = 1.0 取出整数部分 1

即十进制 0.25 的二进制为 0.01 （ 第一次所得到为最高位，最后一次得到为最低位）<br />此时我们可以试试十进制 0.1 和 0.2 如何转为二进制，就知道为啥 0.1 + 0.2 不等于 0.3 了

```
0.1(十进制) = 0.0001100110011001(二进制)
十进制数0.1转二进制计算过程：
0.1*2＝0.2……0——整数部分为“0”。整数部分“0”清零后为“0”，用“0.2”接着计算。
0.2*2＝0.4……0——整数部分为“0”。整数部分“0”清零后为“0”，用“0.4”接着计算。
0.4*2＝0.8……0——整数部分为“0”。整数部分“0”清零后为“0”，用“0.8”接着计算。
0.8*2＝1.6……1——整数部分为“1”。整数部分“1”清零后为“0”，用“0.6”接着计算。
0.6*2＝1.2……1——整数部分为“1”。整数部分“1”清零后为“0”，用“0.2”接着计算。
0.2*2＝0.4……0——整数部分为“0”。整数部分“0”清零后为“0”，用“0.4”接着计算。
0.4*2＝0.8……0——整数部分为“0”。整数部分“0”清零后为“0”，用“0.8”接着计算。
0.8*2＝1.6……1——整数部分为“1”。整数部分“1”清零后为“0”，用“0.6”接着计算。
0.6*2＝1.2……1——整数部分为“1”。整数部分“1”清零后为“0”，用“0.2”接着计算。
0.2*2＝0.4……0——整数部分为“0”。整数部分“0”清零后为“0”，用“0.4”接着计算。
0.4*2＝0.8……0——整数部分为“0”。整数部分“0”清零后为“0”，用“0.2”接着计算。
0.8*2＝1.6……1——整数部分为“1”。整数部分“1”清零后为“0”，用“0.2”接着计算。
……
……
所以，得到的整数依次是：“0”，“0”，“0”，“1”，“1”，“0”，“0”，“1”，“1”，“0”，“0”，“1”……。
由此，大家肯定能看出来，整数部分出现了无限循环。
```

接下来看 0.2

```
0.2化二进制是
0.2*2=0.4,整数位为0
0.4*2=0.8,整数位为0
0.8*2=1.6,整数位为1,去掉整数位得0.6
0.6*2=1.2,整数位为1,去掉整数位得0.2
0.2*2=0.4,整数位为0
0.4*2=0.8.整数位为0
就这样推下去！小数*2整,一直下去就行
最后得到
0.0011001
```

所以我们可以看到，如果你对计算要求十分精确的话，不应该使用浮点数。<br />对于浮点数的讲解就不深入了，有兴趣可以去看我的【前端的后端体系小册】计算机组成原理。

#### 设置最大位数和小数位数

在定义浮点数类型时，还可以在 FLOAT 或者 DOUBLE 后边跟上两个参数，就像这样：

```
FLOAT(M, D)
DOUBLE(M, D)
```

对于用户而言，通常使用的是十进制小数。如果我们事先知道表中的某个列要存储的小数范围，我们可以使用 FLOAT(M, D) 或者 DOUBLE(M, D) 来限制该列中可存储的小数范围。具体说明如下：

- M 表示该小数最多需要的十进制有效数字个数。 注意，这里的有效数字个数是指小数的位数，例如对于小数 -1.1 来说，有效数字个数为 2；对于小数 0.2 来说，有效数字个数为 1。
- D 表示该小数的小数点后的十进制数字个数。 简而言之，D 的值就等于小数点后的十进制数字个数。

举个例子看一下，设置了 M 和 D 的单精度浮点数的取值范围的变化：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052598127-e89c9efa-5a08-4327-ae07-a72ad526d01c.png#averageHue=%23fbfafa&clientId=u589f0ee3-d396-4&from=paste&id=u43631f5a&originHeight=764&originWidth=616&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u9ec44f4c-8575-4d94-977a-dc5b06fc25a&title=)<br />可以观察到，在相同的 D 值下，M 越大，该类型的取值范围就越大；而在相同的 M 值下，D 越大，该类型的取值范围就越小。<br />浮点数类型的取值范围为 M（1 ～ 255）和 D（1 ～ 30，而且 D 的值必须不大于 M），分别表示显示宽度和小数位数。M 和 D 在 FLOAT 和 DOUBLE 中是可选的，FLOAT 和 DOUBLE 类型将被保存为硬件所支持的最大精度。<br />下表中列出了 MySQL 中的小数类型和存储需求。<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052598122-8b918392-c5c8-4c9f-a9b0-0aaaf92edf65.png#averageHue=%23f0f0f0&clientId=u589f0ee3-d396-4&from=paste&id=ucd0669c3&originHeight=240&originWidth=932&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u2ddc8190-ad83-4108-a660-3ea924fa566&title=)<br />无论是否显式设置了精度（M， D），MySQL 在处理方案上如下：

- 如果存储时整数部分超出范围，MySQL 将报错，不允许存储这样的值。
- 如果存储时小数部分超出范围，将有以下情况：
  - 如果经过四舍五入后，整数部分未超出范围，MySQL 将发出警告，但仍能成功操作并四舍五入删除多余的小数位后保存。例如，在 FLOAT（5，2） 列中插入 999.009，近似结果将是 999.01。
  - 如果经过四舍五入后，整数部分超出范围，MySQL 将报错并拒绝处理。例如，在 FLOAT（5，2） 列中插入 999.995 或 -999.995 都会报错。

从 MySQL 8.0.17 开始，官方文档已明确不推荐使用 FLOAT（M， D） 和 DOUBLE（M， D） 的用法，并且将来可能会被移除。此外，关于浮点型 FLOAT 和 DOUBLE 的 UNSIGNED 也不推荐使用，将来也可能会被移除。

### 定点数类型

由于浮点数在表示小数时可能有时候不精确，为了解决这个问题，MySQL 的设计者们引入了一种称为定点数（DECIMAL）的数据类型，它是另一种存储小数的方式。<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052598460-3cfde2a3-e884-432c-9f70-dedc9cd87f79.png#averageHue=%23f7f7f6&clientId=u589f0ee3-d396-4&from=paste&id=u04ee849a&originHeight=270&originWidth=1022&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u60d8b133-adbb-444c-a441-d73d7cd49ee&title=)<br />在上面的语法中：

- M 是表示有效数字数的精度。 M 范围为 1〜65。M 的默认值是 10。
- D 是表示小数点后的位数。 D 的范围是 0 ～ 30。MySQL 要求 D 小于或等于（<=）P。
- DECIMAL(P，D)表示列可以存储 M 位小数的 P 位数。十进制列的实际范围取决于精度和刻度。
- DECIMAL 类型也具有 UNSIGNED 和 ZEROFILL 属性。 如果使用 UNSIGNED 属性，则 DECIMAL UNSIGNED 的列将不接受负值。

如果使用 ZEROFILL，MySQL 将把显示值填充到 0 以显示由列定义指定的宽度。 另外，如果我们对 DECIMAL 列使用 ZEROFILL，MySQL 将自动将 UNSIGNED 属性添加到列。<br />这里有一个很重要的问题要讨论，就是我们上面已经论证过了，10 进制和 2 进制在小数层面转换的时候，往往会有精度误差，定点数是如何做到没有精度误差呢？

#### 定点数如何避免进制转化时的精度误差

实际上，小数只是将两个十进制整数用小数点分隔开而已。我们只需要将小数点左边和右边的两个十进制整数分别存储起来，是不是精度就不丢失了（整数的转换不会丢失精度的）。<br />举个例子，对于十进制小数 1.12，我们可以将小数点左边的整数 1 和小数点右边的整数 12 分别保存起来。<br />我们看看 mysql 到底是如何存储 decimal 的。<br />MySQL 分别为整数和小数部分分配存储空间。 MySQL 使用二进制格式存储 DECIMAL 值。它将 9 位数字包装成 4 个字节。<br />比如 DEMCIMAL(16, 4)来说：

- 首先 整数保存 16 - 4 = 12 个十进制，小数保存 4 个十进制
- 从小数点位置出发，每个整数每隔 9 个十进制位划分为 1 组，效果就是这样：

![](https://cdn.nlark.com/yuque/0/2023/webp/432603/1703052598633-367d1686-c4fd-4fee-bae0-d7f1e2ece1ff.webp#averageHue=%23f7f2ef&clientId=u589f0ee3-d396-4&from=paste&id=udae1a9e7&originHeight=402&originWidth=1397&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u606efeae-8773-4360-a592-a2f6643a71f&title=)

- 从图中可以看出，如果不足 9 个十进制位，也会被划分成一组。
- **组中包含的十进制位数不同，占据存储空间也不同，具体见下表：**

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052598680-46d57401-7e00-4c60-9933-1b4cb24416e6.png#averageHue=%23fcfcfb&clientId=u589f0ee3-d396-4&from=paste&id=u7966aaa4&originHeight=444&originWidth=1486&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u3c68d986-203b-4380-9c4a-6ee97fe06dc&title=)<br />所以 DECIMAL(16, 4)共需要占用 8（2+4+2）个字节的存储空间大小。

## 日期和时间类型

Mysql 中有多处表示日期的数据类型：**YEAR**、**TIME**、**DATE**、**DTAETIME**、**TIMESTAMP**。当只记录年信息的时候，可以只使用 YEAR 类型。 每一个类型都有合法的取值范围，当指定确定不合法的值时，系统将“零”值插入数据库中。 下表中列出了 MySQL 中的日期与时间类型。<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052598718-9f82cbd2-5b55-4189-b87c-b1e9d1ab41cb.png#averageHue=%23fcfcfb&clientId=u589f0ee3-d396-4&from=paste&id=u8f033a6c&originHeight=736&originWidth=1388&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u14d17b66-4f19-47ce-85a7-3aafc3a67e5&title=)<br />在 MySQL5.6.4 这个版本之后，TIME、DATETIME、TIMESTAMP 这几种类型添加了对毫秒、微秒的支持。由于毫秒、微秒都不到 1 秒，所以也被称为小数秒，MySQL 最多支持 6 位小数秒的精度，详情如下：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052598820-acbacf93-9da2-497f-a1d7-fafce27df05a.png#averageHue=%23fcfbfb&clientId=u589f0ee3-d396-4&from=paste&id=u5fd9fc12&originHeight=772&originWidth=1428&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u58259627-2ec0-4a7a-b947-d778e2d56d9&title=)<br />例如：比如 DATETIME(0)表示精确到秒，DATETIME(3)表示精确到毫秒，DATETIME(5)表示精确到 10 微秒。可以看到，如果仅仅精确到秒，DATETIME 只需要 5 个字节。

## YEAR 类型

YEAR 类型是一个单字节类型，用于表示年，在存储时只需要 1 个字节。可以使用各种格式指定 YEAR，如下所示：

- 以 4 位字符串或者 4 位数字格式表示的 YEAR，范围为 '1901'～'2155'。输入格式为 'YYYY' 或者 YYYY，例如，输入 '2010' 或 2010，插入数据库的值均为 2010。
- 以 4 位字符串或数字格式表示 YEAR 类型，其格式为 YYYY，最小值为 1901，最大值为 2155。

注：**从 MySQL5.5.27 开始，2 位格式的 YEAR 已经不推荐使用**。从 MySQL 8.0.19 开始，不推荐使用指定显示宽度的 YEAR（4）数据类型。

## TIME 类型

TIME 类型用于只需要时间信息的值，在存储时需要 3 个字节。格式为 HH:MM:SS。HH 表示小时，MM 表示分钟，SS 表示秒。<br />TIME 类型的取值范围为 -838：59：59 ～ 838：59：59，小时部分如此大的原因是 TIME 类型不仅可以用于表示一天的时间（必须小于 24 小时），还可能是某个事件过去的时间或两个事件之间的时间间隔（可大于 24 小时，或者甚至为负）。<br />可以使用各种格式指定 TIME 值，我们就不用记了，多此一举。

- 使用 CURRENT_TIME()或者 NOW()，会插入当前系统的时间。

## DATE 类型

DATE 类型用于仅需要日期值时，没有时间部分，在存储时需要 3 个字节。日期格式为 'YYYY-MM-DD'，其中 YYYY 表示年，MM 表示月，DD 表示日。<br />在给 DATE 类型的字段赋值时，可以使用字符串类型或者数字类型的数据插入，只要符合 DATE 的日期格式即可。如下所示：

- 以 'YYYY-MM-DD' 或者 'YYYYMMDD' 字符中格式表示的日期，取值范围为 '1000-01-01'～'9999-12-3'。例如，输入 '2015-12-31' 或者 '20151231'，插入数据库的日期为 2015-12-31。
- 不建议使用 YY-MM-DD 格式。
- 使用 CURRENT_DATE 或者 NOW（），插入当前系统日期。

## DATETIME 类型

DATETIME 类型用于需要同时包含日期和时间信息的值，在存储时需要 8 个字节。日期格式为 'YYYY-MM-DD HH：MM：SS'，其中 YYYY 表示年，MM 表示月，DD 表示日，HH 表示小时，MM 表示分钟，SS 表示秒。

- 使用函数 CURRENT_TIMESTAMP()和 NOW()，可以向 DATETIME 类型的字段插入系统的当前日期和时间。

## TIMESTAMP 类型

TIMESTAMP 的显示格式与 DATETIME 相同，显示宽度固定在 19 个字符，日期格式为 YYYY-MM-DD HH：MM：SS，在存储时需要 4 个字节。但是 TIMESTAMP 列的取值范围小于 DATETIME 的取值范围，为 '1970-01-01 00：00：01'UTC ～'2038-01-19 03：14：07'UTC。在插入数据时，要保证在合法的取值范围内。<br />提示：协调世界时（英：Coordinated Universal Time，法：Temps Universel Coordonné）又称为世界统一时间、世界标准时间、国际协调时间。英文（CUT）和法文（TUC）的缩写不同，作为妥协，简称 UTC。<br />TIMESTAMP 与 DATETIME 除了存储字节和支持的范围不同外，还有一个最大的区别是：

- DATETIME 在存储日期数据时，按实际输入的格式存储，即输入什么就存储什么，与时区无关；
- 而 TIMESTAMP 值的存储是以 UTC（世界标准时间）格式保存的，存储时对当前时区进行转换，检索时再转换回当前时区。即查询时，根据当前时区的不同，显示的时间值是不同的。

总体而言，因为 TIMESTAMP 类型时间戳范围只能到 2038，所以一般情况下可以存储 DateTime 类型，如果想加上时区，可以存储 UTC 时间，真正转换的时候，让前端转换即可。

## 字符串类型

讲字符串类型之前，我们需要具备字符串编码的基础知识。所以我先普及一下基本的编码类型：

### 字符串编码知识

#### ASCII

最开始计算机只在美国用，八位的字节可以组合出 256 种不同状态。0-32 种状态规定了特殊用途，一旦终端、打印机遇上约定好的这些字节被传过来时，就要做一些约定的动作，如：

- 遇上 0×10， 终端就换行；
- 遇上 0×07， 终端就向人们嘟嘟叫；

又把所有的空格、标点符号、数字、大小写字母分别用连续的字节状态表示，一直编到了第 127 号，这样计算机就可以用不同字节来存储英语的文字了<br />这 128 个符号（包括 32 个不能打印出来的控制符号），只占用了一个字节的后面 7 位，最前面的一位统一规定为 0<br />![](https://cdn.nlark.com/yuque/0/2023/jpeg/432603/1703052599123-25e9c315-6e79-4f07-9f30-8b29660aa3dc.jpeg#averageHue=%23e3e48a&clientId=u589f0ee3-d396-4&from=paste&id=uedbc1a0c&originHeight=494&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u2d475e2c-7105-48c8-a1ef-f1b189163c3&title=)<br />这个方案叫做 ASCII 编码<br />American Standard Code for Information Interchange：美国信息互换标准代码

#### GB2312

后来西欧一些国家用的不是英文，它们的字母在 ASCII 里没有为了可以保存他们的文字，他们使用 127 号这后的空位来保存新的字母，一直编到了最后一位 255。比如法语中的 é 的编码为 130。当然了不同国家表示的符号也不一样，比如，130 在法语编码中代表了 é，在希伯来语编码中却代表了字母 Gimel （ג）。<br />从 128 到 255 这一页的字符集被称为扩展字符集。<br />中国为了表示汉字，把 127 号之后的符号取消了，规定

- 一个小于 127 的字符的意义与原来相同，但两个大于 127 的字符连在一起时，就表示一个汉字；
- 前面的一个字节（他称之为高字节）从 0xA1 用到 0xF7，后面一个字节（低字节）从 0xA1 到 0xFE；
- 这样我们就可以组合出大约 7000 多个（247-161）\*（254-161）=（7998）简体汉字了。
- 还把数学符号、日文假名和 ASCII 里原来就有的数字、标点和字母都重新编成两个字长的编码。这就是全角字符，127 以下那些就叫半角字符。
- 把这种汉字方案叫做 GB2312。GB2312 是对 ASCII 的中文扩展

#### GBK

后来还是不够用，于是干脆不再要求低字节一定是 127 号之后的内码，只要第一个字节是大于 127 就固定表示这是一个汉字的开始，又增加了近 20000 个新的汉字（包括繁体字）和符号。

#### Unicode

ISO 的国际组织废了所有的地区性编码方案，重新搞一个包括了地球上所有文化、所有字母和符 的编码！ Unicode 当然是一个很大的集合，现在的规模可以容纳 100 多万个符号。

- International Organization for Standardization：国际标准化组织。
- Universal Multiple-Octet Coded Character Set，简称 UCS，俗称 Unicode

ISO 就直接规定必须用两个字节，也就是 16 位来统一表示所有的字符，对于 ASCII 里的那些 半角字符，Unicode 保持其原编码不变，只是将其长度由原来的 8 位扩展为 16 位，而其他文化和语言的字符则全部重新统一编码。<br />从 Unicode 开始，无论是半角的英文字母，还是全角的汉字，它们都是统一的一个字符！同时，也都是统一的 两个字节

- 字节是一个 8 位的物理存贮单元，
- 而字符则是一个文化相关的符号。

#### UTF-8

Unicode 在很长一段时间内无法推广，直到互联网的出现，为解决 Unicode 如何在网络上传输的问题，于是面向传输的众多 UTF 标准出现了，<br />Universal Character Set（UCS）Transfer Format：UTF 编码

- UTF-8 就是在互联网上使用最广的一种 Unicode 的实现方式
- UTF-8 就是每次以 8 个位为单位传输数据
- 而 UTF-16 就是每次 16 个位
- UTF-8 最大的一个特点，就是它是一种变长的编码方式，_UTF-8 使用一至四个字节为每个字符编码_

#### MySQL 中的 utf8 和 utf8mb4

我们上边说 utf8 字符集表示一个字符需要使用 1 ～ 4 个字节，但是我们常用的一些字符使用 1 ～ 3 个字节就可以表示了。而在 MySQL 中字符集表示一个字符所用最大字节长度在某些方面会影响系统的存储和性能，所以设计 MySQL 里的 utf8 并不是我们上面说的 utf8，而是 utf8mb3

- utf8mb3：阉割过的 utf8 字符集，只使用 1 ～ 3 个字节表示字符。
- utf8mb4：上面所说的 utf8 字符集，使用 1 ～ 4 个字节表示字符。

所以当你需要存储一些 emoji 表情的时候，utf8mb3 是无法满足需求的，请使用 utf8mb4。

### 字符串类型介绍

如下图：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052599207-8a17e958-7493-43c5-8010-1f7ee4f02480.png#averageHue=%23ebebeb&clientId=u589f0ee3-d396-4&from=paste&id=u0f7e7bd5&originHeight=410&originWidth=1205&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u6681881e-b7e5-44ae-b409-4ae302aef46&title=)

#### CHAR

- CHAR（M）类型通常需要预先定义字符串的长度。如果未指定（M），则默认长度为 1 个字符。
- 在 CHAR（M）中，M 表示该类型可以存储的最大字符数量。请注意，这里的 M 是字符数量，而不是字节数量。M 的取值范围为 0 ～ 255。
- 如果保存的数据实际长度小于 CHAR 类型声明的长度，将会在右侧填充空格以达到指定的长度。当使用 MySQL 检索 CHAR 类型的数据时，字段尾部的空格将被移除。
- 定义 CHAR 类型字段时，声明的字段长度即为该类型字段所占的存储空间的字节数。

在不同的字符集下，CHAR（M）所需的存储空间也会有所不同。假设某个字符集编码一个字符最多需要 W 个字节，那么 CHAR（M）类型占用的存储空间大小将为 M×W 个字节。举例说明：

- 对于采用 ASCII 字符集的 CHAR（5）类型来说，ASCII 字符集编码一个字符最多需要 1 个字节，即 M=5、W=1。因此，在这种情况下，该类型占用的存储空间大小为 5×1=5 个字节。
- 对于采用 UTF-8 字符集的 CHAR（5）类型来说，UTF-8 字符集编码一个字符最多需要 3 个字节，即 M=5、W=3。

#### VARCHAR

- VARCHAR（M） 定义时，必须指定长度 M，否则报错。
- MySQL4.0 版本以下，varchar（20）：指的是 20 字节，如果存放 UTF8 汉字时，只能存 6 个（每个汉字 3 字节）MySQL5.0 版本以上，varchar（20）：指的是 20 字符。虽然 VARCHAR(M)中的 M 也是代表该类型最多可以存储的字符数量，理论上的取值范围是 1~65535。但是 MySQL 中还有一个规定，表中某一行包含的所有列中存储的数据大小总共不得超过 65535 个字节（注意是字节），也就是说 VARCHAR(M)类型实际能够容纳的字符数量是小于 65535 的。
- 检索 VARCHAR 类型的字段数据时，会保留数据尾部的空格。VARCHAR 类型的字段所占用的存储空间为字符串实际长度加 1 个字节。

所以我们举一个例子，"王二狗"这三个字，如果使用 mysql 的 utf8mb3 存储，一个中文占 3 个字节，那么"王二狗"一共占据 3\*3 = 9 个字节（其实 VARCHAR 还要存储这个数据占据的字节数量，我们这里略过不提了，不了解也没什么），此时如果要比较是否超过类型字节数的上限，就要跟 65535 比较。

#### **CHAR 或 VARCHAR 比较**

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052599514-1cbfb7fb-88a7-4a38-95c2-8290bb9c0411.png#averageHue=%23f4f5f5&clientId=u589f0ee3-d396-4&from=paste&id=u1984633f&originHeight=316&originWidth=1984&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uea522978-b9b6-423e-a8ba-066bdbca18a&title=)

- 对于频繁改变的列，VARCHAR 需要每次存储都进行计算，包括长度等操作。如果列经常改变，那么使用 VARCHAR 会消耗大量的计算资源，而 CHAR 则不需要这些计算。
- 根据具体的存储引擎来选择：
  - 对于 MyISAM 存储引擎和数据列，最好使用固定长度（CHAR）的数据列代替可变长度（VARCHAR）的数据列。这样可以使整个表静态化，提高数据检索速度，用空间换取时间。
  - 对于 MEMORY 存储引擎和数据列，无论使用 CHAR 还是 VARCHAR 列都没有区别，因为它们都被处理为 CHAR 类型。
  - 对于 InnoDB 存储引擎，建议使用 VARCHAR 类型。因为 InnoDB 的内部行存储格式不区分固定长度和可变长度列，性能的关键是数据行使用的存储总量。由于 CHAR 平均占用的空间多于 VARCHAR，除非是简短且固定长度的信息，否则更推荐使用 VARCHAR 来节省空间，减少磁盘 I/O 和数据存储总量。

### TEXT 类型

- 在 MySQL 中，TEXT 用来保存文本类型的字符串，总共包含 4 种类型，分别为 TINYTEXT、TEXT、MEDIUMTEXT 和 LONGTEXT 类型。
- 在向 TEXT 类型的字段保存和查询数据时，系统自动按照实际长度存储，不需要预先定义长度。这一点和 VARCHAR 类型相同。

详细信息如下：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052599541-ed90c458-cc26-449b-af9e-7c682971494f.png#averageHue=%23f5f5f5&clientId=u589f0ee3-d396-4&from=paste&id=u2d422c86&originHeight=450&originWidth=1988&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ub2eb6579-737f-4857-86d3-0b689e0808f&title=)

- **由于实际存储的长度不确定，MySQL 不允许 TEXT 类型的字段做主键**。遇到这种情况，你只能采用 CHAR（M），或者 VARCHAR（M）。
- TEXT 文本类型，可以存比较大的文本段，搜索速度稍慢，因此如果不是特别大的内容，建议使用 CHAR，VARCHAR 来代替。还有 TEXT 类型不用加默认值，加了也没用。而且 text 和 blob 类型的数据删除后容易导致“空洞”，使得文件碎片比较多，所以频繁使用的表不建议包含 TEXT 类型字段，建议单独分出去，单独用一个表。

### ENUM 类型

- ENUM 类型也叫作枚举类型，ENUM 类型的取值范围需要在定义字段时进行指定。
- 相当于前端的单选框，只能设置一个值。

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052599603-62672964-cf34-4a39-9754-c62b698076e5.png#averageHue=%23f6f6f7&clientId=u589f0ee3-d396-4&from=paste&id=uac082fa0&originHeight=224&originWidth=2002&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uc13dd90b-408d-4661-84bf-1b0accd8f0e&title=)

- 当 ENUM 类型包含 1 ～ 255 个成员时，需要 1 个字节的存储空间；
- 当 ENUM 类型包含 256 ～ 65535 个成员时，需要 2 个字节的存储空间。
- ENUM 类型的成员个数的上限为 65535 个。

### SET 类型

SET 表示一个字符串对象，可以包含 0 个或多个成员，但成员个数的上限为 64。设置字段值时，可以取取值范围内的 0 个或多个值。相当于前端的多选框，可以选择多个值。

- 当 SET 类型包含的成员个数不同时，其所占用的存储空间也是不同的，具体如下：

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052599970-0b7bdda5-aeaf-4671-9c45-e3df106c0f24.png#averageHue=%23f9f9f9&clientId=u589f0ee3-d396-4&from=paste&id=udea596b6&originHeight=532&originWidth=1958&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ub602e53a-64b0-4143-8243-4658212fc0d&title=)

### BINARY（M）与 VARBINARY（M）

- BINARY（M）和 VARBINARY（M）与之前提到的 CHAR（M）和 VARCHAR（M）相对应。它们都是数据类型，其中 BINARY（M）和 VARBINARY（M）用于存储字节，而 CHAR（M）和 VARCHAR（M）用于存储字符。
- BINARY （M）为固定长度的二进制字符串，M 表示最多能存储的字节数，取值范围是 0 ～ 255。如果未指定（M），表示只能存储 1 个字节。例如 BINARY （8），表示最多能存储 8 个字节，如果字段值不足（M）个字节，将在右边填充’0’以补齐指定长度。

### 总结

关于字符串的选择，建议参考如下阿里巴巴的《Java 开发手册》规范：

- 任何字段如果为非负数，必须是 UNSIGNED
- 【强制】小数类型为 DECIMAL，禁止使用 FLOAT 和 DOUBLE。

说明：在存储的时候，FLOAT 和 DOUBLE 都存在精度损失的问题，很可能在比较值的时候，得到不正确的结果。如果存储的数据范围超过 DECIMAL 的范围，建议将数据拆成整数和小数并分开存储。

- 【强制】如果存储的字符串长度几乎相等，使用 CHAR 定长字符串类型。
- 【强制】VARCHAR 是可变长字符串，不预先分配存储空间，长度不要超过 5000。如果存储长度大于此值，定义字段类型为 TEXT，独立出来一张表，用主键来对应，避免影响其它字段索引效率。

# SOL 语句的分类和特点

初学的时候我也不重视这个分类和特点，也很正常，只有后面接触的内容多了，在回过头看这个分类才会觉得比较重要。<br />SQL 语言共分为四大类

- 数据查询语言（DQL）
- 数据操纵语言（DML）
- 数据定义语言（DDL）
- 数据控制语言（DCL）

大家看，这些都是 DxL，只有中间的 x 不一样，D 指的是 Data，数据的意思，L 是指 Language，语言。

## DQL

所以我们先看 DQL，Q 指的是查询，Query<br />我们常见的什么 SELECT，FORM，WHERE 都是查询表的，我们可以简单理解为就是 DQL 语句，例如，我们查询 customers 表中有什么数据

```
SELECT * FROM customers;
```

## DDL

中间的 D 指的是 Definition，定义的意思，我们简单理解为就是创建库创建表的语言

- CREATE 创建

```
-- 创建customers表
CREATE TABLE customers (
  id INT PRIMARY KEY,
  name VARCHAR(50),
  email VARCHAR(100)
);
```

- ALTER 修改

```
-- 修改表结构，增加一个 address 列
ALTER TABLE customers ADD COLUMN address VARCHAR(200);
```

- 删除 customers 表

```
DROP TABLE customers;
```

## DML

M 是 manipulation，操纵的意思，我们简单理解为对数据表里的每一条数据：添加更新和删除操作

- INSERT INTO：插入数据

```
INSERT INTO customers (id, name, email) VALUES (1, 'John Doe', 'john@example.com');
```

- UPDATE：更新数据

```
UPDATE customers SET email = 'newemail@example.com' WHERE id = 1;
```

- DELETE：删除数据

```
DELETE FROM customers WHERE id = 1;
```

## DCL

C 是 control 的意思，简单理解就是 mysql 里的事务。

```
COMMIT  提交
ROLLBACK   回滚
SET TRANSACTION  设置当前事务的特性，它对后面的事务没有影响
```

## 注意事项

- SQL 可以写在一行或者多行。为了提高可读性，各子句分行写，必要时使用缩进
- 每条命令以 ； 或 \g 或 \G 结束
- 关键字不能被缩写也不能分行
- 必须使用英文状态下的半角输入方式
- 字符串型和日期时间类型的数据可以使用单引号（’ '）表示
- 列的别名，尽量使用双引号（" "），而且不建议省略 as
- MySQL 在 Windows 环境下是大小写不敏感的
- MySQL 在 Linux 环境下是大小写敏感的
  - 数据库名、表名、表的别名、变量名是严格区分大小写的
  - 关键字、函数名、列名（或字段名）、列的别名（字段的别名） 是忽略大小写的。
- 可以使用如下格式的注释结构
  - 单行注释：#注释文字（MySQL 特有的方式）
  - 单行注释：-- 注释文字（--后面必须包含一个空格。)
  - 多行注释：/_ 注释文字 _/
- 数据导入指令

在命令行客户端登录 mysql，使用 source 指令导入

# 数据库操作

MySQL 包含很多数据库，数据库中有很多表，关系如下图：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052599966-f1ef4833-4ede-4b94-94f3-31ebbfe31944.png#averageHue=%23ecf1fa&clientId=u589f0ee3-d396-4&from=paste&id=u55f3f816&originHeight=718&originWidth=1266&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u326ef973-2fbc-4ac5-bb3f-7ad35e9bb2b&title=)<br />接下来我们就看看一些针对数据库的操作：

## 创建数据库

方式 1：创建数据库

```
CREATE DATABASE 数据库名;
```

方式 2：创建数据库并指定字符集

```
CREATE DATABASE 数据库名 CHARACTER SET 字符集 COLLATE 排序规则;
```

方式 3：判断数据库是否已经存在，不存在则创建数据库（推荐）

```
-- 如果MySQL中已经存在相关的数据库，则忽略创建语句，不再创建数据库。
CREATE DATABASE IF NOT EXISTS 数据库名;
```

注：DATABASE 不能改名<br />我们这里解释一下字符集和排序规则。

#### 指定字符集的方式创建数据库

5.7 的时候，mysql 默认字符集简单来说只支持英文，所以一般都要把字符集设置为 utf8，支持所有的语言。mysql 8 是默认 utf8mb4，我们肯定用 8 啊，5.7 人家官方都放弃维护了。<br />这里简单说一下 utf8 和 utf8mb4 的区别：<br />UTF-8 和 UTF8MB4 是 MySQL 中的两种字符集。

- UTF-8：这是一种常见的 Unicode 字符集，可以表示大多数字符，但不能表示某些特殊的字符（如 Emoji 字符）。
- UTF8MB4：这是一种扩展的 UTF-8 字符集，可以表示所有的 Unicode 字符，包括 Emoji 字符。

当您需要存储更多字符，特别是 Emoji 字符时，建议使用 UTF8MB4 字符集。在使用 UTF8MB4 字符集时，您需要使用 4 个字节来存储一个字符，而使用 UTF-8 字符集则只需要使用 3 个字节。因此，如果您的数据存储需求中需要存储更多字符，则使用 UTF8MB4 字符集可能会带来更多的空间开销。<br />语法：

```
CREATE DATABASE [IF NOTEXISTS] 数据库名 CHARACTER 字符集
```

对比 js 其实就是面向对象 new 一个新的数据库，然后传参字符集是啥

#### 排序规则

我们拿 utf8 字符集指定的 utf8_general_ci 和 utf8_bin 对数据排序的规则来说明：

- utf8_general_ci 是指对大小写不敏感，a 和 A 会在字符集里判断一致。
- utf8_bin 则要区分

语法：

```
CREATE DATABASE [IF NOTEXISTS] 数据库名 CHARACTER 字符集 COLLATE utf8_bin
```

## 查看当前所有的数据库

```
-- 有一个S，代表多个数据库
SHOW DATABASES;
```

## 查看当前正在使用的数据库

```
-- 使用的一个 mysql 中的全局函数
SELECT DATABASE();
```

## 查看指定库下所有的表

```
SHOW TABLES FROM 数据库名;
```

## 查看数据库的创建信息

```
SHOW CREATE DATABASE 数据库名;
```

## 使用/切换数据库

```
USE 数据库名;
```

## 修改数据库

更改数据库字符集

```
ALTER DATABASE 数据库名 CHARACTER SET 字符集;  #比如：gbk、utf8等
```

## 删除数据库

```
DROP DATABASE IF EXISTS 数据库名;
```

# DDL（表的基本操作）

数据定义语言 DDL 用来创建数据库中的各种对象，例如创建表，我们看下创建的一些规则：

- 数据库名、表名不得超过 30 个字符，变量名限制为 29 个。
- 只能包含 A–Z， a–z， 0–9， \_ 共 63 个字符。
- 数据库名、表名、字段名等对象名中间不要包含空格。
- 同一个 MySQL 软件中，数据库不能同名；同一个库中，表不能重名；同一个表中，字段不能重名。
- 必须确保字段没有与保留字、数据库系统或常用方法冲突。如果坚持使用，请在 SQL 语句中使用 `（着重号）引起来。
- 保持字段名和类型的一致性：在命名字段并为其指定数据类型时，一定要保持一致性。例如，如果一个表中的数据类型是整数，在另一个表中不要将其改为字符型。

## 创建表

### 创建表必要的做的事

- 命名表名。
- 定义列名。
- 定义列的数据类型。
- 如果有需要的话，可以给这些列定义一些列的属性，比如不许存储 NULL，设置默认值等等。

注意创建表的前提是：

- CREATE TABLE 权限
- 存储空间足够

### 语法

```
CREATE TABLE [IF NOT EXISTS] 表名(
        字段1, 数据类型 [约束条件] [默认值],
        字段2, 数据类型 [约束条件] [默认值],
        字段3, 数据类型 [约束条件] [默认值],
        ……
        [表约束条件]
);
```

加上了 IF NOT EXISTS 关键字，则表示：如果当前数据库中不存在要创建的数据表，则创建数据表；如果当前数据库中已经存在要创建的数据表，则忽略建表语句，不再创建数据表。<br />例如：

```
CREATE TABLE IF NOT EXISTS student {
    id INT,
    name VARCHAR(20),
    birthday DATETIME
}
```

### 拿其它表的数据来创建表

```
CREATE TABLE emp1 AS SELECT * FROM employees;
-- WHERE 语句后面会讲WHERE用法
CREATE TABLE emp2 AS SELECT * FROM employees WHERE employees.id=2
```

## 表约束

我们在建表的时候，还可以在表里添加一些约束，比如说 UNIQUE 唯一约束，就限制我们填写这一列的内容必须不能重复，比如手机号码。

### 为什么需要表约束

数据完整性（Data Integrity）是指数据的精确性（Accuracy）和可靠性（Reliability）。它是防止数据库中存在不符合语义规定的数据和防止因错误信息的输入输出造成无效操作或错误信息而提出的。

### 约束的分类

- **根据约束数据列的限制**，约束可分为：
  - **单列约束**：每个约束只约束一列
  - **多列约束**：每个约束可约束多列数据
- **根据约束的作用范围**，约束可分为：
  - **列级约束**：只能作用在一个列上，跟在列的定义后面
  - **表级约束**：可以作用在多个列上，不与列一起，而是单独定义

详细信息如下：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052600257-53c0186c-782d-4675-9bf5-974e7d9bf913.png#averageHue=%23f3f4f4&clientId=u589f0ee3-d396-4&from=paste&id=uf2c849ed&originHeight=290&originWidth=1976&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u2fed6bd0-b552-4871-8b2c-dced0681852&title=)<br />接下来就开始分别介绍每一种约束：

### DEFAULT 约束

给某个字段/某列指定默认值，一旦设置默认值，在插入数据时，如果此字段没有显式赋值，则赋值为默认值。<br />例如：

```
create table employee (
    id int primary key,
    gender char not null default '男'
);
```

上面我们设置 性别 列为 char 类型，并且如果不填写默认就是'男'。我们插入一条语句

```
insert into employee(id) values(1);

mysql> select * from employee;
+-----+-------+--
| eid | gender |
+-----+-------+---
|   1 | 男     |
+-----+-------+---
```

### NOT NULL 属性

有时候我们需要要求表中的某些列中必须有值，不能存放 NULL，那么可以用这样的语法来定义这个列：

```
列名 列的类型 NOT NULL
```

注意事项

- 默认，所有的类型的值都可以是 NULL，包括 INT、FLOAT 等数据类型
- 非空约束只能出现在表对象的列上，只能某个列单独限定非空，不能组合非空
- 一个表可以有很多列都分别限定了非空
- 空字符串’ '不等于 NULL，0 也不等于 NULL

举个例子：

```
CREATE IF NOT EXIST TABLE student(
    id int,
    name varchar(20) NOT NULL,
);

insert into student values(3,null); # error
```

删除非空约束

```
alter table 表名称 modify 字段名 数据类型 NULL;

ALTER TABLE student
MODIFY name VARCHAR(20) NULL;
```

### 主键

用来唯一标识表中的一行记录。主键约束列不允许重复，也不允许出现空值。<br />注意事项：

- 一个表最多只能有一个主键约束，建立主键约束可以在列级别创建，也可以在表级别上创建。
- 主键约束对应着表中的一列或者多列（复合主键）
- 如果是多列组合的复合主键约束，那么这些列都不允许为空值，并且组合的值不允许重复。
- MySQL 的主键名总是 PRIMARY，就算自己命名了主键约束名也没用。
- 当创建主键约束时，系统默认会在所在的列或列组合上建立对应的主键索引（能够根据主键查询的，就根据主键查询，效率更高）。如果删除主键约束了，主键约束对应的索引就自动删除了。
- 需要注意的一点是，不要修改主键字段的值。因为主键是数据记录的唯一标识，如果修改了主键的值，就有可能会破坏数据的完整性。

在列上声明主键：

```
CREATE TABLE student (
    id INT PRIMARY KEY,
    name VARCHAR(5),
);
```

在表级别声明主键：

```
CREATE TABLE student (
    id INT,
    name VARCHAR(5),
    PRIMARY KEY (id)
);

insert into student values(1,'张三');#成功
insert into student values(2,'李四');#成功

mysql> select * from temp;
+----+------+
| id | name |
+----+------+
|  1 | 张三 |
|  2 | 李四 |
+----+------+
```

MySQL 会对我们插入的记录做校验，如果已经有相同值的主键在表中了，就会插入失败，如下：

```
insert into temp values(1,'张三');#失败
```

我们可以对多个字段声明联合主键，语法如下：

```
create table 表名称(
    字段名  数据类型,
    字段名  数据类型,
    字段名  数据类型,
    primary key(字段名1,字段名2)
);
```

例如，选课表中，学生和课程可以组成联合主键

```
-- 学生表
create table student(
    sid int primary key,  -- 学号
    sname varchar(20)     -- 学生姓名
);

-- 课程表
create table course(
    cid int primary key,  -- 课程编号
    cname varchar(20)     -- 课程名称
);

-- 选课表
create table student_course(
    sid int,
    cid int,
    score int,
    primary key(sid,cid)  -- 复合主键
);
```

### UNIQUE 属性

用来限制某个字段/某列的值不能重复。<br />注意事项：

- 同一个表可以有多个唯一约束。
- 唯一约束可以是某一个列的值唯一，也可以多个列组合的值唯一。
- 唯一性约束允许列值为空。
- 在创建唯一约束的时候，如果不给唯一约束命名，就默认和列名相同。
- **MySQL 会给唯一约束的列上默认创建一个唯一索引。**

语法：

```
-- 列级约束
create table 表名称(
    字段名  数据类型,
    字段名  数据类型  unique,
    字段名  数据类型  unique key,
    字段名  数据类型
);

-- 表级约束
create table 表名称(
    字段名  数据类型,
    字段名  数据类型,
    字段名  数据类型,
    [constraint 约束名] unique key(字段名)
);
```

主键和 UNIQUE 约束都确保了某列或列组合的唯一性，但有以下区别：

1. 主键只能在表中定义一次，而可以定义多个 UNIQUE 约束。
2. 主键列不允许包含 NULL 值，而声明了 UNIQUE 属性的列可以包含 NULL 值，并且 NULL 值可以在多条记录中重复出现。

### 外键

限定某个表的某个字段的引用完整性。在阿里的规范里面强制不能使用外键，大家了解一下就行。至于为什么，我们讲完外键揭晓。<br />以下是关于外键的一些要点：

1. 外键列必须引用主表的主键或唯一约束列。这是因为外键列依赖于主表的唯一值。确保被依赖的值是唯一的是外键约束的关键。
2. 创建外键约束时，可以选择为外键约束命名，否则系统会自动生成一个默认的外键名（例如，student_ibfk_1）。可以根据需要指定外键约束的名称。
3. 在创建表时，应首先创建主表，然后再创建从表，并在从表中指定外键约束。这是因为外键依赖于主表的存在。
4. 删除表时，应先删除从表（或删除外键约束），然后再删除主表。这是因为如果主表的记录被从表引用，主表的记录将无法删除。在删除数据之前，必须先删除从表中引用该记录的数据。
5. 一个表可以建立多个外键约束，因此在从表中可以指定多个外键列。
6. 从表的外键列与主表被引用的列名可以不同，但数据类型必须相同且具有相同的逻辑意义。如果数据类型不匹配，将导致创建子表时出现错误。
7. 创建外键约束时，默认情况下系统会在相应的列上创建普通索引。索引名将与外键约束的名称相同。这有助于提高外键查询的效率。
8. 在删除外键约束后，必须手动删除相应的索引。

语法：

```
create table 主表名称(
    字段1  数据类型  primary key,
    字段2  数据类型
);

create table 从表名称(
    字段1  数据类型  primary key,
    字段2  数据类型,
    [CONSTRAINT <外键约束名称>] FOREIGN KEY（从表的某个字段) references 主表名(被参考字段)
);
```

举例：

```
CREATE TABLE student_score (
    number INT,
    subject VARCHAR(30),
    score TINYINT,
    PRIMARY KEY (number, subject),
    CONSTRAINT FOREIGN KEY(number) REFERENCES student_info(number)
);
```

### AUTO_INCREMENT 属性

某个字段需要自增功能，

- 首先，确保该字段的数据类型是允许自增的，例如使用 INT 或 BIGINT 类型。
- 在创建表时，为该字段添加自增属性。在 MySQL 中，可以使用 AUTO_INCREMENT 关键字来指定该字段为自增列。例如，创建一个名为 id 的自增主键列：

```
id INT AUTO_INCREMENT PRIMARY KEY
```

注意事项：

1. 插入新记录时，无需显式指定自增字段的值。如果未指定值、将其设为 NULL 或 0，数据库将自动将该字段的值设置为当前最大值加 1。
2. 表中最多只能有一个具有 AUTO_INCREMENT 属性的列。
3. 具有 AUTO_INCREMENT 属性的列必须建立索引。主键和具有 UNIQUE 属性的列会自动建立索引。
4. 自增字段不能通过指定 DEFAULT 属性来设置默认值。
5. 通常，具有 AUTO_INCREMENT 属性的列用作主键，用于自动生成唯一标识记录的主键值。

语法：

```
create table 表名称(
    字段名  数据类型  primary key auto_increment,
    字段名  数据类型  unique key not null,
);
```

### 列的注释

在建表语句中，可以使用 COMMENT 语句为整个表添加注释。另外，我们也可以在每个列的定义中使用 COMMENT 语句为列添加注释。例如：

```
CREATE TABLE users (
    id INT COMMENT '用户ID',
    name VARCHAR(50) COMMENT '用户名',
    email VARCHAR(255) COMMENT '电子邮件',
    age INT COMMENT '年龄'
);
```

## 查看表结构

```
DESC 表名
```

例如：

```
mysql> DESC student_info;
+-----------------+-------------------+------+-----+---------+-------+
| Field           | Type              | Null | Key | Default | Extra |
+-----------------+-------------------+------+-----+---------+-------+
| number          | int(11)           | YES  |     | NULL    |       |
| name            | varchar(5)        | YES  |     | NULL    |       |
| sex             | enum('男','女')   | YES  |     | NULL    |       |
| id_number       | char(18)          | YES  |     | NULL    |       |
| department      | varchar(30)       | YES  |     | NULL    |       |
| major           | varchar(30)       | YES  |     | NULL    |       |
| enrollment_time | date              | YES  |     | NULL    |       |
+-----------------+-------------------+------+-----+---------+-------+
7 rows in set (0.00 sec)

mysql>
```

如果想要看表的存储引擎和字符编码，可以使用

```
SHOW CREATE TABLE 表名字
```

例如：

```
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL,
  `email` varchar(100) NOT NULL,
   PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

# DQL（查询语句）

学习查询语句之前，我们需要先有一些数据。

```
CREATE TABLE IF NOT EXISTS student_info (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(5),
    sex ENUM('男', '女'),
    department VARCHAR(30),
    major VARCHAR(30),
    createTime DATE
);
CREATE TABLE IF NOT EXISTS student_score (
    id INT AUTO_INCREMENT PRIMARY KEY,
    student_id INT UNSIGNED,
    subject VARCHAR(30),
    score TINYINT
);
INSERT INTO student_info(name, sex, department, major, createTime) VALUES
('杜子腾', '男', '计算机学院', '计算机科学与工程', NOW()),
('杜琦燕', '女', '计算机学院', '计算机科学与工程', NOW()),
('范统', '男', '计算机学院', '软件工程', NOW()),
('史珍香', '女', '计算机学院', '软件工程', NOW()),
('范剑', '男', '航天学院', '飞行器设计', NOW()),
('朱逸群', '男', '航天学院', '电子信息', NOW());

INSERT INTO student_score (subject, student_id, score) VALUES
('计算机网络', 1, 78),
('操作系统', 2, 88),
('操作系统', 3, 100),
('操作系统', 1, 98),
('计算机网络', 2, 59),
('计算机网络', 4, 61),
('计算机网络', 5, 55),
('计算机网络', 3, 46);
```

除了之前说的 docker 环境做练习，也可以访问一些在线 sql 平台，http://sqlfiddle.com/#!9/e8e680/2<br />然后我们的表数据如下：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052600231-c4ad7d82-376e-4ca6-9201-61671652fcbe.png#averageHue=%23f8f8f8&clientId=u589f0ee3-d396-4&from=paste&id=u923fd92c&originHeight=544&originWidth=2893&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u2a59cfae-19fb-4dad-b7c8-4b94dc31797&title=)<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052600285-a889b802-29ca-45e0-9b12-ae238b957fbc.png#averageHue=%23faf9f9&clientId=u589f0ee3-d396-4&from=paste&id=ub05c8da1&originHeight=703&originWidth=2893&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ua2c60e73-d710-49ae-b0d0-15c59331de3&title=)

### 简单查询

语法：

#### SELECT…

Select 语句可以直接查询常量，得到的结果也是常量

```
SELECT 1;
SELECT 9/2;
```

#### SELECT … FROM

- 语法：

```
SELECT  标识选择哪些列  FROM   标识从哪个表中选择
```

举例：

```
mysql> SELECT name FROM student_info;
```

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052600671-aae7c16e-56a7-4e9a-b9e6-b7d79aab4846.png#averageHue=%23f9f9f9&clientId=u589f0ee3-d396-4&from=paste&id=u39acdb09&originHeight=532&originWidth=669&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u11c6ecc1-e88b-4ebf-9306-b307615b717&title=)<br />注意：MySQL 中的 SQL 语句是不区分大小写的，因此 SELECT 和 select 的作用是相同的，但是，许多开发人员习惯将关键字大写、数据列和表名小写，读者也应该养成一个良好的编程习惯，这样写出来的代码更容易阅读和维护。

#### 列的别名

作用：

- 重命名一个列
- 紧跟列名，也可以**在列名和别名之间加入关键字 AS，别名使用双引号**，以便在别名中包含空格或特殊的字符并区分大小写。
- AS 可以省略
- 建议别名简短，见名知意

语法：

#### 去除重复行

默认情况下，查询会返回全部行，包括重复行

```
SELECT 列名 AS 学号 FROM student_info;

or

SELECT 列名 学号 FROM student_info;
```

举例

```
SELECT name as student_name FROM student_info;
```

```
SELECT subject FROM student_score;
```

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052600711-66f9b1d2-63cf-4298-b45e-1251f5138dc5.png#averageHue=%23f9f8f8&clientId=u589f0ee3-d396-4&from=paste&id=uf85fbfad&originHeight=594&originWidth=794&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ue8ffaa1a-f9fa-4133-b5d1-1a296f00594&title=)<br />在 SELECT 语句中使用关键字 DISTINCT 去除重复行

```
SELECT DISTINCT subject FROM student_score;
```

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052600858-ffd477eb-65f1-458e-917f-8256cf47973f.png#averageHue=%23f7f7f6&clientId=u589f0ee3-d396-4&from=paste&id=u02905f7f&originHeight=265&originWidth=419&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u15595488-bf68-4618-bd20-6b4c98e605e&title=)

- DISTINCT 需要放到所有列名的前面，如果写成 SELECT id, DISTINCT subject FROM student_score 会报错。

#### 空值参与运算

- 所有运算符或列值遇到 null 值，运算的结果都为 null

例如：

```
SELECT NULL + 5;
```

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052601014-00544e3b-ce63-464a-9440-11ce78b7d653.png#averageHue=%23f8f8f7&clientId=u589f0ee3-d396-4&from=paste&id=u80b97481&originHeight=185&originWidth=314&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uc476f016-9be7-49f1-983c-7b6a43df622&title=)

#### 查询常数的作用

可以加常数列，例如：

```
SELECT name, 'abc school' as school FROM student_info;
```

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052600988-5a119a81-1719-4357-8e9c-d02829a0d902.png#averageHue=%23f9f9f9&clientId=u589f0ee3-d396-4&from=paste&id=u15f131e4&originHeight=557&originWidth=1849&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u3c6615a0-0bd7-46e3-8c14-13ab71b7cf0&title=)

#### 过滤数据

语法：

```
SELECT 字段1,字段2 FROM 表名 WHERE 过滤条件
```

注意：

- 使用 WHERE 子句，将不满足条件的行过滤掉
- **WHERE 子句紧随 FROM 子句**

举例：

```
SELECT id, score FROM student_score WHERE score=78 ;
```

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052601284-447ba911-1636-43ad-bbfc-8361088d29c8.png#averageHue=%23fbfbfb&clientId=u589f0ee3-d396-4&from=paste&id=ufd73b672&originHeight=174&originWidth=1506&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u510a6866-18e9-4f7c-b8cb-312af84b91e&title=)

### 排序和分页

语法：

```
ORDER BY 列名 ASC|DESC
```

举例：

```
SELECT subject, score FROM student_score ORDER BY score;
```

我们可以看到下图的 score 是升序排列的，如果要降序，在 score 后面加上 DESC 即可<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052601333-cb85c49b-c36f-474a-a377-883278ca3bf9.png#averageHue=%23f9f9f9&clientId=u589f0ee3-d396-4&from=paste&id=ua66612ac&originHeight=732&originWidth=2948&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ud599614d-9d5f-4c80-9c3c-765bc70c325&title=)

#### 多列排序

语法：

```
ORDER BY 列1 ASC|DESC, 列2 ASC|DESC ...
```

举例：

```
SELECT student_id, score FROM student_score ORDER BY student_id, score;
```

如下图，我们先按照 student_id 升序排列，然后相同的 student_id 再升序排列<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052601579-7f494dc4-9965-4dca-a89c-9858c2a5abef.png#averageHue=%23fafafa&clientId=u589f0ee3-d396-4&from=paste&id=u9d6d2f40&originHeight=704&originWidth=2494&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ucae13e3a-d221-4af7-b956-f6965d09e47&title=)

#### 分页

分页的使用场景如下：

- 背景 1：查询返回的记录太多了，查看起来很不方便
- 背景 2：表里有 4 条数据，我们只想要显示第 2、3 条数据

语法：

```
LIMIT 开始行, 限制条数;
```

LIMIT 的优点是通过限制返回结果的数量，可以优化数据表的网络传输量，并提升查询效率。当我们确定只需要返回一条记录时，可以使用 LIMIT 1 来指示 SELECT 语句只返回一条记录。这样的好处是，SELECT 不需要扫描整个表，只需检索到满足条件的一条记录即可返回。这种优化可以减少不必要的数据传输和提高查询性能。

- 分页公式

```
SELECT * FROM table
LIMIT(PageNo - 1)*PageSize,PageSize;
```

例如我们要选取第二页，每页 2 条数据，我们就可以使用（PageNo = 2， PageSize = 2）

```
SELECT name, major FROM student_info LIMIT 2, 2;
```

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052601688-a5507f68-e754-4caa-bce1-85092170c44b.png#averageHue=%23f8f8f8&clientId=u589f0ee3-d396-4&from=paste&id=u42dbc99a&originHeight=292&originWidth=2122&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u8633944f-7705-4354-a5e6-11ccc1d32e2&title=)

### 运算符

#### 算术运算符

算术运算符主要用于数学运算，其可以连接运算符前后的两个数值或表达式，对数值或表达式进行加（+）、减（-）、乘（\*）、除（/）和取模（%）运算。<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052601752-b2b06311-55b7-4e4f-bf3f-3e7bc4651c1b.png#averageHue=%23eeeeee&clientId=u589f0ee3-d396-4&from=paste&id=u01b02fe1&originHeight=433&originWidth=1221&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u7f32a1be-8c29-4ca1-8032-08d6c534cfe&title=)

- 对于 -、+ 和 \*，如果两个操作数都是整数，则结果将以 BIGINT（64 位）精度计算。
- 如果两个操作数都是整数并且其中任何一个是无符号的，则结果是无符号整数。 对于减法，如果启用 NO_UNSIGNED_SUBTRACTION SQL 模式，则即使任何操作数无符号，结果也会有符号。
- 如果 +、-、/、\*、% 的任何操作数为浮点数或字符串值，则结果的精度为最大精度操作数的精度。
- 在使用 / 执行除法时，使用两个精确值操作数时的结果小数位数是第一个操作数的小数位数加上 div\_ precision_increment 系统变量的值（默认为 4）。 例如，表达式 5.05 / 0.014 的结果具有六位小数 (360.714286)。

#### 比较运算符

- 比较运算符用来对表达式左边的操作数和右边的操作数进行比较，比较的结果为真则返回 1，比较的结果为假则返回 0，其他情况则返回 NULL。
- 这些运算对数字和字符串都有效。字符串会根据需要自动转换为数字，并将数字转换为字符串。
- 比较运算符经常被用来作为 SELECT 查询语句的条件来使用，返回符合条件的结果记录。

以下是比较运算符简介，我们着重介绍一部分比较运算符，例如>=这种特别简单的运算符就不做介绍了。

| **名字**                                                                                                  | **描述**                                                                                                                        |
| --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| [>](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_greater-than)              | Greater than operator（大于运算符）                                                                                             |
| [>=](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_greater-than-or-equal)    | Greater than or equal operator（大于等于运算符）                                                                                |
| [<](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_less-than)                 | Less than operator（小于运算符）                                                                                                |
| [<>, !=](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_not-equal)            | Not equal operator（不等运算符）                                                                                                |
| [<=](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_less-than-or-equal)       | Less than or equal operator（小于等于运算符）                                                                                   |
| [<=>](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_equal-to)                | NULL-safe equal to operator（安全等于运算符）                                                                                   |
| =                                                                                                         | Equal operator（等于运算符）                                                                                                    |
| [BETWEEN ... AND ...](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_between) | Whether a value is within a range of values（是否值在一个范围）                                                                 |
| [COALESCE()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#function_coalesce)         | Return the first non-NULL argument（返回第一个非 null 的参数）                                                                  |
| [GREATEST()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#function_greatest)         | Return the largest argument（返回最大的参数）                                                                                   |
| [IN()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_in)                     | Whether a value is within a set of values（是否 value 在一组值里）                                                              |
| [INTERVAL()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#function_interval)         | Return the index of the argument that is less than the first argument（返回小于第一个参数的参数的下标）                         |
| [IS NOT NULL](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is-not-null)     | NOT NULL value test（判断一个值是否不为 NULL，如果不为 NULL 则返回 1，否则返回 0）                                              |
| [IS NULL](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is-null)             | NULL value test（判断一个值是否为 NULL，如果为 NULL 则返回 1，否则返回 0）                                                      |
| [LEAST()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#function_least)               | Return the smallest argument（在有两个或多个参数的情况下，返回最小值，假如任意一个自变量为 NULL，则 GREATEST()的返回值为 NULL） |
| [LIKE](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_like)            | Simple pattern matching（当有两个或多个参数时，返回值为最大值。假如任意一个自变量为 NULL，则 GREATEST()的返回值为 NULL）        |

##### 等号运算符

等号运算符（=）用于判断等号两边的值、字符串或表达式是否相等。如果相等，返回结果为 1；如果不相等，返回结果为 0。 在使用等号运算符时，请遵循以下规则，以确保正确比较：

- 如果等号两边的值、字符串或表达式都是字符串，MySQL 会按照字符串的方式进行比较。它会逐个比较每个字符串中的字符的 ANSI 编码，判断它们是否相等。
- 如果等号两边的值都是整数，MySQL 会按照整数的大小比较这两个值。
- 如果等号两边的值分别是一个整数和一个字符串，MySQL 会将字符串转换为数字，然后再比较它们的大小。
- 如果等号两边的值、字符串或表达式中有一个为 NULL，那么比较的结果也将为 NULL。

```
mysql> SELECT 1 = 1, 1 = '1', 1 = 0, 'a' = 'a', (5 + 3) = (2 + 6), '' = NULL , NULL = NULL;
+-------+---------+-------+-----------+-------------------+-----------+-------------+
| 1 = 1 | 1 = '1' | 1 = 0 | 'a' = 'a' | (5 + 3) = (2 + 6) | '' = NULL | NULL = NULL |
+-------+---------+-------+-----------+-------------------+-----------+-------------+
|    1  |     1   |   0   |      1    |             1     |    NULL   |        NULL  |
+-------+---------+-------+-----------+-------------------+-----------+-------------+
1 row in set (0.00 sec)
```

##### 安全等于运算符

安全等于运算符（<=>）与等于运算符（=）的作用是相似的，唯一的区别是‘<=>’可以用来对 NULL 进行判断。在两个操作数均为 NULL 时，其返回值为 1，而不为 NULL；当一个操作数为 NULL 时，其返回值为 0，而不为 NULL。

##### BETWEEN AND 运算符

BETWEEN AND 运算符用于在指定的范围内进行查询。<br />举个例子，假设我们有一个名为 age 的列，我们想要查询年龄在 18 到 30 之间的用户，可以使用以下语句：

```
SELECTFROM users WHERE age BETWEEN 18 AND 30;
```

这将返回所有年龄在 18 到 30 之间的用户记录。

##### **IN 运算符：**

IN 运算符用于匹配多个值。<br />举个例子，假设我们有一个名为 country 的列，我们想要查询来自美国、英国和加拿大的用户，可以使用以下语句：

```
SELECT * FROM users WHERE country IN ('USA', 'UK', 'Canada');
```

这将返回所有来自美国、英国和加拿大的用户记录。

##### LIKE 运算符

LIKE 运算符主要用来匹配字符串，通常用于模糊匹配，如果满足条件则返回 1，否则返回 0。如果给定的值或者匹配条件为 NULL，则返回结果为 NULL。<br />LIKE 运算符支持两种通配符来表示模式：

1. %：表示任意字符序列（可以是零个或多个字符）。
2. \_：表示单个字符。

下面是一些示例：

- SELECT \* FROM users WHERE name LIKE 'J%'：返回所有名字以字母 "J" 开头的用户。
- SELECT \* FROM users WHERE email LIKE '%@gmail.com'：返回所有邮箱地址以 "@gmail.com" 结尾的用户。
- SELECT \* FROM users WHERE phone LIKE '1**-\_-**'：返回所有电话号码以 "1" 开头，并且符合形式 "1XX-XXX-XXXX" 的用户。

此外，LIKE 运算符还可以与 NOT 运算符结合使用，以排除匹配特定模式的数据。

#### 逻辑运算符

在 MySQL 中，有几个逻辑运算符可用于在查询中组合和操作条件。常见的逻辑运算符包括：AND、OR、NOT。<br />下面是对每个逻辑运算符的简要介绍：

- AND：AND 运算符用于将多个条件组合在一起，并要求它们同时为真才返回结果。如果所有条件都为真，则返回 TRUE；如果任何一个条件为假，则返回 FALSE。

例如，以下语句将返回年龄大于 18 岁且国家为中国的用户记录。

```
SELECT * FROM users WHERE age > 18 AND country = 'China';
```

- OR：OR 运算符用于将多个条件组合在一起，并要求它们中的至少一个为真才返回结果。如果任何一个条件为真，则返回 TRUE；如果所有条件都为假，则返回 FALSE。

例如，以下将返回年龄大于 18 岁或者或者年龄大于 30 岁的

```
SELECT * FROM users WHERE age > 18 OR country = 'China;
```

- NOT：NOT 运算符用于取反一个条件的结果。它用于否定一个条件的真假值。如果条件为真，则返回 FALSE；如果条件为假，则返回 TRUE。

例如，以下将返回年龄不等于 18 岁

```
SELECTFROM users WHERE NOT age = 18;
```

逻辑运算符可用于构建复杂的查询条件，通过组合多个条件来过滤和检索数据。同时，它们还可以与括号结合使用以定义优先级，确保查询条件按预期进行组合和计算。

## 关联查询

指的是同时操作两个或更多个表进行查询操作。<br />在进行多表查询之前，需要满足一些前提条件。这些表之间必须存在关联关系，可以是一对一的关系或一对多的关系。<br />比如说学生表有学生的 id，成绩表也有学生的 id，为了获取更多学生信息，比如成绩， 就可以通过这个共同的 id 进行关联查询，这就是关联查询存在的意义。<br />MySQL 支持以下类型的连接：

- 内部联接
- 左连接
- 右连接
- 交叉连接

要了解关联查询之前，必须理解笛卡尔积，也就是交叉连接。为了讲解本章内容，我们需要提前准备表和数据。<br />首先，创建两个表，分别称为“members”和“committees”：

```
CREATE TABLE members (
    member_id INT AUTO_INCREMENT,
    name VARCHAR(100),
    PRIMARY KEY (member_id)
);

CREATE TABLE committees (
    committee_id INT AUTO_INCREMENT,
    name VARCHAR(100),
    PRIMARY KEY (committee_id)
);
```

其次，在表 members 和 commitments 中插入一些行。

```
INSERT INTO members(name)
VALUES('John'),('Jane'),('Mary'),('David'),('Amelia');

INSERT INTO committees(name)
VALUES('John'),('Mary'),('Amelia'),('Joe');
```

接着，查询 members 和 committee 表数据

```
SELECT * FROM members;

+-----------+--------+
| member_id | name   |
+-----------+--------+
|         1 | John   |
|         2 | Jane   |
|         3 | Mary   |
|         4 | David  |
|         5 | Amelia |
+-----------+--------+
5 rows in set (0.00 sec)
```

```
SELECT * FROM committees;

+--------------+--------+
| committee_id | name   |
+--------------+--------+
|            1 | John   |
|            2 | Mary   |
|            3 | Amelia |
|            4 | Joe    |
+--------------+--------+
4 rows in set (0.00 sec)
```

有些人在 committee 表，有些则不在。

### 笛卡尔积(交叉连接)的理解

简单介绍交叉连接的结果就是：假设第一个表有 n 行，第二个表有 m 行。 连接表的交叉连接将返回 nxm 行。<br />举例如下：

```
SELECT
    m.member_id,
    m.name AS member,
    c.committee_id,
    c.name AS committee
FROM
    members m
CROSS JOIN committees c;

+-----------+--------+--------------+-----------+
| member_id | member | committee_id | committee |
+-----------+--------+--------------+-----------+
|         1 | John   |            4 | Joe       |
|         1 | John   |            3 | Amelia    |
|         1 | John   |            2 | Mary      |
|         1 | John   |            1 | John      |
|         2 | Jane   |            4 | Joe       |
|         2 | Jane   |            3 | Amelia    |
|         2 | Jane   |            2 | Mary      |
|         2 | Jane   |            1 | John      |
|         3 | Mary   |            4 | Joe       |
|         3 | Mary   |            3 | Amelia    |
|         3 | Mary   |            2 | Mary      |
|         3 | Mary   |            1 | John      |
|         4 | David  |            4 | Joe       |
|         4 | David  |            3 | Amelia    |
|         4 | David  |            2 | Mary      |
|         4 | David  |            1 | John      |
|         5 | Amelia |            4 | Joe       |
|         5 | Amelia |            3 | Amelia    |
|         5 | Amelia |            2 | Mary      |
|         5 | Amelia |            1 | John      |
+-----------+--------+--------------+-----------+
20 rows in set (0.00 sec)
```

可以看到，members m CROSS JOIN committees c 就是把所有的连接可能都合并了一次。<br />注意：SQL92 中，笛卡尔积也称为交叉连接，英文是 CROSS JOIN。在 SQL99 中也是使用 CROSS JOIN 表示交叉连接。它的作用就是可以把任意表进行连接，即使这两张表不相关。<br />再次放一张关于 cross join 的图，一定要理解笛卡尔积。<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052601907-3b795df7-e5b6-4211-9e29-9acc0d458da9.png#averageHue=%23f6f3f3&clientId=u589f0ee3-d396-4&from=paste&id=ua274cbd2&originHeight=433&originWidth=913&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ub307f727-d7ac-4136-ad7b-fe498135731&title=)

### 内连接

内连接的本质如下图：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052602127-024462aa-9f61-417a-8557-f88f10c90853.png#averageHue=%23fdfdfd&clientId=u589f0ee3-d396-4&from=paste&id=u42f52e01&originHeight=235&originWidth=393&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u0b39ee29-a8da-410c-881a-924fa154a18&title=)<br />可以简单理解为，members 表和 committees 表做了一个笛卡尔积，然后筛选出符合条件（ON 后面的连接条件）的数据。举例如下：

```
SELECT
    m.member_id,
    m.name AS member,
    c.committee_id,
    c.name AS committee
FROM
    members m
INNER JOIN committees c ON c.name = m.name;

+-----------+--------+--------------+-----------+
| member_id | member | committee_id | committee |
+-----------+--------+--------------+-----------+
|         1 | John   |            1 | John      |
|         3 | Mary   |            2 | Mary      |
|         5 | Amelia |            3 | Amelia    |
+-----------+--------+--------------+-----------+
3 rows in set (0.00 sec)
```

可以看到，把名字相同的数据筛选了出来。

### 左连接

左连接的本质是如下图：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052602396-4ff67308-8949-4254-a51c-1b5ce9547399.png#averageHue=%23a7dff8&clientId=u589f0ee3-d396-4&from=paste&id=u974a52e3&originHeight=226&originWidth=395&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u3ca03842-07dc-48b8-8f48-5cb6cf88eb2&title=)<br />我们再来一张图，注意体会里面的细节:<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052602355-ad32d62e-7833-4403-96bd-d6e37c5ef6ce.png#averageHue=%23fdfdfd&clientId=u589f0ee3-d396-4&from=paste&id=udd23a33f&originHeight=120&originWidth=418&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u082843dd-154b-46c8-bfa9-063f7e525e2&title=)<br />上图是 A 表 LEFT JOIN B 表，A 表中无论跟 B 表重合还是不重合的数据都留下来了。整体过程如下：

- 左连接选择从左表开始的数据。 对于左表中的每一行，左连接将与右表中的每一行进行比较。
- 如果两行中的值满足联接条件，则左联接子句将创建一个新行，该行的列包含两个表中行的所有列，并将该行包含在结果集中。
- 如果两行中的值不匹配，左连接子句仍会创建一个新行，该新行的列包含左表中该行的列，而右表中该行的列为 NULL。
- 换句话说，无论右表中是否存在匹配的行，左连接都会选择左表中的所有数据。

举例如下：

```
SELECT
    m.member_id,
    m.name AS member,
    c.committee_id,
    c.name AS committee
FROM
    members m
LEFT JOIN committees c ON member.name = committee.name


+-----------+--------+--------------+-----------+
| member_id | member | committee_id | committee |
+-----------+--------+--------------+-----------+
|         1 | John   |            1 | John      |
|         2 | Jane   |         NULL | NULL      |
|         3 | Mary   |            2 | Mary      |
|         4 | David  |         NULL | NULL      |
|         5 | Amelia |            3 | Amelia    |
+-----------+--------+--------------+-----------+
5 rows in set (0.00 sec)
```

可以的看到出来就是 member 表去找每一条 committee 表的记录，如果有符合 ON 条件的，就合并在一起，没有的就返回 NULL。<br />此时我们可以更进一步，如果想要查找下图所示内容：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052602642-10d53496-6110-4472-9483-c4e431e61338.png#averageHue=%23afe1f9&clientId=u589f0ee3-d396-4&from=paste&id=u67ad4013&originHeight=221&originWidth=350&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u66cc7b63-0ffd-4133-a727-081aa87e29f&title=)<br />是不是就是把 NULL 的部分取出来，所以 Sql 可以在末尾添加 WHERE c.committee_id IS NULL,如下：

```
SELECT
    m.member_id,
    m.name AS member,
    c.committee_id,
    c.name AS committee
FROM
    members m
LEFT JOIN committees c USING(name)
WHERE c.committee_id IS NULL;
```

### 右连接

右连接就不多说了，跟左连接差不多，只是左连接以左表为主，右连接以右表为主。我们直接看案例：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052602845-11ce2a88-0c32-48b3-9acf-0939d907f22f.png#averageHue=%239bdaf8&clientId=u589f0ee3-d396-4&from=paste&id=u181c2e7f&originHeight=220&originWidth=357&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uffc8d622-1ea5-4ca2-969c-4bc033ac1ec&title=)

```
SELECT
    m.member_id,
    m.name AS member,
    c.committee_id,
    c.name AS committee
FROM
    members m
RIGHT JOIN committees c ON c.name = m.name;

+-----------+--------+--------------+-----------+
| member_id | member | committee_id | committee |
+-----------+--------+--------------+-----------+
|         1 | John   |            1 | John      |
|         3 | Mary   |            2 | Mary      |
|         5 | Amelia |            3 | Amelia    |
|      NULL | NULL   |            4 | Joe       |
+-----------+--------+--------------+-----------+
4 rows in set (0.00 sec)
```

如何得到下图的结果呢？<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052602816-804ad610-37ee-435b-96a2-0bb2db6abae0.png#averageHue=%23ade1f9&clientId=u589f0ee3-d396-4&from=paste&id=u46f982b4&originHeight=208&originWidth=363&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u51add51e-179d-4ae9-993c-189441cf5ce&title=)

```
SELECT
    m.member_id,
    m.name AS member,
    c.committee_id,
    c.name AS committee
FROM
    members m
RIGHT JOIN committees c ON c.name = m.name
WHERE m.member_id IS NULL;

+-----------+--------+--------------+-----------+
| member_id | member | committee_id | committee |
+-----------+--------+--------------+-----------+
|      NULL | NULL   |            4 | Joe       |
+-----------+--------+--------------+-----------+
1 row in set (0.00 sec)
```

# mysql 常用函数

MySQL 提供了丰富的内置函数，用于在查询和数据处理中进行各种操作。这里，我将这些丰富的内置函数再分为两类：单行函数、聚合函数（或分组函数）。<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052602962-1832ee9e-1582-4e82-83ae-d4d7f87e5a91.png#averageHue=%23f9f0e9&clientId=u589f0ee3-d396-4&from=paste&id=u40a9a4bc&originHeight=268&originWidth=500&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ua7ed7c0a-4b74-44f7-994e-a6b358f5c5e&title=)

## 单行函数

### 常用的字符串函数

### concat(s1,s2...,sn)

将 s1,s2...,sn 连接成字符串，如果该函数中的任何参数为 null，返回结果为 null<br />举例：

```
SELECT
  CONCAT('John', ' ', 'Doe') full_name;

+-----------+
| full_name |
+-----------+
| John Doe  |
+-----------+
1 row in set (0.01 sec)
```

- concat_ws(sep,s1,s2...,sn)

将 s1,s2...,sn 连接成字符串,并用 sep 字符间隔<br />举例：

```
SELECT CONCAT_WS(',', 'John', 'Doe') full_name;

+-----------+
| full_name |
+-----------+
| John,Doe  |
+-----------+
1 row in set (0.00 sec)
```

- substr(str,start,len)

从 start 位置开始截取字符串，len 表示要截取的长度；

```
SELECT substr('abcdefg', 2, 5);

+-----------+
| substr('abcdefg', 2, 5) |
+-----------+
| bcdef  |
+-----------+
1 row in set (0.00 sec)
```

- lcase(str)或 lower(str)

将字符串 str 转换为小写形式<br />举例：

```
SELECT LOWER('MySQL');

+----------------+
| LOWER('MySQL') |
+----------------+
| mysql          |
+----------------+
1 row in set (0.00 sec)
```

- upper(str)和 ucase(str)

将字符串转换为大写形式 <br />举例：

```
SELECT UPPER('MySQL');

+----------------+
| UPPER('MySQL') |
+----------------+
| MYSQL          |
+----------------+
1 row in set (0.00 sec)
```

- length(s)

返回字符串 str 中的字符数<br />因为一般都是使用 utf8mb4 作为 mysql 的默认字符集，所以中文的一个字也算一个字符<br />举例：

```
SELECT LENGTH('孟祥同学') as length;

+--------+
| length |
+--------+
|      4 |
+--------+
1 row in set (0.00 sec)
```

- trim(str)

去除字符串首部和尾部的所有空格

```
SELECT TRIM(' MySQL TRIM Function ') as trim;

+--------+
| trim |
+--------+
| MySQL TRIM Function |
+--------+
1 row in set (0.00 sec)
```

- left(str,x)

返回字符串 str 中最左边的 x 个字符 举例：

```
SELECT LEFT('MySQL LEFT', 5) as left;

+--------+
| left  |
+--------+
| MySQL |
+--------+
1 row in set (0.00 sec)
```

- right(str,x)

返回字符串 str 中最右边的 x 个字符

```
SELECT RIGHT('MySQL', 3);

+-------------------+
| RIGHT('MySQL', 3) |
+-------------------+
| SQL               |
+-------------------+
1 row in set (0.00 sec)
```

### 数字函数

### 日期函数

### 聚合函数

### 流程控制函数

## 聚合函数

聚合函数作用于一组数据，并对一组数据返回一个值。<br />聚合函数类型

### AVG()

MySQL AVG()函数是一个[聚合函数](http://www.yiibai.com/mysql/aggregate-functions.html)，它用于计算一组值或表达式的平均值。

```
AVG(DISTINCT expression)
```

您可以使用 AVG()函数中的[DISTINCT](http://www.yiibai.com/mysql/distinct.html)运算符来计算不同值的平均值。 例如，如果您有一组值 1,1,2,3，具有 DISTINCT 操作的 AVG()函数将返回不同值的和，即：(1 + 2 + 3)/3 = 2.00 。

### SUM()

SUM()函数用于计算一组值或表达式的总和。

```
SUM(DISTINCT expression)
```

SUM()函数是如何工作的？

- 如果在没有返回匹配行[SELECT](http://www.yiibai.com/mysql/select-statement-query-data.html)语句中使用 SUM 函数，则 SUM 函数返回 NULL，而不是 0。
- DISTINCT 运算符允许计算集合中的不同值。
- SUM 函数忽略计算中的 NULL 值。

### MAX()

MAX()函数返回一组值中的最大值。MAX()函数在许多查询中非常方便，例如查找最大数量，最昂贵的产品以及客户的最大付款。

```
MAX(DISTINCT expression);
```

如果添加 DISTINCT 运算符，则 MAX 函数返回不同值的最大值，它与所有值的最大值相同。 这意味着 DISTINCT 运算符不会对 MAX 函数产生任何影响(用不用 DISTINCT 运算符都可以)。<br />DISTINCT 运算符在其他[聚合函数](http://www.yiibai.com/mysql/aggregate-functions.html)(如[COUNT](http://www.yiibai.com/mysql/count.html)，[SUM](http://www.yiibai.com/mysql/sum.html)和[AVG](http://www.yiibai.com/mysql/avg.html))中生效。

### MIN()

MIN()函数返回一组值中的最小值。MIN()函数在某些情况下非常有用，例如找到最小的数字，选择最便宜的产品，获得最低的信用额度等。

```
MIN(DISTINCT expression);
```

如果指定 DISTINCT 运算符，则 MIN 函数返回不同值的最小值，与省略 DISTINCT 相同。换句话说，DISTINCT 运算符对 MIN 函数没有任何影响。

### COUNT()

COUNT()函数返回表中的行数。 COUNT()函数允许您对表中符合特定条件的所有行进行计数。

```
COUNT(expression)
```

COUNT()函数的返回类型为 BIGINT。 如果没有找到匹配的行，则 COUNT()函数返回 0。<br />COUNT 函数有几种形式：COUNT(\*)，COUNT(expression)和 COUNT(DISTINCT expression)。

- **MySQL COUNT(\*)函数**

COUNT(_)函数返回由[SELECT](http://www.yiibai.com/mysql/select-statement-query-data.html)语句返回的结果集中的行数。COUNT(_)函数计算包含 NULL 和非 NULL 值的行，即：所有行。<br />如果使用 COUNT(\*)函数对表中的数字行进行计数，而不使用[WHERE 子句](http://www.yiibai.com/mysql/where.html)选择其他列，则其执行速度非常快。<br />这种优化仅适用于*MyISAM*表，因为*MyISAM*表的行数存储在 information_schema 数据库的 tables 表的 table_rows 列中; 因此，MySQL 可以很快地检索它。

- **MySQL COUNT(expression)**

COUNT(expression)返回不包含 NULL 值的行数。

- **MySQL COUNT(DISTINCT expression)**

MySQL COUNT(DISTINCT expression)返回不包含 NULL 值的唯一行数。<br />以上的聚合函数需要跟 GROUP BY 来根据列或表达式的值将行分组。所以以下的举例会结合 GROUP BY 操作符。<br />语法如下：

```
SELECT
    c1, c2,..., cn, aggregate_function(ci)
FROM
    table_name
WHERE
    conditions
GROUP BY c1 , c2,...,cn;
```

在此语法中，将 GROUP BY 子句放置在 FROM 和 WHERE 子句之后。 在 GROUP BY 关键字后面，列出要分组的列或表达式，并用逗号分隔。<br />注：mysql 关键字的执行顺序：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052603053-ebb71ab8-a529-4af2-8e8d-33de9e5bfe61.png#averageHue=%23b7b7b7&clientId=u589f0ee3-d396-4&from=paste&id=u673b013b&originHeight=62&originWidth=1056&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=udeefa3d7-e4f7-41ee-b6cc-ad6c22cba94&title=)<br />练习的表如下：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052603361-3a5a7bd7-b24f-41a9-b72c-c3acbff2db70.png#averageHue=%23dcdcdc&clientId=u589f0ee3-d396-4&from=paste&id=u8e9ab8f2&originHeight=219&originWidth=167&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ud1874eac-fe67-4a8b-af21-c50f7b3d21b&title=)

```
CREATE TABLE orders (
  orderNumber int,
  orderDate date NOT NULL,
  requiredDate date NOT NULL,
  shippedDate date DEFAULT NULL,
  status varchar(15) NOT NULL,
  comments text,
  customerNumber int NOT NULL,
  PRIMARY KEY (orderNumber),
);
```

因为数据太多，大家展示一下字段内容：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052603400-b5cec3d7-e0e1-4ed4-ad02-cac8965c7442.png#averageHue=%23eceded&clientId=u589f0ee3-d396-4&from=paste&id=ue34c222c&originHeight=430&originWidth=1478&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u7cd40acc-77ee-4185-b574-17e94c04b59&title=)

### GROUP BY

我们用一张图让大家理解 GROUP BY 的用法：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052603504-2302c17c-040e-4c90-bf28-e1faace0d686.png#averageHue=%23f4eae7&clientId=u589f0ee3-d396-4&from=paste&id=ua04a913c&originHeight=374&originWidth=573&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ue22cc626-ffaa-497f-b350-737a7603bd5&title=)<br />上面是一个员工表，为了求出每个部门的平均工资，让其对 department_id 进行分组，然后使用 AVG 函数求出每个分组的平均工资：

```
SELECT department_id, AVG(SALARY) FROM employees GROUP BY department_id;
```

上面的案例应该足够让大家理解 GROUP BY 的使用方法了，下面是更多的案例，使用的是上面提到的 orders 表：<br />以下 GROUP BY 语句筛选出了所有的 status

```
SELECT
  status
FROM
  orders
GROUP BY
  status;

+------------+
| status     |
+------------+
| Shipped    |
| Resolved   |
| Cancelled  |
| On Hold    |
| Disputed   |
| In Process |
+------------+
6 rows in set (0.02 sec)
```

GROUP BY 出现，一般都是跟聚合函数相关，如下跟 COUNT 结合，计算每种 status 的个数

```
SELECT
  status,
  COUNT(*)
FROM
  orders
GROUP BY
  status;

+------------+----------+
| status     | COUNT(*) |
+------------+----------+
| Shipped    |      303 |
| Resolved   |        4 |
| Cancelled  |        6 |
| On Hold    |        4 |
| Disputed   |        3 |
| In Process |        6 |
+------------+----------+
6 rows in set (0.01 sec)
```

以下是连表查询的示例，聚合相同年份的总收入：

```
SELECT
  YEAR(orderDate) AS year,
  SUM(quantityOrdered * priceEach) AS total
FROM
  orders
  INNER JOIN orderdetails USING orders.name = orderdetails.name
WHERE
  status = 'Shipped'
GROUP BY
  YEAR(orderDate);

+------+------------+
| year | total      |
+------+------------+
| 2003 | 3223095.80 |
| 2004 | 4300602.99 |
| 2005 | 1341395.85 |
+------+------------+
3 rows in set (0.02 sec)
```

### Having

- HAVING 是对 GROUP BY 后的数据进行再次筛选。
- HAVING 不能单独使用，必须要跟 GROUP BY 一起使用。

语法如下：

```
SELECT
    select_list
FROM
    table_name
WHERE
    search_condition
GROUP BY
    group_by_expression
HAVING
    group_condition;
```

我们用一个例子来理解 having 的作用：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052603597-f62f6da1-35d7-46fe-ae0a-1005a793e7a5.png#averageHue=%23f7efeb&clientId=u589f0ee3-d396-4&from=paste&id=uc23793b4&originHeight=388&originWidth=645&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u50c55a52-7d77-44c5-a94c-fe0831c7cfb&title=)<br />上图筛选出了每个部门最高工资比 10000 高的部门：<br />首先题目说每个部门，那么需要使用 GROUP BY 来分组，然后用聚合函数 max 求出每个部门的最高工资，如下：

```
SELECT department_id, MAX(salary) FROM employees GROUP BY department_id;
```

在以上结果的基础上用 having 筛选：

```
SELECT department_id, MAX(salary) as maxSalary FROM employees GROUP BY department_id HAVING maxSalary > 10000;
```

拥有 having 的 mysql 查询的执行顺序如下：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052603760-389f2c5c-6529-4a04-9474-9765d8abf67d.png#averageHue=%23b6b6b6&clientId=u589f0ee3-d396-4&from=paste&id=u568a58cc&originHeight=62&originWidth=1215&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u5da23085-9326-4634-9888-e0f3fd47294&title=)<br />在 SELECT 语句执行这些步骤的时候，每个步骤都会产生一个虚拟表，然后将这个虚拟表传入下一个步骤中作为输入。需要注意的是，这些步骤隐含在 SQL 的执行过程中，对于我们来说是不可见的。<br />SELECT 是先执行 FROM 这一步的。在这个阶段，如果是多张表联查，还会经历下面的几个步骤：

- 首先先通过 CROSS JOIN 求笛卡尔积，相当于得到虚拟表 vt（virtual table）1-1；
  - 通过 ON 进行筛选，在虚拟表 vt1-1 的基础上进行筛选，得到虚拟表 vt1-2；
  - 添加外部行。如果我们使用的是左连接、右链接或者全连接，就会涉及到外部行，也就是在虚拟表 vt1-2 的基础上增加外部行，得到虚拟表 vt1-3。
  - 当然如果我们操作的是两张以上的表，还会重复上面的步骤，直到所有表都被处理完为止。这个过程得到是我们的原始数据。
- 当我们拿到了查询数据表的原始数据，也就是最终的虚拟表 vt1，就可以在此基础上再进行 WHERE 阶段。在这个阶段中，会根据 vt1 表的结果进行筛选过滤，得到虚拟表 vt2。
- 然后进入第三步和第四步，也就是 GROUP 和 HAVING 阶段。在这个阶段中，实际上是在虚拟表 vt2 的基础上进行分组和分组过滤，得到中间的虚拟表 vt3 和 vt4。
- 当我们完成了条件筛选部分之后，就可以筛选表中提取的字段，也就是进入到 SELECT 和 DISTINCT 阶段。
- 首先在 SELECT 阶段会提取想要的字段，然后在 DISTINCT 阶段过滤掉重复的行，分别得到中间的虚拟表 vt5-1 和 vt5-2。
- 当我们提取了想要的字段数据之后，就可以按照指定的字段进行排序，也就是 ORDER BY 阶段，得到虚拟表 vt6。
- 最后在 vt6 的基础上，取出指定行的记录，也就是 LIMIT 阶段，得到最终的结果，对应的是虚拟表 vt7。

当然我们在写 SELECT 语句的时候，不一定存在所有的关键字，相应的阶段就会省略。<br />同时因为 SQL 是一门类似英语的结构化查询语言，所以我们在写 SELECT 语句的时候，还要注意相应的关键字顺序，所谓底层运行的原理，就是我们刚才讲到的执行顺序。

# 子查询

MySQL 子查询是嵌套在另一个查询（例如 SELECT、INSERT、UPDATE 或 DELETE）中的查询。 此外，子查询可以嵌套在另一个子查询中。<br />MySQL 子查询称为内部查询，而包含子查询的查询称为外部查询。 子查询可以在使用表达式的任何地方使用，并且必须用括号括起来。

## 基本概念

我们先举个例子来理解什么是子查询（按照 JavaScript 的角度来说，就是嵌套循环）：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052604032-3dcb74aa-025a-4b67-aac1-303e3b50a558.png#averageHue=%23eec99a&clientId=u589f0ee3-d396-4&from=paste&id=u8564c16e&originHeight=342&originWidth=705&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u97030ce4-4037-43f0-a0d3-9fdcf99ec75&title=)<br />例如，我们要知道谁的工资比 Abel 高？那么这个问题可以拆分为两个子问题：

- Abel 的工资是多少

```
SELECT salary
FROM employees
WHERE last_name = 'Abel'
```

- 谁的工资比 Abel 的工资高

```
SELECT last_name,salary
FROM employees
WHERE salary > Abel的工资
```

所以我们只要把二者结合，得到:

```
SELECT last_name,salary
FROM employees
WHERE salary > (
                SELECT salary
                FROM employees
                WHERE last_name = 'Abel'
                );
```

括号内的就是子查询，括号外的是外查询。<br />上面的子查询语法如下，其中红框部分就是子查询，红框外是外查询：<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052604123-c42ea13d-8281-4b2b-a181-cc8522562a46.png#averageHue=%23fafac8&clientId=u589f0ee3-d396-4&from=paste&id=u2d609f77&originHeight=155&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u50af3cd0-99e0-44c8-9f76-0278d972d7f&title=)<br />子查询（内查询）在主查询之前一次执行完成。

- 子查询的结果被主查询（外查询）使用 。
- **注意事项**
  - 子查询要包含在括号内
  - 将子查询放在比较条件的右侧
  - 单行操作符对应单行子查询，多行操作符对应多行子查询

## 子查询的分类

子查询我们主要了解两类，一类是相关子查询，一类是不相关子查询。它们到底有什么区别呢，我举一个例子：

```
-- 相关子查询
SELECT *
FROM table1
WHERE column1 = (SELECT column2 FROM table2 WHERE table2.column3 = table1.column4);

-- 非相关子查询
SELECT *
FROM table1
WHERE column1 = (SELECT column2 FROM table2);
```

在第一个示例中，内部查询使用了外部查询的列 table1.column4 作为过滤条件，因此它是一个相关子查询。也就是子查询内部引用外部列。<br />而在第二个示例中，内部查询不引用外部查询的列，它只是独立地返回 table2 表中的 column2 列，因此它是一个非相关子查询。

## 子查询练习表

接下来我们会通过案例，来练习和加深对子查询的理解，所以，先把表信息列出：

## 表信息

接下来我们大致介绍一下表信息<br />![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052604097-a95254e4-f329-40c9-9ddc-17d18a4dc823.png#averageHue=%23efeeee&clientId=u589f0ee3-d396-4&from=paste&id=u5552158d&originHeight=402&originWidth=1015&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u8e2b96f9-a7c3-4ed6-854a-87d16e2a577&title=)<br />主要分为员工表 EMPLOYEES，部门表 DEPARTMENTS，地点表 LOCATIONS

- EMPLOYEES 包含
  - employee_id 员工 id，唯一
  - first_name 名
  - last_name 姓
  - email 邮箱
  - phone_number 手机号
  - job_id 职位 id
  - salary 奖金
  - commission_pct 提成，也可以说奖金率
  - manager_id 自己上司的 employee_id
  - department_id 部门 id
-
- DEPARTMENT 包含
  - department_id 部门 id
  - department_name 部门名字
  - manager_id 自己上司的 employee_id
  - location_id 工作地点 id
- LOCATIONS 包含
  - location_id 工作地点 id
  - street_address 地址
  - postal_code 邮政编码
  - city 城市
  - state_province 州省
  - country_id 国家 id

## 不相关子查询

我们通过几个案例来理解不想管子查询、

1. **查询工资大于 149 号员工工资的员工的信息**

我们需要将 149 号员工的工资信息作为子查询的结果

```
SELECT last_name
FROM   employees
WHERE  salary >
                (SELECT salary
                 FROM employees
                 WHERE employees_id = 149);
```

1. **返回公司工资最少的员工的 last_name,job_id 和 salary**

题目要先求出工资最少的员工的 salary,然后用它去员工表里找

```
SELECT last_name, job_id, salary
FROM   employees
WHERE  salary =
                (SELECT MIN(salary)
                 FROM   employees);
```

下面一个不相关子查询的题目稍微难一点，不会也没关系：

1. **查询与 141 号或 174 号员工的 manager_id 和 department_id 相同的其他员工的 employee_id，manager_id，department_id**

```
SELECT  employee_id, manager_id, department_id
FROM    employees
WHERE   manager_id IN
                  (SELECT  manager_id
                   FROM    employees
                   WHERE   employee_id IN (174,141))
AND     department_id IN
                  (SELECT  department_id
                   FROM    employees
                   WHERE   employee_id IN (174,141))
AND        employee_id NOT IN(174,141);
```

上面的案例用到了 IN，是因为子查询结果不止一条，所以又称之为多行子查询，接下来我们介绍一下多行比较操作符

- IN： 等于列中任意一个值
- ANY： 需要和单行比较操作符（例如 大于 等于等等）一起使用，和子查询返回的某一个值比较
- ALL：需要和单行比较符一起使用，和子查询返回的所有值比较

举例：

1. 返回其它 job_id 中比 job_id 为‘IT_PROG’部门任一工资低的员工的员工号、姓名、job_id 以及 salary

![](https://cdn.nlark.com/yuque/0/2023/png/432603/1703052604344-16263ea3-6287-4f3c-bd93-a95e27261857.png#averageHue=%23f7f7c5&clientId=u589f0ee3-d396-4&from=paste&id=uae0c4526&originHeight=201&originWidth=699&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u987c31b6-19d9-48e5-b8cf-502aba0aa9e&title=)

## 相关子查询

查询各部门中工资比本部门平均工资高的员工的 last_name, salary, department_id

```
SELECT last_name, salary, department_id
FROM employees AS e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employee e2
    WHERE depatrtment_id = e1.`department_id`
);
```

查询每个部门下的部门人数大于 5 的部门名称

```
SELECT department_name
FROM departments AS d
WHERE 5 < (
    SELECT COUNT(*)
    FROM employee e
    WHERE d.depatrtment_id = e.`department_id`
);
```

- 参考资料
  - 《mysql 从入门到精通 - 基础》- 康师傅
  - 《从零蛋开始学习 mysql》 - 小孩子
  - 《mysql8.0 从入门到精通》- 老刘大数据
  - 《mysqltutorial》- website
