# 安全

*  Crypto (加密)
*  HTTPS
*  XSS
*  CSRF
*  中间人攻击
*  Sql/Nosql 注入攻击


## Crypto

Node.js 的 `crypto` 模块封装了诸多的加密功能, 包括 OpenSSL 的哈希、HMAC、加密、解密、签名和验证函数等.

Node.js 的加密貌似有点问题, 某些算法算出来跟别的语言 (比如 Python) 不一样. 具体情况还在整理中 (时间不定), 欢迎补充.

简单说几种常用的crypto的业务中的使用
```javascript
const crypto = require('crypto');
const md5 = crypto.createHash('md5');//返回哈希算法
const md5Sum = md5.update('hello');//指定要摘要的原始内容,可以在摘要被输出之前使用多次update方法来添加摘要内容
const result = md5Sum.digest('hex');//摘要输出，在使用digest方法之后不能再向hash对象追加摘要内容。
console.log(result);
```
上面这种散列(哈希)算法，也可以加盐，升级一下破解难度，也就是HMAC算法。

这里简单写一下非对称加密rsa相关代码，面试常考的https就跟这个有关。

首先用openssl创建ras的公钥和私钥
```javascript
openssl rsa -in ras_private.key -puhout -out rsa_public.key
```

用起来，看如何用公钥加密，私钥解密
```javascript
const crypto = require('crypto');
const fs = require('fs');
const key = fs.readFileSync(path.join(_dirname, 'rsa_private.key'));
const cert = fs.readFileSync(path.join(_dirname, 'rsa_public.key'));
// 公钥加密
const secret = crypto.publicEncrypt(cert, buffer);
// 私钥揭秘
crypto.privateDecrypt(key, secret)
```

顺便还可以问下面试者，什么是签名，签名有啥用。

在网络中，私钥的拥有者可以在一段数据被发送之前先对数据进行签名得到一个签名 通过网络把此数据发送给数据接收者之后，数据的接收者可以通过公钥来对该签名进行验证,以确保这段数据是私钥的拥有者所发出的原始数据，且在网络中的传输过程中未被修改

## HTTPS

### https解决http的哪些问题？https是如何解决
#### 机密性
- http使用明文传输，使得通信过程可能被窃听。
- https在交换密钥阶段使用非对称加密，建立通信后使用对称加密。所以数据是经过加密的。
#### 完整性
- 由于HTTP协议无法验证报文的完整性，因此数据在传输过程中可能已经遭到中间人攻击，导致报文被篡改。
- 摘要算法是常说的散列函数和哈希函数，可以为数据生成独一无二的指纹，用于校验数据的完整性。客户端在发送数据之前通过摘要算法算出明文的的指纹，发送的时候明文和指纹会一起发送，校验的时候会先把明文用相同的摘要算法算成指纹，对比前后指纹是否相同，看数据是否被篡改了。
#### 身份认证
- HTTP协议中不会对请求和应答的双方进行身份确认，所以身份有可能遭遇伪装。HTTPS的SSL不仅提供加密服务，还提供证书，用来确定通信方。证书是由值得信任的第三方机构办法，伪造证书异常困难，所以只要能够确认通信方持有的证书，则可以确认对方身份。

#### https大致流程

- 客户端发起一个http请求，告诉服务器自己支持哪些hash算法。

- 服务端把自己的信息以数字证书的形式返回给客户端（证书内容有密钥公钥，网站地址，证书颁发机构，失效日期等）。证书中有一个公钥来加密信息，私钥由服务器持有。

- 验证证书的合法性

客户端收到服务器的响应后会先验证证书的合法性（证书中包含的地址与正在访问的地址是否一致，证书是否过期）。

- 客户端使用伪随机数生成器生成加密所使用的对称密钥，然后用证书的公钥加密这个对称密钥，发给Server。
- 服务端使用自己的私钥（private key）解密这个消息，得到对称密钥。至此，Client和Server双方都持有了相同的对称密钥。

## XSS

XSS精品文章参见美团技术部出品（https://tech.meituan.com/2018/09/27/fe-security.html）

我们这里提几个XSS面试题

### 1、 过滤 Html 标签能否防止 XSS? 请列举不能的情况?
答案肯定是不能的，比如我们常见的转义函数如下：
```javascript
const escapeHTML = str =>
  str.replace(
    /[&<>'"]/g,
    tag =>
      ({
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;',
        "'": '&#39;',
        '"': '&quot;'
      }[tag] || tag)
  );
```
为啥防不住呢，如下的案例：
```html
<img src="javascript:alert('xss')">
```
这个src里的`javascript:`不会被转义，照样这个xss会执行。

用户除了上传

```html
<script>alert('xss');</script>
```

还可以使用图片 url 等方式来上传脚本进行攻击

```html
<table background="javascript:alert(/xss/)"></table>
<img src="javascript:alert('xss')">
```

还可以使用各种方式来回避检查, 例如空格, 回车, Tab

```html
<img src="javas cript:
alert('xss')">
```

还可以通过各种编码转换 (URL 编码, Unicode 编码, HTML 编码, ESCAPE 等) 来绕过检查

```
<img%20src=%22javascript:alert('xss');%22>
<img src="javascrip&#116&#58alert(/xss/)">
```

## CSRF

跨站请求伪造 (Cross-Site Request Forgery, CSRF, https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet) 是一种伪造跨站请求的攻击方式. 例如利用你在 A 站 (攻击目标) 的 cookie / 权限等, 在 B 站 (恶意/钓鱼网站) 拼装 A 站的请求.

