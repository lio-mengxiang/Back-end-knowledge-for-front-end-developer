# 测试

* [单元测试]
* [压力测试]
## 简述

> <a name="q-why-write-test"></a> 为什么要写测试? 写测试是否会拖累开发进度?

项目在多人合作的时候, 为了某个功能修改了某个模块的某部分代码, 实际的情况中修改一个地方可能会影响到别人开发的多个功能, 在自己不知情的情况下想要保证自己修改的代码不影响到其他功能, 最简单的办法是通过测试来保证.

```
A
  \
    E
  /   \
B       H
  \   /
    F
  / 
C
  \
    G
  / 
D
```

如上述情况, ABCD 是逻辑层, EFGH 等是更低一次层 (比如工具层等), 当你为了功能 A 的 BUG 修改了 H 的代码, 那么实际受影响的功能除了 A 之外还有 BC, 如果你有针对每一个逻辑的测试, 那么修改了 H 的代码之后, 跑一遍测试即可保证对 H 的修改不会影响到 BC (如果有影响, 那么相应的测试会报错). 利用这种特性, 你还可以基于测试去做重构, 在通过原有测试的情况下, 即表明新的重构版本可以替代原有的版本.

而这样的效果, 只有当覆盖率达到了一定程度 (通常是 80% 以上, 90% 以上为最理想) 才能实现, 如果测试的覆盖率低, 无法覆盖到多种情况, 那么测试对你的项目可能是没有用甚至起到反作用的 (让你误以为你的修改没问题而发布等).

写测试是否会拖累开发进度要视具体情况而定. 需要考虑到, 开发进度包含功能和品质两个方面, 单纯写代码的速度不能完全代表开发进度. 测试在适当的情况下可以保证项目的品质从而得到更好的开发进度.

如上述的例子, 在修改功能 A 的 BUG 的时候, 如果你不知道 H 会影响到 BC 又没有测试的话, 那么开发 BC 的同学可能会出现十分经典的 **"昨天还好好的, 今天怎么就不能用了?"** 的情况. 

当然写测试拖累开发进度的情况也是客观存在的, 通常是有以下几种情况:

* 不会写测试
* 过度测试, 不必要的测试
* 为了迎合测试, 而忽略了实际需求


> <a name="q-death-loop"></a> 测试是如何保证业务逻辑中不会出现死循环的?

你可以通过测试来避免坑爹的同事在某些逻辑中写出死循环, 在通常的测试中加上超时的时间, 在覆盖率足够的情况下, 就可以通过跑出超时的测试来排查出现死循环以及低性能的情况.


## 测试方法

### 黑盒测试

黑盒测试 (Black-box Testing), 测试应用程序的功能, 而不是其内部结构或运作. 测试者不需了解代码、内部结构等, 只需知道什么是应用应该做的事, 即当键入特定的输入, 可得到一定的输出. 测试者通过选择`有效输入`和`无效输入`来验证是否正确的输出. 此测试方法可适合大部分的软件测试, 例如集成测试 (Integration Testing) 以及系统测试 (System Testing).

### 白盒测试

白盒测试 (White-box Testing) 测试应用程序的内部结构或运作, 而不是测试应用程序的功能 (即黑盒测试). 在白盒测试时, 以编程语言的角度来设计测试案例. 白盒测试可以应用于单元测试 (Unit Testing)、集成测试 (Integration Testing) 和系统的软件测试流程, 可测试在集成过程中每一单元之间的路径, 或者主系统跟子系统中的测试.


## 单元测试

单元测试 (Unit Testing) 是白盒测试的一种, 用于针对程序模块进行正确性检验的测试工作. 单元 (Unit) 是指**最小可测试的部件**. 在过程化编程中, 一个单元就是单个程序、函数、过程等; 对于面向对象编程, 最小单元就是方法, 包括基类、抽象类、或者子类中的方法.

这里我们着重简述一下关于jest单元测试node和react的方法。
单元测试框架基本原理
-
例如如下的一个测试用例，感受一下基本的样子长啥，我们后面会把其中用到的方法自己实现一个简单版本
```javascript
// 意思是字符串hello是否包含ll
test('测试字符串中是否包含 ll'), () => {
    expect(findStr('hello')).toMatch('ll')
})

function findStr(str){
    return `${str} world`
}
```

我们可以简单的实现一下上面测试用例用到的方法，`test、expect、toMatch`，这样就算掌握了基本的测试框架原理


