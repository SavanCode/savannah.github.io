---
title: mockjs模拟数据
top: false
cover: false
toc: true
mathjax: true
date: 2021-02-20 11:03:02
password:
summary: mockjs模拟数据
tags: [mock.js,Vue]
categories: mock.js
---

## 了解mockjs

实际开发中，前后端分离，前端需要后端的接口去完成页面的渲染，但是并不能等到后端成员写完接口再开始进行测试。大部分情况下，前后端需要同时进行开发。因此便需要**mockjs生成随机数据 & 拦截 Ajax 请求**来进行后端接口模拟。

(后面的例子会使用到Axios,axios 是一个基于 Promise 用于浏览器和 nodejs 的 HTTP 客户端，我们后续需要用来发送 http 请求)

![](mockjs/image-20210226005153551.png)

[使用文档](https://github.com/nuysoft/Mock/wiki/Getting-Started)


官网地址：http:*//mockjs.com/*

github 地址：https:*//github.com/nuysoft/Mock*

## Vue的引入Mockjs - demo1

**完整代码 git: mockjsDemo1**

```sh
npm install mockjs
yarn add mockjs
//下面的例子要用axios
yarn add axios
```

- 在src路径下创建mock.js文件

- 在main.js引入mock.js文件

```js
  require('./mock')
//或者
import './mock'// 这里不用写 ./mock.js
```

- 在mock.js文件中写

```js
  //引入mockjs
  const Mock = require('mockjs')
  // 获取 mock.Random 对象
  const Random = Mock.Random;
  //使用mockjs模拟数据
  Mock.mock('/api/data', (req, res) => {//当post或get请求到/api/data路由时Mock会拦截请求并返回上面的数据
      let list = [];
      for(let i = 0; i < 30; i++) {
          let listObject = {
              title: Random.csentence(5, 10),//随机生成一段中文文本。
              company: Random.csentence(5, 10),
              attention_degree: Random.integer(100, 9999),//返回一个随机的整数。
              photo: Random.image('114x83', '#00405d', '#FFF', 'Mock.js')
          }
          list.push(listObject);
      }
      return {
          data: list
      }
  })

//上面也可以写成
Mock.mock('https://xxxxxxxxxx',{
    'list|5-10': [{// 直接5-10的程度生成
        'id|+1': 1,
        'username':'@cname',
        'email': '@email',
        'price': '@integer',
        'gender': '@boolean'
    }]
})
```
  - 在Vue中设置

```js
      mounted:function() {
            axios.get('/api/data').then(res => {//get()中的参数要与mock.js文件中的Mock.mock()配置的路由保持一致
              this.data = res.data.data;
              console.log(res.data);//在console中看到数据
              this.tableData=res.data.data
            }).catch(res => {
              alert('wrong');
            })
          },  
```

上面的运行成功了~ 那么就可以开心的用mockjs了 ~~下面的只是更换所需的功能

## [Mock函数的解释](https://github.com/nuysoft/Mock/wiki/Mock.mock())

Mock.mock(rurl? rtype? template|function(options))   生成模拟数据

- rurl - 可选
    表示需要拦截的 URL，可以是 '字符串' 或 '正则'。例如：
``` js
 'www.xxx.com/list.json'
  /\/www\.xxx\.com\/list\.json/
```
- rtype - 可选
    表示需要拦截的 Ajax 请求方法。例如： GET、POST、PUT、DELETE 等
- template - 可选
    表示数据模板，可以是 '对象' 或 '字符串'。例如：{ 'data|1-10': [{}] }	 或  '@EMAIL'

- function(options) - 可选
    表示用于生成响应数据的函数。
    options 指向本次请求的 Ajax 选项集，包含： url type body  3 个属性

## mockjs 数据生成语法

稍微看看吧， 不想看可以直接按照例子走就完事儿

### mock.js 的语法规范包含 2 部分
  1.数据模板定义规则(Data Template Definition, DTD)
  2.数据占位符定义规则(Data Placeholder Definition, DPD)

### DTD 规范
每个属性由 3 部分组成：属性名、生成规则、属性值。

**'name|rule': value**

- name - 属性名
- rule - 生成规则
- value - 属性值

#### 注意点

1.属性名和生成规则，之间有一个 '|'
2.生成规则是 '可选的'
3.生成规则有 '7' 种格式：
- 1>min-max
- 2>count
- 3>min-max.dmin-dmax
- 4>min-max.dcount
- 5>count.dmin-dmax
- 6>count.dcount
- 7>+step

4.生成规则的含义，依赖 '属性值类型' 来确定(意思是：生成规则，根据属性值 '类型' 的不同，会生成不同的模拟数据) 
5.属性值中可以含有 '@占位符'
6.属性值，指定了最终值的 '初始值' 和 '类型'

### DPD 规范
占位符，只是在 '属性值' 中占个位置，并不会出现在生成的属性值中。

- @占位符
- @占位符(参数[, 参数])

#### 注意点

- 1.用 '@' 来标识其后的字符串为 '占位符'
- 2.占位符引用的是 'Mock.Random' 中的方法
- 3.通过 Mock.Random.extend() 来 '自定义' 占位符
- 4.'占位符' 也 '可以引用'  '数据模板' 的属性
- 5.'占位符' 会 '优先' 引用 '数据模板' 的属性
- 6.'占位符' 支持 '相对路径' 和 '绝对路径'

#### 例子

注意下面的参数都是可选的哈

```js
"base": {
    "range": "@range(3)",// [1,2,3]
    "range": "@range(3, 7)",// [3,4,5,6]
    "rangeWithGap": "@range(1, 10, 3)",//start-1，stop-10，step-3 [1,4,7]
    "string": "@string(7, 20)", //输出7-20个字符长度的字符串
     "string": "@string('lower', 5)",//Random.string(pool, length)或者(pool,min,max) pool:'lower/upper/number/symbol'
    "character": "@character('abcde')",//随机从abcde中选一个字母
    //Random.character('lower/upper/number/symbol') 
    "float": "@float(60, 100)",//60 到 100 的浮点数 Random.float(min, max, dmin, dmax) dmin- 小数部分单个位数最小值 dmax 小数部分单个位数最大
    "integer": "@integer(60, 100)",//60 到 100 的整数
    "natural": "@natural(60, 100)",//60 到 100 的自然数
    "boolean": "@boolean" //boolean 类型 true,false
  },
  <!--时间类型-->
  "date": {
    "date": "@date",//1982-07-20
    "time": "@time",// 11:21:39
    "datetime": "@datetime",// 1972-12-16 02:04:24
    "now": "@now" //当前时间 2018-07-17 18:19:29  Mock.Random.now() Ranndom.now(format)
      //Mock.Random.now('month') => Ranndom.now(unit) unit = year、month、week、day、hour、minute、second、week
  },
  <!--图片-->
  "image": {
    "image": "@image('200x100', '#50B347', '#FFF', 'EasyMock')" // https://dummyimage.com/200x100/50B347/FFF&text=EasyMock 尺寸 背景 文字颜色 提示信息
  },
  <!--颜色系列-->
  "color": {
    "color": "@color", //16进制颜色值#79e0f2
    "hex": "@hex", // #f2e179
    "rgb": "@rgb", //rgb(189, 121, 242)
    "rgba": "@rgba",//rgba(189, 121, 242, .7)
    "hsl": "@hsl" //hsl(136, 82, 71)
  },
  <!--文案类-->
  "text": {
    "paragraph": "@paragraph(1, 3)",// 随机段落
    "sentence": "@sentence(3, 5)",// 随机句子
    "word": "@word(3, 5)",// 随机3-5个字母
    "title": "@title(3, 5)",// 随机3-5个单词的title
    "cparagraph": "@cparagraph(1, 3)",
    "csentence": "@csentence(3, 5)",
    "cword": "@cword('零一二三四五六七八九十', 5, 7)",
    "ctitle": "@ctitle(3, 5)"
  },
  <!--姓名-->
  "name": {
    "first": "@first",//姓
    "last": "@last",//名
    "name": "@name",//姓名
    "cfirst": "@cfirst",
    "clast": "@clast",
    "cname": "@cname"
  },
  <!--网站-->
  "web": {
    "url": "@url",//url地址
    "domain": "@domain",//域名
    "protocol": "@protocol",//协议
    "tld": "@tld",
    "email": "@email",//邮箱
    "ip": "@ip"//ip地址
  },
  <!--地址-->
  "address": {
    "region": "@region",//区域
    "province": "@province",//省
    "city": "@city(true)",//市
    "county": "@county(true)",//区 带true则携带上级
    "zip": "@zip"
  },
  "helper": {
    "capitalize": "@capitalize('hello')",
    "upper": "@upper('hello')",//全大写单词
    "lower": "@lower('HELLO')",//全小写单词
    "pick": "@pick(['a', 'e', 'i', 'o', 'u'])", //随机选择一个字母
    "shuffle": "@shuffle(['a', 'e', 'i', 'o', 'u'])" //打乱数组顺序
  },
  "miscellaneous": {
    "id": "@id",//身份证id
    "guid": "@guid",//生成32位的随机id
    "increment": "@increment(1)"//自增数，阶度为1
  }
```

#### date format 

日期的模板 其实这个很多地方也都通用 airbnb那个以及moment js 都是这个

| format | Description                                              | **Example**  |
| ------ | -------------------------------------------------------- | ------------ |
| yyyy   | A full numeric representation of a year, 4 digits        | 1999 or 2003 |
| yy     | A two digit representation of a year                     | 99 or 03     |
| y      | A two digit representation of a year                     | 99 or 03     |
| MM     | Numeric representation of a month, with leading zeros    | 01 to 12     |
| M      | Numeric representation of a month, without leading zeros | 1 to 12      |
| dd     | Day of the month, 2 digits with leading zeros            | 01 to 31     |
| d      | Day of the month without leading zeros                   | 1 to 31      |
| HH     | 24-hour format of an hour with leading zeros             | 00 to 23     |
| H      | 24-hour format of an hour without leading zeros          | 0 to 23      |
| hh     | 12-hour format of an hour without leading zeros          | 1 to 12      |
| h      | 12-hour format of an hour with leading zeros             | 01 to 12     |
| mm     | Minutes, with leading zeros                              | 00 to 59     |
| m      | Minutes, without leading zeros                           | 0 to 59      |
| ss     | Seconds, with leading zeros                              | 00 to 59     |
| s      | Seconds, without leading zeros                           | 0 to 59      |
| SS     | Milliseconds, with leading zeros                         | 000 to 999   |
| S      | Milliseconds, without leading zeros                      | 0 to 999     |
| A      | Uppercase Ante meridiem and Post meridiem                | AM or PM     |
| a      | Lowercase Ante meridiem and Post meridiem                | am or pm     |
| T      | Milliseconds, since 1970-1-1 00:00:00 UTC                | 759883437303 |

## [数据生成助手](http://mockjs.com/0.1/editor.html#help)

## 数据模板生成模板 - demo2

http://mockjs.com/examples.html


## mockJs 的模拟get请求

```js
//vue
<button @click="getGoodsList">获取商品列表</button>
 methods: {
    async getGoodsList() {
      const { data: res } = await this.$http.get('/api/goodslist')
      console.log(res)
    }
 }
//mockjs
Mock.mock('/api/goodslist', 'get', {
  status: 200,
  message: '获取商品列表成功！',
  'data|5-10': [
    {
      id: '@increment(1)', // 自增的Id值
      // 'id|+1': 0, // 这也是在模拟一个自增长的 Id 值
      name: '@cword(2, 8)', // 随机生成中文字符串
      price: '@natural(2, 10)', // 自然数
      count: '@natural(100, 999)',
      img: '@dataImage(78x78)', // 指定宽高图片
      'answer|1': ["是", "否"]
    }
  ]
})
```

## mockJs 的模拟post请求

```js
//vue
<button @click="addGoods">添加商品</button>
  methods: { 
    async addGoods() {
      const { data: res } = await this.$http.post('/api/addgoods', {
        name: '菠萝',
        price: 8,
        count: 550,
        img: '',
        "user|1-3": [{   // 随机生成1到3个数组元素
            'name': '@cname',  // 中文名称
            'id|+1': 88,    // 属性值自动加 1，初始值为88
            'age|18-28': 0,   // 18至28以内随机整数, 0只是用来确定类型
            'birthday': '@date("yyyy-MM-dd")',  // 日期 
        },
      }) 
      console.log(res)
    }
  }
//mockjs
Mock.mock('/api/addgoods', 'post', function(option) {
  // 这里的 option 是请求相关的参数
  console.log(option)

  //一般的默认的return
  // return{
  //   status:200,
  //   message:'商品添加成功'
  // }

  //这里如果你要使用mock返回信息的话 就需要这样
  return Mock.mock({
    status: 200,
    message: '@cword(2,5)'
  })
})
```

## 删加增减的功能练习

**注意这里的设置会有不同这里的完整代码 github_demo2**

上面的都属于基本的简单实用 ~~**下面咋们看看封装版本**

## 封装mock模板!! - admin-ui

(下面的例子 基本完成了mock的基本设置 axios的基本设置 然后还有做axios封装)

### 目的

为了统一可以统一管理和集中控制数据模拟接口，我们对 mock 模块进行了封装，可以方便的定制模拟接口的统一开关和个体开关

### 文件结构

```
|- mock 
 |--modules
   |--- index.js：模拟接口模块聚合文件
   |--- login.js：登录相关的接口模拟
   |--- user.js：用户相关的接口模拟
   |--- menu.js：菜单相关的接口模拟
```

**代码内容都是admin-ui里面的!!!!**

#### 封装之前

```js
//简单的搭建沟通
import Mock from 'mockjs'

//前面的link可以不写
Mock.mock('http://localhost:8080/login',{
    data:{
        'token':'123159753456789'
        //其他数据
    }
})

Mock.mock('http://localhost:8080/user',{
    'name':'@name',//随机生成名字
    'email':'@email',//随机生成
    'age|1-10':5
})

Mock.mock('http://localhost:8080/menu',{
    'id':'@increment',//随机生成名字
    'name':'@menu',//随机生成
    'order|10-20':12
}) 
```

#### 封装开始!!index.js

```js
import Mock from 'mockjs'
import * as login from './modules/login'
import * as user from './modules/user'
import * as menu from './modules/menu'

// 1. 开启/关闭[业务模块]拦截, 通过调用fnCreate方法[isOpen参数]设置.
// 2. 开启/关闭[业务模块中某个请求]拦截, 通过函数返回对象中的[isOpen属性]设置.
fnCreate(login, true)//现在就暂时只有三个部分
fnCreate(user, true)
fnCreate(menu, true)

/**
 * 创建mock模拟数据
 * @param {*} mod 模块
 * @param {*} isOpen 是否开启?
 */
function fnCreate (mod, isOpen = true) {
  if (isOpen) {
    for (var key in mod) {
      ((res) => {
        if (res.isOpen !== false) {
          Mock.mock(new RegExp(res.url), res.type, (opts) => {
            opts['data'] = opts.body ? JSON.parse(opts.body) : null
            delete opts.body
            console.log('\n')
            console.log('%cmock拦截, 请求: ', 'color:blue', opts)
            console.log('%cmock拦截, 响应: ', 'color:blue', res.data)
            return res.data
          })
        }
      })(mod[key]() || {})
    }
  }
}
```

#### login.js

关于练习里面的login练习, 只是利用了session storage，实际上可以利用Vuex。 这里详细在另一篇文章

```js
// 登录接口
export function login () {
  return {
    // isOpen: false,
    url: 'http://localhost:8080/login',
    type: 'get',
    data: {
      'msg': 'success',
      'code': 0,
      'data': {
        'token': '4344323121398'
        // 其他数据
      }
    }
  }
}
```

#### user.js

```js
// 获取用户信息
export function getUser () {
  return {
    // isOpen: false,
    url: 'http://localhost:8080/user',
    type: 'get',
    data: {
      'msg': 'success',
      'code': 0,
      'data': {
        'id': '@increment', 
        'name': '@name', // 随机生成姓名
        'email': '@email', // 随机生成姓名
        'age|10-20': 12
        // 其他数据
      }
    }
  }
}
```

#### menu.js

```js
// 获取菜单信息
export function getMenu () {
  return {
    // isOpen: false,
    url: 'http://localhost:8080/menu',
    type: 'get',
    data: {
      'msg': 'success',
      'code': 0,
      'data': {
        'id': '@increment', 
        'name': 'menu', // 随机生成姓名
        'order|10-20': 12
        // 其他数据
      }
    }
  }
}
```

#### 修改引入

##### 组件内

````js
//之前是
import mock from '@/mock/mock.js';
//改为
import mock from '@/mock/index.js';
````

这是封装成功的效果

![](mockjs/image-20210225155138789.png)

## 踩坑

在vue中使用axios做网络请求的时候，会遇到this不指向vue，而为undefined。

解决 this的指向问题，要么箭头函数，要么直接let that=this

```js
//mockjsDemo2
updateItem(id){
        var that = this;
        this.$http.post('/updateItem',{
          params: {
            updateId:id
          }
        }).then(function(res){
          console.log("更新数据",res);
          that.list = res.data.data;
        }).catch((err) => {
          console.log(err)
        })
      } 
```

## mockjs 与 import export的大坑！！！！！！！！！！！！

> 注意 这里跟定义const let 这些其实没有很大关系 主要是理解三者联系
>
> 一般来说，如果是只是想测试写出比较简单的测试， 实际上直接写json获取 或者说直接写api接口都可以了，不用写复杂的mock文件

需求：写一个mockjs，模拟一个数据库，生成一个固定的数据array，并且统一操作删加减查

问题： data.js export getter函数给api文件，但是api文件直接使用export的getter函数时候，获取的值是空的

因为你在做梦啊~~~~  让mockjs 生成一个对于全部所有页面固定数据的表，因为要确保每个页面call的时候都是同一个数据arr。那么我总得存起来，但是由于是用import export的做法。那么我每次页面call的时候，还是要确保数据是新init的 。但是这时候init数据就变化了  所以总结一句**“你在想peach！！”**

## Reference

 [在vue-cli项目下简单使用mockjs模拟数据](https://segmentfault.com/a/1190000016730919) (demo1)

[掌握MockJS](https://www.bilibili.com/video/BV1Tt411T7Cw?p=1) (demo2)

[Vue + Element UI 实现权限管理系统](https://blog.csdn.net/xifengxiaoma/article/details/92839222?spm=1001.2014.3001.5501) (admin-ui)