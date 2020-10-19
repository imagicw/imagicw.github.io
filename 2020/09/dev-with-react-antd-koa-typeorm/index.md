# React + antd + koa + typeORM 开发的学习笔记（持续更新）


2周前，公司因为业务需要，要开发一个集装箱散货拼单系统。

本次主要想不使用 `php` 语言，同时想要尝试目前主流的开发模式，所以最终选用了 `react` `antd Pro` 框架进行快速开发。

<!--more-->

理想是美好的，现实很骨感。这些框架对于我来说都是新的，第一周几乎都是在看框架手册、教程以及跟着入门教学一字一字码出来。

中间绕了不少弯路，比如一开始直接上手 `antd Pro`，结果其中的代码逻辑一点都搞不明白。后来就降级从 `antd` 开始，看着官方写的教程一步一步进行下去，中间一半发现卡住了，后来发现教程版本是 `v3`，而我使用的是 `v4`，无奈，只好再各种翻阅资料。

在一开始做前端的时候，我就一直很疑惑如何开发后端。翻阅了2天资料，以及按照按照网络上的教程，最终选用了 `koa` `typeORM` `mysql` 作为后端。


以下主要记录在实战过程中犯的错以及解决的疑难问题。

### 前端

####  token 的处理

遇到的问题

- 用户登出后，换了账号登陆，所有请求依然是上一个用户的 `token`。即使在控制台看到 `token` 已经改变。刷新页面可以解决。
- 未对用户在线状态检验。

第一个问题主要由于 `request` 设置问题引起的，原先代码看起来是判断token是否存在，实际上并不会每次都会检查。解决方案是使用了拦截器，每次请求的时候都会检查 `localStorage`。

原先的代码

```js
const request = extend({
  errorHandler,
  // 默认错误处理
  credentials: 'include', // 默认请求是否带上cookie
  headers: {
  	Authorization: localStorage.getItem('token')? `Bearer ${localStorage.getItem('token')}` : null,
  }
});
```

改成拦截器的方式

```js
const request = extend({
  errorHandler,
  // 默认错误处理
  credentials: 'include', // 默认请求是否带上cookie
});

// 此处为拦截器，每次发送请求之前判断能否取到token
request.interceptors.request.use(async (url, options) => {  
  if (localStorage.getItem('token')) {
    const headers = {
      'Content-Type': 'application/json',
      Accept: 'application/json',
      Authorization: `Bearer ${localStorage.getItem('token')}`,
    };
    return {
      url,
      options: { ...options, headers },
    };
  }
  return {
    url,
    options: { ...options },
  };
});
```

### 后端

#### 数据拆分

问题现象：遍历过程中在对数据库进行新增操作时，只保存最后一条信息或是保存多条与最后一条相同的信息。前者用了 `for of`，后者用了 `forEach`。

问题代码：

```js
const bulk = await bulkRepository.findOne(bulkId);

const newBulk = new Bulkdata();

for ( const [index, item] of seperateQtys.entries())
{
  newBulk.style = bulk.style;
  newBulk.po = bulk.po;
  newBulk.shippingNo = bulk.shippingNo;
  newBulk.handover = bulk.handover;
  newBulk.singleGross = bulk.singleGross;
  newBulk.singleVolume = bulk.singleVolume;
  newBulk.cartonQty = bulk.cartonQty;
  newBulk.cartonSize = bulk.cartonSize;
  newBulk.factory = bulk.factory;
  newBulk.merchandiser = ctx.state.user.id;
  newBulk.country = bulk.country;

  newBulk.cartonQty = item;
  newBulk.gross = item * bulk.singleGross;
  newBulk.volume = Number((item * bulk.singleVolume).toFixed(5));

  if(index === 0)
  {
      await bulkRepository.update(bulkId, newBulk);
  }
      else
  {
      const result = await bulkRepository.save(newBulk);
      await bulkRepository
          .createQueryBuilder()
          .relation(Bulkdata, "parentBulk")
          .of(result)
          .set(bulkId)
  }
};
```

问题原因：因 `newBulk` 在循环之外创建，整个循环都是操作的同一个对象。

解决方案：将 `new Bulkdata()` 操作放进循环内。

```js
const bulk = await bulkRepository.findOne(bulkId);

for ( const [index, item] of seperateQtys.entries())
{
  const newBulk = new Bulkdata();

  newBulk.style = bulk.style;
  // 余下内容与之前相同
};
```