### test
```javascript
function test(desc, fn){
    try{
        fn();
        console.log(`✅  通过测试用例`)
    }catch{
        console.log(`❌ 没有通过测试用例`)
    }
}
```
### expect、toMatch
```javascript
function expect(ret){
    return {
        toMatch(expRet){
            if(typeof ret === 'string'){ throw Error('') }
            if(!ret.includes(expRet)){ throw Error('') }
        }
    }
}
```

jest基本配置
-
必备工具：
```javascript
$ npm i -D jest babel-jest ts-jest @types/jest
```
参考配置jest.config.js，测试文件均放在tests目录中：
下面的testRegex表示匹配的tests文件夹下的以test或者spec结尾的jsx或者tsx文件
```javascript
module.exports = {
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },
  testRegex: '(/tests/.*|(\\.|/)(test|spec))\\.(jsx?|tsx?)$',
  moduleFileExtensions: ['tsx', 'ts', 'js', 'jsx', 'json', 'node'],
};
```

最后在package.json的scripts中加入
```javascript
{
    test: "jest"
    // 如果要测试覆盖率，后面加上--coverage
    // 如果要监听所有测试文件 --watchAll
}
```

匹配器
-
匹配器（Matchers）是Jest中非常重要的一个概念，它可以提供很多种方式来让你去验证你所测试的返回值。举个例子就明白什么是匹配器了。

这里的匹配器扫一眼即可，大概知道有那么回事，用的时候查你想要的匹配器就行，不用刻意去记忆。

相等匹配，这是我们最常用的匹配规则　　
```javascript
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});
```
在这段代码中 `expact(2 + 2)` 将返回我们期望的结果，通常情况下我们只需要调用`expect`就可以，括号中的可以是一个具有返回值的函数，也可以是表达式。后面的` toBe ` 就是一匹配器。

下面列举一些常用的匹配器：


### 普通匹配器
- toBe：object.is 相当于 ===
```JAVASCRIPT
test('测试加法 3 + 7', () => {
  // toBe 匹配器 matchers object.is 相当于 ===
  expect(10).toBe(10)
})
```

- toEqual：内容相等，匹配内容，不匹配引用
```JAVASCRIPT
test('toEqual 匹配器', () => {
  // toEqual 匹配器 只会匹配内容，不会匹配引用
  const a = { one: 1 }
  expect(a).toEqual({ one: 1 })
})
```
### 与真假有关的匹配器

- 真假
- toBeNull：只匹配 Null
```JAVASCRIPT
test('toBeNull 匹配器', () => {
  // toBeNull
  const a = null
  expect(a).toBeNull()
})
```
toBeUndefined：只匹配 undefined
```javascript
test('toBeUndefined 匹配器', () => {
  const a = undefined
  expect(a).toBeUndefined()
})
```
toBeDefined： 与 toBeUndefined 相反，这里匹配 null 是通过的
```
test('toBeDefined 匹配器', () => {
  const a = null
  expect(a).toBeDefined()
})
```
toBeTruthy：匹配任何 if 语句为 true
```javascript
test('toBeTruthy 匹配器', () => {
  const a = 1
  expect(a).toBeTruthy()
})
```
toBeFalsy：匹配任何 if 语句为 false
```javascript
test('toBeFalsy 匹配器', () => {
  const a = 0
  expect(a).toBeFalsy()
})
```
not：取反
```javascript
test('not 匹配器', () => {
  const a = 1
  // 以下两个匹配器是一样的
  expect(a).not.toBeFalsy()
  expect(a).toBeTruthy()
})
```
### 数字
toBeGreaterThan：大于
```javascript
test('toBeGreaterThan', () => {
  const count = 10
  expect(count).toBeGreaterThan(9)
})
```
toBeLessThan：小于
```javascript
test('toBeLessThan', () => {
  const count = 10
  expect(count).toBeLessThan(12)
})
```
toBeGreaterThanOrEqual：大于等于
```javascript
test('toBeGreaterThanOrEqual', () => {
  const count = 10
  expect(count).toBeGreaterThanOrEqual(10) // 大于等于 10
})
```
toBeLessThanOrEqual：小于等于
```javascript
test('toBeLessThanOrEqual', () => {
  const count = 10
  expect(count).toBeLessThanOrEqual(10) // 小于等于 10
})
```
toBeCloseTo：计算浮点数
```javascript
test('toBeCloseTo', () => {
  const firstNumber = 0.1
  const secondNumber = 0.2
  expect(firstNumber + secondNumber).toBeCloseTo(0.3) // 计算浮点数
})
```
### 字符串
toMatch： 匹配某个特定项字符串，支持正则
```javascript
test('toMatch', () => {
  const str = 'http://www.zsh.com'
  expect(str).toMatch('zsh')
  expect(str).toMatch(/zsh/)
})
```
### 数组
toContain：匹配是否包含某个特定项
```javascript
test('toContain', () => {
  const arr = ['z', 's', 'h']
  const data = new Set(arr)
  expect(data).toContain('z')
})
```
### 异常
toThrow
```javascript
const throwNewErrorFunc = () => {
  throw new Error('this is a new error')
}
test('toThrow', () => {
  // 抛出的异常也要一样才可以通过，也可以写正则表达式
  expect(throwNewErrorFunc).toThrow('this is a new error')
})
```


