# 全栈公开课2020学习笔记 04 - 测试 Express 服务端程序, 以及用户管理

<!--more-->

### 从后端结构到测试入门

#### 项目结构

遵循 Node.js 最佳实践，目录结构按照如下：

```
├── index.js
├── app.js
├── build
│   └── ...
├── controllers
│   └── notes.js
├── models
│   └── note.js
├── package-lock.json
├── package.json
├── utils
│   ├── config.js
│   ├── logger.js
│   └── middleware.js  
```

将原先代码中所有的 `console.log` 和 `console.error` 集成在 `utils/logger.js` 模块中。

```js
const info = (...params) =>
{
  console.log(...params)
}

const error = (...params) =>
{
  console.error(...params)
}

module.exports =
{
  info,
  error
}
```

logger 提供打印 info 信息以及 error 信息的功能。

> 将日志记录功能提取到一个单独的模块是个不错的实践。 如果我们后来想将日志写入一个文件，或者将它们发送到一个外部日志服务中，比如 [graylog](https://www.graylog.org/) 或者 [papertrail](https://papertrailapp.com/)，我们只需要在一个地方进行修改就可以了。

我们先简化 `index.js`：

```js
const app = require('./app')                // 引用 app
const http = require('http')
const config = require('./utils/config')    // 引用环境变量
const logger = require('./utils/logger')    // 引用日志模块

const server = http.createServer(app)

server.listen(config.PORT, () =>
{
  logger.info(`服务器运行在 ${config.PORT}`)
})
```

将环境变量设置存放到 `utils/config.js` 中：

```js
require('dotenv').config()

const PORT = process.env.PORT
const MONGODB_URI = process.env.MONGODB_URI

module.exports =
{
  PORT,
  MONGODB_URI
}
```

通过 `const config = require('./utils/config')` 来访问环境变量。

路由处理程序通常称为 controller，所以将路由处理程序存放至 `controllers` 目录下。

新建 `notes.js`，并将原先 `index.js` 中的路由处理程序复制过来并稍作修改:

```js
const notesRouter = require('express').Router() // 创建新的 router 对象
const Note = require('../models/note')

notesRouter.get('/', (req, res) =>
{
  Note.find({})
    .then(notes =>
    {
      res.json(notes)
    })
})

notesRouter.get('/:id', (req, res, next) =>
{
  Note.findById(req.params.id)
    .then(note =>
    {
      if (note)
      {
        res.json(note)
      }
      else
      {
        res.status(404).end()
      }
    })
    .catch(error => next(error))
})

notesRouter.post('/', (req, res, next) =>
{
  const body = req.body

  if (!body.content)
  {
    return res.status(400).json(
      {
        error: '内容缺失'
      })
  }

  const note = new Note
  ({
    content: body.content,
    important: body.important || false,
    date: new Date()
  })

  note.save()
    .then(savedNote =>
    {
      res.json(savedNote)
    })
    .catch(error => next(error))
})

notesRouter.put('/:id', (req, res, next) =>
{
  const body = req.body

  const note =
  {
    content: body.content,
    important: body.important,
  }

  Note.findByIdAndUpdate(req.params.id, note, { new: true })
    .then(updatedNote =>
    {
      res.json(updatedNote)
    })
    .catch(error => next(error))
})

module.exports = notesRouter
```

需要注意的是，路由处理程序中的地址已经被缩短：

```js
// 原先 index.js 的路由处理程序
app.delete('/api/notes/:id', (request, response) =>
  // ...

// 现在 note.js 的路由处理程序
notesRouter.delete('/:id', (request, response) =>
  // ...
```

路由对象是什么？

> A router object is an isolated instance of middleware and routes. You can think of it as a “mini-application,” capable only of performing middleware and routing functions. Every Express application has a built-in app router.
>
> 路由对象是中间件和路由的单例。 您可以把它看作是一个「迷你应用」 ，只能执行中间件和路由功能。 每个 Express 应用都有一个内置的应用路由。

由此可见，路由对象是一个中间件，那么可以在 `app.js` 中使用 `use` 方法来使用这些路由处理程序。

```js
const notesRouter = require('./controllers/notes')

app.use('/api/notes', notesRouter)
```

当请求当 URL 是以 '/api/notes' 开头的，则会使用 `notesRouter` 路由处理程序定义的路由。

在 utils 目录下新增 `middleware.js`

```js
const errorHandler = (err, req, res, next) =>
{
  console.log(err)

  if (err.name === 'CastError' && err.kind === 'ObjectId')
  {
    return res.status(400).send({ error: 'error id' })
  }
  else if (err.name === 'ValidationError')
  {
    return res.status(400).json({ error: err.message })
  }

  next(err)
}

const requestLogger = (request, response, next) =>
{
  console.log('Method: ', request.method)
  console.log('Path: ', request.path)
  console.log('Body: ', request.body)
  next() // 调用作为参数传递的下一个函数。 函数将控制权交给下一个中间件。
}

const unknownEndpoint = (req, res) =>
{
  res.status(404).send({ error: '访问的页面不存在' })
}

module.exports =
{
  errorHandler,
  requestLogger,
  unknownEndpoint
}
```

在项目根目录新建 `app.js`

```js
const express = require('express')
const mongoose = require('mongoose')
const cors = require('cors')

const config = require('./utils/config')
const middleware = require('./utils/middleware')
const logger = require('./utils/logger')
const notesRouter = require('./controllers/notes')

const app = express()

logger.info('正在连接 ', config.MONGODB_URI)

mongoose.connect(
  config.MONGODB_URI,
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
    useCreateIndex: true
  })
  .then(() =>
  {
    logger.info('成功连接 MongoDB')
  })
  .catch((error)=>
  {
    logger.error('连接 MongoDB 失败：', error.message)
  })

app.use(cors())
app.use(express.static('build'))
app.use(express.json())

app.use(middleware.requestLogger)

app.use('/api/notes', notesRouter)

app.use(middleware.unknownEndpoint)
app.use(middleware.errorHandler)

module.exports = app
```

因为数据库的连接已经交给 `app.js` 去处理了，所以 `models` 下的 `note.js` 文件只需要定义 notes 的 Mongoose Schema 即可：

```js
const mongoose = require('mongoose')

const noteSchema = new mongoose.Schema({
  content:
  {
    type: String,   // 字符串
    minLength: 5,   // 最小长度 5 字符
    required: true  // 必填
  },
  date:
  {
    type: Date,     // 日期
    required: true  // 必填
  },
  important: Boolean  // 布尔值
})

noteSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString()
    delete returnedObject._id
    delete returnedObject.__v
  }
})

module.exports = mongoose.model('Note', noteSchema)
```

这样目录结构就比较清晰：

```
├── app.js        // 应用程序
├── build         // 前端 build
│   └── ...
├── controllers   // controller 存放处
│   └── notes.js
├── models        // model 存放数
│   └── note.js
├── package-lock.json
├── package.json
├── utils         // util 存放处
│   ├── config.js
│   ├── logger.js
│   └── middleware.js  
```

#### Testing Node application 测试 Node 应用

测试需要：

- 测试的 utils
- 测试库 Jest
- 测试 tests

编写测试 util，存储在 `/utils/for_testing.js`：

```js
// 回文测试
const palindrome = (string) =>
{
  return string
    .split('')
    .reverse()
    .join('')
}

// 平均数
const average = (array) =>
{
  const reducer = (total, current) => total + current

  return array.reduce(reducer, 0) / array.length
}

module.exports =
{
  palindrome,
  average,
}
```

以上测试需要使用到 `Jest` 测试库来测试，它可以很好的测试后端，对 React 应用测试也表现出色。

安装开发依赖：

```bash
npm install Jest --save-dev
```

并且需要在 `package.json` 中定义 `scripts`：

```json
{
  // ...
  "scripts":
  {
    // ...
    "test": "jest --verbose"
  }
}
```

`Jest` 需要指定执行环境，一般有两种方式：

- 在 `package.json` 添加。

```json
{
  // ...
  "jest" :
  {
    "testEnvironment": "node"
  }
}
```

- 在项目根目录添加 `jest.config.js` 中设置。

```js
module.exports =
{
  testEnvironment: 'node',
}
```

编写需要测试的内容存储在根目录下的 `tests` 文件夹中，通常约定测试文件以 `.test.js` 结尾：

`palindrome.test.js` ：

```js
const palindrome = require('../utils/for_testing.js').palindrome

test('a 的回文', () =>
{
  const result = palindrome('a')  // 测试的结果
  expect(result).toBe('a')        // 期待返回的结果
})

test('react 的回文', () =>
{
  const result = palindrome('react')
  expect(result).toBe('tcaer')
})

test('releveler 的回文', () =>
{
  const result = palindrome('releveler')
  expect(result).toBe('releveler')
})
```

如果遇到 ESlint 提示 `test` 和 `expect` 命令的问题，只需要在 `.eslintrc.js` 的 `env` 属性中添加 `"jest": true` 即可。

通过执行 `npm run test` 来执行测试：

{{< figure src="https://fullstackopen.com/static/03a306005b711bf48ab85efb74524f87/5a190/1.png" >}}

average 测试同理，可以通过添加 `describe` 来描述测试块的名称：

```js
describe('average', ()=>
{
  // tests
})
```

{{< figure src="https://fullstackopen.com/static/004954133431038eae2beda821cec5d7/5a190/4.png" >}}


### 测试后端应用

后端测试数据库可以使用以下几种方式来测试：

- 可以使用 `mongo-mock` 模拟数据库来进行后端测试。
- 使用本地构建的 `mongo` 数据库进行测试。
- 使用 Mongo Altas 新建 test 数据库进行测试。

#### Test environment 测试环境

Node 中约定 `NODE_ENV` 环境变量定义应用的执行模式，通常的做法是为开发和测试定义不同的模式。

修改 `package.json`

```json
{
  // ...
  "script":
  {
    "start": "NODE_ENV=production node index.js",
    "dev": "NODE_ENV=development nodemon index.js",
    // ...
    "test": "NODE_ENV=test jest --verbose --runInBand"
  }
  // ...
}
```

通过 `NODE_ENV` 定义了应用运行模式，`--runInBand` 是防止 `jest` 并行运行测试而添加的。

Windows 平台上会需要安装 `cross-env` 作为开发依赖包来解决脚本不能正常运行的问题：

```json
{
  // ...
  "script":
  {
    "start": "cross-env NODE_ENV=production node index.js",
    // 下同
  }
}
```

> 可以在 Mongo DB Atlas 中创建单独的测试数据库。但在多人开发同一个应用的情况下，这不是一个最佳解决方案。特别是测试执行时，通常要求并发运行的测试，因此不能使用单个数据库实例。

截止到目前 Note 应用较为简单，此处依旧使用 Mongo Altas 中新建 `note-app-test` 数据库进行测试。

修改环境变量 `.env`：

```js
MONGODB_URI='mongodb+srv://fullstack:password@icluster.fmdqa.mongodb.net/note-app?retryWrites=true&w=majority'
PORT=3001

TEST_MONGODB_URI='mongodb+srv://fullstack:password@icluster.fmdqa.mongodb.net/note-app-test?retryWrites=true&w=majority'
```

在应用设置 `config.js` 中进行调整以在不同的应用模式下使用正确的数据库：

```js
require('dotenv').config()

let PORT = process.env.PORT
let MONGODB_URI = process.env.MONGODB_URI

if (process.env.NODE_ENV === 'test')
{
  MONGODB_URI = process.env.TEST_MONGODB_URI
}

module.exports =
{
  PORT,
  MONGODB_URI
}
```

#### supertest

使用 [`supertest`](https://github.com/visionmedia/supertest) 来帮助编写 API 测试。

安装 `supertest` ：

```bash
npm install supertest --save-dev
```

在 `tests` 目录下新建 `note_api.test.js` 并编写测试代码：

```js
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app')

const api = supertest(app)

test('返回的 Notes 是 JSON', async ()=>
{
  await api
    .get('/api/notes')
    .expect(200)                // 验证状态码是否是 200
    .expect('Content-Type', /application\/json/)    // 验证 Content-Type 是否是 /application/json 格式
    // 以上使用的是 supertest 的 expect 方法
})

test('数据库中有 2 条数据', async ()=>
{
  const response = await api.get('/api/notes')
  expect(response.body).toHaveLength(2)         // jest 的 expect 方法
})

test('第一条数据是 "我喜欢 react"', async () =>
{
  const response = await api.get('/api/notes')
  expect(response.body[0].content).toBe('我喜欢 react')  // jest 的 expect 方法
})

afterAll(()=>
{
  mongoose.connection.close()   //关闭数据库
})
```

> 测试从 app.js 模块导入 Express 应用，并用 supertest 函数将其包装成一个所谓的 [superagent](https://github.com/visionmedia/superagent) 对象。 这个对象被分配给 api 变量，测试可以使用它向后端发出 HTTP 请求。

代码中 `async` 和 `await` 是异步操作相关的代码，后面会具体讲到 [async/await](https://facebook.github.io/jest/docs/en/asynchronous.html) 语法，使用 async/await 语法的好处开始变得明显。通常情况下，我们必须使用回调函数来访问由 promises 返回的数据，但是使用 async/await 语法会更加方便。

在测试全部完成之后，通过 [`afterAll`](https://facebook.github.io/jest/docs/en/api.html#afterallfn-timeout) 方法来关闭数据库。

如果没有在 `jest.config.js` 设置测试的环境为 `node`，控制台会发出相应的警告，只需按照之前的方式设置可以解决问题。


supertest 负责在内部使用端口启动被测试的应用。以下是文档说明：

> if the server is not already listening for connections then it is bound to an ephemeral port for you so there is no need to keep track of ports.
>
> 如果服务器还没有监听连接，那么它就会绑定到一个临时端口，因此没有必要跟踪端口。

在测试时会发现 `logger` 中间件会阻碍测试执行输出，将 `logger` 中间件的 `console.log` 方法仅在非 `test` 环境下执行：

```js
const info = (...params) =>
{
  if (process.env.NODE_ENV !== 'test')
  {
    console.log(...params)
  }
}

const error = (...params) =>
{
  console.error(...params)
}

module.exports =
{
  info,
  error
}
```

#### Initializing the database before tests 在测试前初始化数据库