比如 Q 君是某论坛管理员. 已知这个论坛 A 删除的接口是 post 到某个地址, 并指定一个帖子的 id.  那么我可以在自己的博客 B 上组织一个 CSRF 请求. 然后诱使 Q 君来访问我的博客. 就可以在 Q 君不知情的情况下删除掉我想删的某个帖子.

钓鱼方式包括但不限于公开网站 (xss), 攻击者的恶意网站, email 邮件, 微博, 微信, 短信等及时消息.

同源策略是最早用于防止 CSRF 的一种方式, 即关于跨站请求 (Cross-Site Request) 只有在同源/信任的情况下才可以请求. 但是如果一个网站群, 在互相信任的情况下, 某个网站出现了问题:

```
a.public.com
b.public.com
c.public.com
...
```

以上情况下, 如果 c.public.com 上没有预防 xss 等情况, 使得攻击者可以基于此站对其他信任的网站发起 CSRF 攻击.

另外同源策略主要是浏览器来进行验证的, 并且不同浏览器的实现又各自不同, 所以在某些浏览器上可以直接绕过, 而且也可以直接通过短信等方式直接绕过浏览器.

预防:

1. A 站 (预防站) 检查 http 请求的 header 确认其 origin
2. 检查 CSRF token

### 1.同源检查 

通过检查来过滤简单的 CSRF 攻击, 主要检查一下两个 header:

* Origin Header
* Referer Header

### 2.CSRF token 

简单来说, 对需要预防的请求, 通过特别的算法生成 token 存在 session 中, 然后将 token 隐藏在正确的界面表单中, 正式请求时带上该 token 在服务端验证, 避免跨站请求.


## 中间人攻击

中间人 (Man-in-the-middle attack, MITM) 是指攻击者与通讯的两端分别创建独立的联系, 并交换其所收到的数据, 使通讯的两端认为他们正在通过一个私密的连接与对方直接对话, 但事实上整个会话都被攻击者完全控制. 在中间人攻击中, 攻击者可以拦截通讯双方的通话并插入新的内容.

目前比较常见的是在公共场所放置精心准备的免费 wifi, 劫持/监控通过该 wifi 的流量. 或者攻击路由器, 连上你家 wifi 攻破你家 wifi 之后在上面劫持流量等.

对于通信过程中的 MITM, 常见的方案是通过 PKI / TLS 预防, 及时是通过存在第三方中间人的 wifi 你通过 HTTPS 访问的页面依旧是安全的. 而 HTTP 协议是明文传输, 则没有任何防护可言.

不常见的还有强力的互相认证, 你确认他之后, 他也确认你一下; 延迟测试, 统计传输时间, 如果通讯延迟过高则认为可能存在第三方中间人; 等等.

## SQL 注入

注入攻击是指当所执行的一些操作中有部分由用户传入时, 用户可以将其恶意逻辑注入到操作中. 当你使用 eval, new Function 等方式执行的字符串中有用户输入的部分时, 就可能被注入攻击. 上文中的 XSS 就属于一种注入攻击. 前面的章节中也提到过 Node.js 的 child_process.exec 由于调用 bash 解析, 如果执行的命令中有部分属于用户输入, 也可能被注入攻击.

### SQL

Sql 注入是网站常见的一种注入攻击方式. 其原因主要是由于登录时需要验证用户名/密码, 其执行 sql 类似:

```sql
SELECT * FROM users WHERE usernae = 'myName' AND password = 'mySecret';
```

其中的用户名和密码属于用户输入的部分, 那么在未做检查的情况下, 用户可能拼接恶意的字符串来达到其某种目的, 例如上传密码为 `'; DROP TABLE users; --` 使得最终执行的内容为:

```sql
SELECT * FROM users WHERE usernae = 'myName' AND password = ''; DROP TABLE users; --';
```

防止的手段目前普遍都采用参数化查询。比如mysql，是本身就有的功能。

在使用参数化查询的情况下，数据库服务器不会将参数的内容视为SQL指令的一部份来处理，而是在数据库完成SQL指令的编译后，才套用参数运行，因此就算参数中含有指令，也不会被数据库运行。

我们来实际举一个例子：

假设我们有一个简单的登录表单，其中包含用户名和密码字段，并且我们需要在数据库中验证用户是否存在。我们可以使用以下查询语句来验证用户：

sql
Copy code
SELECT * FROM users WHERE username = '$username' AND password = '$password'
其中，$username和$password是从表单输入中获得的变量。这个查询语句将用户输入的用户名和密码作为字符串直接插入到查询语句中，这种方式容易受到SQL注入攻击的影响。

现在假设攻击者在用户名输入框中输入了一个恶意的值，如下所示：

```
' OR 1=1--
```
在这种情况下，查询语句将变成：

```sql
SELECT * FROM users WHERE username = '' OR 1=1--' AND password = '$password'
```
这个查询语句将返回所有用户记录，因为它使用了OR 1=1语句，这个条件将始终为真。攻击者现在可以通过尝试各种密码来登录任何用户账户。

为了避免这种情况，我们可以使用参数化查询，通过占位符将输入数据与查询语句分离，例如：

```sql
SELECT * FROM users WHERE username = ? AND password = ?
```
然后，在查询执行之前，我们将输入数据传递给数据库引擎，例如：

```sql
$username = 'john';
$password = 'password123';
# 创建查询语句
const ss = ("SELECT * FROM users WHERE username = ? AND password = ?");
# 绑定查询参数
bind_param(ss, $username, $password);
# 执行查询
execute();
```


在这种情况下，如果攻击者输入了一个恶意的值，例如 ' OR 1=1--，那么它仍然会被视为一个普通的字符串值，并作为参数传递给查询语句，而不会被视为SQL代码。这样，即使攻击者使用了SQL注入攻击，也不会对数据库造成任何伤害。