测试异步代码
-

假设请求函数如下
```jjavascript
const fethUserInfo = fetch('http://xxxx')
```
测试异步代码有好几种方式，我就推荐一种我认为比较常用的方式
```javascript
// fetchData.test.js

// 测试promise成功需要加.resolves方法
test('the data is peanut butter', async () => {
    await expect(fethUserInfo()).resolves.toBe('peanut butter');
});

// 测试promise成功需要加.rejects方法
test('the fetch fails with an error', async () => {
    await expect(fethUserInfo()).rejects.toMatch('error');
});
```

作用域
-

jest提供一个describle函数来分离各个test测试用例，就是把相关的代码放到一类分组中，这么简单，看个例子就懂了。

```javascript
// 分组一
describe('Test xxFunction', () => {
  test('Test default return zero', () => {
      expect(xxFunction()).toBe(0)
  })

  // ...其它test
})

// 分组二
describe('Test xxFunction2', () => {
  test('Pass 3 can return 9', () => {
      expect(xxFunction2(3)).toBe(9)
  })

  // ...其它test
})
```

钩子函数
-
jest中有4个钩子函数

-   beforeAll：所有测试之前执行
-   afterAll：所有测试执行完之后
-   beforeEach：每个测试实例之前执行
-   afterEach：每个测试实例完成之后执行

我们举例来说明为什么需要他们。


在 `index.js` 中写入一些待测试方法

```
export default class compute {
  constructor() {
    this.number = 0
  }
  addOne() {
    this.number += 1
  }
  addTwo() {
    this.number += 2
  }
  minusOne() {
    this.number -= 1
  }
  minusTwo() {
    this.number -= 2
  }
}
```

假如我们要
在 `index.test.js` 中写测试实例

```javascript
import compute from './index'

const Compute = new compute()

test('测试 addOne', () => {
  Compute.addOne()
  expect(Compute.number).toBe(1)
})

test('测试 minusOne', () => {
  Compute.minusOne()
  expect(Compute.number).toBe(0)
})
```

- 这里两个测试实例相互之间影响了，共用了一个computet实例，我们可以将`const Compute = new compute()`放在beforEach里面就可以解决了，每次测试实例之前先重新new compute

- 同理，你想在每个test测试完毕后单独运行什么可以放入到`afterEach`中

我们接着看一下什么情况下使用`beforeAll`，假如我们测试数据库数据是否保存正确
- 我们在测试最开始,也就是 `beforeAll`生命周期里， 新增1条数据到数据库里
- 测试完后，也就是 `afterAll`周期里， 删除之前添加的数据
- 最后利用全局作用域 `afterAll` 确认数据库是否还原成初始状态

这里说到

```javascript
// 模拟数据库
const userDB = [
  { id: 1, name: '小明' },
  { id: 2, name: '小花' },
]

// 新增数据
const insertTestData = data => {
  // userDB,push数据
}

// 删除数据
const deleteTestData = id => {
  // userDB,delete数据
}

// 全部测试完
afterAll(() => {
  console.log(userDB)
})

describe('Test about user data', () => {

  beforeAll(() => {
      insertTestData({ id: 99, name: 'CS' })
  })
  afterAll(() => {
      deleteTestData(99)
  })

})
```
## jest里的Mock

### **为什么要使用Mock函数？**

在项目中，经常会碰见A模块掉B模块的方法。并且，在单元测试中，我们可能并不需要关心内部调用的方法的执行过程和结果，只想知道它是否被正确调用即可，甚至会指定该函数的返回值。此时，就需要mock函数了。

Mock函数提供的以下三种特性，在我们写测试代码时十分有用：

-   捕获函数调用情况
-   设置函数返回值
-   改变函数的内部实现


### jest.fn()

`jest.fn()`是创建Mock函数最常用的方式。
```javascript
test('测试jest.fn()', () => {
  let mockFn = jest.fn();
  let result = mockFn(1);

  // 断言mockFn被调用
  expect(mockFn).toBeCalled();
  // 断言mockFn被调用了一次
  expect(mockFn).toBeCalledTimes(1);
  // 断言mockFn传入的参数为1
  expect(mockFn).toHaveBeenCalledWith(1);
})
```

`jest.fn()`所创建的Mock函数还可以**设置返回值**，**定义内部实现**或**返回`Promise`对象**。

```javascript
test('测试jest.fn()返回固定值', () => {
  let mockFn = jest.fn().mockReturnValue('default');
  // 断言mockFn执行后返回值为default
  expect(mockFn()).toBe('default');
})

test('测试jest.fn()内部实现', () => {
  let mockFn = jest.fn((num1, num2) => {
    return num1 * num2;
  })
  // 断言mockFn执行后返回100
  expect(mockFn(10, 10)).toBe(100);
})

test('测试jest.fn()返回Promise', async () => {
  let mockFn = jest.fn().mockResolvedValue('default');
  let result = await mockFn();
  // 断言mockFn通过await关键字执行后返回值为default
  expect(result).toBe('default');
  // 断言mockFn调用后返回的是Promise对象
  expect(Object.prototype.toString.call(mockFn())).toBe("[object Promise]");
})
```



### 2. jest.mock()

`fetch.js`文件夹中封装的请求方法可能我们在其他模块被调用的时候，并不需要进行实际的请求（请求方法已经通过单测或需要该方法返回非真实数据）。此时，使用`jest.mock(）`去mock整个模块是十分有必要的。

下面我们在`src/fetch.js`的同级目录下创建一个`src/events.js`。

```javascript
import fetch from './fetch';

export default {
  async getPostList() {
    return fetch.fetchPostsList(data => {
      console.log('fetchPostsList be called!');
      // do something
    });
  }
}
```

```javascript
import events from '../src/events';
import fetch from '../src/fetch';

jest.mock('../src/fetch.js');

test('mock 整个 fetch.js模块', async () => {
  expect.assertions(2);
  await events.getPostList();
  expect(fetch.fetchPostsList).toHaveBeenCalled();
  expect(fetch.fetchPostsList).toHaveBeenCalledTimes(1);
});
```

在测试代码中我们使用了`jest.mock('../src/fetch.js')`去mock整个`fetch.js`模块。如果注释掉这行代码，执行测试脚本时会出现以下报错信息

![]()

从这个报错中，我们可以总结出一个重要的结论：

> 在jest中如果想捕获函数的调用情况，则该函数必须被mock或者spy！

### **3. jest.spyOn()**

`jest.spyOn()`方法同样创建一个mock函数，但是该mock函数不仅能够捕获函数的调用情况，还可以正常的执行被spy的函数。实际上，`jest.spyOn()`是`jest.fn()`的语法糖，它创建了一个和被spy的函数具有相同内部代码的mock函数。

![]()

上图是之前`jest.mock()`的示例代码中的正确执行结果的截图，从shell脚本中可以看到`console.log('fetchPostsList be called!');`这行代码并没有在shell中被打印，这是因为通过`jest.mock()`后，模块内的方法是不会被jest所实际执行的。这时我们就需要使用`jest.spyOn()`。

```
// functions.test.js

import events from '../src/events';
import fetch from '../src/fetch';

test('使用jest.spyOn()监控fetch.fetchPostsList被正常调用', async() => {
  expect.assertions(2);
  const spyFn = jest.spyOn(fetch, 'fetchPostsList');
  await events.getPostList();
  expect(spyFn).toHaveBeenCalled();
  expect(spyFn).toHaveBeenCalledTimes(1);
})
```

执行`npm run test`后，可以看到shell中的打印信息，说明通过`jest.spyOn()`，`fetchPostsList`被正常的执行了。


快照
-

快照就是对你对比的数据会存一份副本，啥意思呢，我们举个例子：

这是`index.js`

```javascript
export const data2 = () => {
  return {
    name: 'zhangsan',
    age: 26,
    time: new Date()
  }
}
```
在 `index.test.js` 中写入一些测试实例

```javascript
import { data2 } from "./index"

it('测试快照 data2', () => {
  expect(data2()).toMatchSnapshot({
    name: 'zhangsan',
    age: 26,
    time: expect.any(Date) //用于声明是个时间类型，否则时间会一直改变，快照不通过
  })
})
```

-   `toMatchSnapshot`会将参数将快照进行匹配
-   `expect.any(Date)` 用于匹配一个时间类型

执行`npm run test`会生成一个`__snapshots__`文件夹，里面是生成的快照，当你修改一下测试代码时，会提示你，快照不匹配。
如果你确定你需要修改，按 u 键，即可更新快照。这用于UI组件的测试非常有用。

React的BDD单测
-

接下来我们看下react代码如何进行测试，用一个很小的例子来说明。

案例中引入了enzyme。*Enzyme* 来自 airbnb 公司，是一个用于 React 的 JavaScript 测试工具，方便你判断、操纵和历遍 React Components 输出。

我们达成的目的是检测：
- 用户进入首页，看到两个按钮，分别是counter1和counter2
- 点击counter1，就能看到两个按钮的文字部分分别是"counter1"和"counter2"

react代码如下
```javascript
import React from 'react';
function Counter(){
    return (
        <ul>
            <li>
                <button id='counter1' className='button1'>counter1</button>
            </li>
            <li>
                <button id='counter2' className='button2'>counter2</button>
            </li>
        </ul>
    )
}
```

单测的文件：

```JAVASCRIPT
import Counter from xx;
import { mount } from 'enzyme';

describle('测试APP',() => {
    test('用户进入首页，看到两个按钮，分别是counter1和counter2,并且按钮文字也是counter1和counter2',()=>{
        const wrapper = mount(<Counter />);
        const button = wrapper.find('button');
        except(button).toHaveLength(2);
        except(button.at(0).text()).toBe('counter1');
        except(button.at(1).text()).toBe('counter2');
    })
})
```
### 覆盖率

测试覆盖率 (Test Coverage) 是指代码中各项逻辑被测试覆盖到的比率, 比如 90% 的覆盖率, 是指代码中 90% 的情况都被测试覆盖到了.

覆盖率通常由四个维度贡献:

* 行覆盖率 (line coverage) 是否每一行都执行了？
* 函数覆盖率 (function coverage) 是否每个函数都调用了？
* 分支覆盖率 (branch coverage) 是否每个if代码块都执行了？
* 语句覆盖率 (statement coverage) 是否每个语句都执行了？

覆盖率在jest框架下，只要带上coverage参数就可以看到。



### 黑盒测试工具

黑盒级别的基准测试, 则推荐 [Apache ab](https://httpd.apache.org/docs/2.4/programs/ab.html) 以及 [wrk](https://github.com/wg/wrk) 等, 例如执行:

```
ab -n 100 -c 10 https://ele.me/
```

可以得到如下的详细数据:

```
Server Software:        Tengine/2.1.1
Server Hostname:        ele.me
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES256-GCM-SHA384,2048,256

Document Path:          /
Document Length:        284 bytes

Concurrency Level:      10
Time taken for tests:   1.775 seconds
Complete requests:      100
Failed requests:        0
Non-2xx responses:      100
Total transferred:      62400 bytes
HTML transferred:       28400 bytes
Requests per second:    56.33 [#/sec] (mean)
Time per request:       177.511 [ms] (mean)
Time per request:       17.751 [ms] (mean, across all concurrent requests)
Transfer rate:          34.33 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       88  116  26.0    104     234
Processing:    33   55  39.6     47     394
Waiting:       33   54  39.0     46     394
Total:        124  171  48.1    152     491

Percentage of the requests served within a certain time (ms)
  50%    152
  66%    184
  75%    193
  80%    199
  90%    224
  95%    242
  98%    288
  99%    491
 100%    491 (longest request)
```

与前者相比, ab 等工具可以设置规模以及并发情况. 在比规模不大/需求不复杂的情况下, ab 以及 wrk 也可以用于做压力测试.


## 压力测试

压力测试 (Stress testing), 是保证系统稳定性的一种测试方法. 通过预估系统所需要承载的 QPS, TPS 等指标, 然后通过如 [Jmeter](http://jmeter.apache.org/) 等压测工具模拟相应的请求情况, 来验证当前应能能否达到目标.

对于比较重要, 流量较高或者后期业务量会持续增长的系统, 进行压力测试是保证项目品质的重要环节. 常见的如负载是否均衡, 带宽是否合理, 以及磁盘 IO 网络 IO 等问题都可以通过比较极限的压力测试暴露出来.



