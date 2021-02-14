# 全栈公开课2020学习笔记 03 - 用 NodeJS 和 Express 写服务端程序

<!--more-->

### Node.js 与 Express

前端浏览器还不支持 JavaScript 的最新特性，所以在浏览器运行的代码是 [babel](https://babeljs.io/) 转译过的，而在后端运行 JavaScript 的情况是使用 Node ，而 Node 支持大部分最新的 JavaScript 特性，所以后端代码不需要转译。

npm 来源于 Node 生态，是管理 JavaScript 包的工具。

一般情况在终端使用 `node index.js` 即可启动项目，但最好的做法是在项目的 `package.json` 中定义脚本：

```json
{
  // ...
  "script":
  {
    "start": "node index.js",
    // ...
  },
  // ...
}
```

之后只要在项目根目录使用 `npm start` 即可启用项目。

#### 简易服务器

该部分仅作了解即可，在实际项目中这样的方式太麻烦。主流的方式是使用例如 Express 这类的库。

新建一个项目，在根目录使用 `npm init` 来初始化项目，创建 `index.js` ：

```js
const http = require('http')

const app = http.createServer((request, response) =>
{
  response.writeHead(200, { 'Content-Type': 'text/plain' })
  response.end('Hello World')
})

const PORT = 3001
app.listen(PORT)
console.log(`服务器运行在 ${PORT} 端口`)
```

通过 `npm start` 即可启动服务器，通过浏览器即可访问 http://localhost:3001。

如服务器提供的数据是 json，需要将 `Content-Type` 改为 `application/json`。输出的数组需要进行 `JSON.stringify(array)` 处理。

Node.js 使用了 [CommonJS](https://en.wikipedia.org/wiki/CommonJS)，还不支持 ES6 模块，所以不能使用 `import http from 'http'`。

#### Web and express

在项目根目录安装 `npm install express`。因为使用了 `npm` 来安装，所以会自动在项目 `package.json` 添加 `express` 依赖：

```json
{
  // ...
  "dependencies":
  {
    "express": "^4.17.1"
  }
}
```

`^4.17.1` 限制了该项目支持的 `express` 的最低版本。

让原先 `index.js` 使用 `express` 库。

```js
const express = require('express')
const app = express()

let notes = [
  // 略
]

// 定义路由
// 处理对应用 '/' 的 HTTP GET 请求
app.get('/', (req, res) =>
{
  // response 响应请求，返回 <h1>Hello World!</h1>。express 会自动设置 Content-Type 为 text/html，状态码 200
  res.send('<h1>Hello World!</h1>')
})

// 处理对应用 '/api/notes' 的 HTTP GET 请求
app.get('/api/notes', (req, res) =>
{
  // response 响应请求，返回 json 数据。express 会自动设置 Content-Type 为 application/json，状态码 200
  res.json(notes)
})

// 服务器运行端口
const PORT = 3001
app.listen(PORT, () =>
{
  console.log(`服务器运行在 ${PORT} 端口`)
})
```

#### nodemon

通过使用 `nodemon`，可以使应用在开发过程中由于代码的修改而自动重启服务，从而避免每次修改完代码后都需要重新 `npm start` 项目。

通过 `npm install nodemon --save-dev` 在项目中添加开发依赖，会在 `package.json` 的开发依赖中定义。

```json
{
  //...
  "dependencies": {
    "express": "^4.17.1",
  },
  "devDependencies": {
    "nodemon": "^2.0.2"
  }
}
```

开发依赖是在开发时需要用到的而在生产环境中用不到的依赖。

使用方法还是在 `package.json` 的增加脚本，而后通过 `npm run dev` 来启动，这于 `start` 和 `test` 不同，需要在命令中加入 `run`。 

```json
{
  // ..
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  // ..
}
```

#### REST

> 在 RESTful thinking 中称为 resource。 每个 resource 都有一个相关联的 URL，这个 URL 是资源的唯一地址。
>
> 一个约定是结合 resource 类型名称和 resource 的唯一标识符来创建 resource 唯一的地址。

假设根目录为 `/api`，那么笔记的集合则是 `/api/notes` ，单个笔记则是 `/api/notes/10`。

对资源又可以进行不同的操作，以下表格可以粗略地定义为 REST 所指的 [统一接口 uniform interface](https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints)：

| URL | Verb | functionality |
|:----|:-----|:--------------|
|notes|  GET | 获取整个 resources 集合 |
|notes| POST | 根据 request 的数据创建新 resource |
|notes/10| GET | 获取指定的 resource |
|notes/10|DELETE| 删除指定的 resource |
|notes/10|PUT | 根据 request 的数据修改指定的整个 resource |
|notes/10|PATCH| 根据 request 的数据修改指定的 resource 的部分数据 |

*REST API 可以当作为是一种约定，而非一种标准*，世界上大多数 REST API 都不符合 Fielding 在其论文中概述的原始标准。所以在编写 API 时要有一致性，以便系统进行合作。

#### Fetching a single resource 获取一个单一资源

我们约定的单个笔记的唯一地址是 `notes/10`，其中 10 是笔记的唯一 id 号。通过冒号语法为路由定义参数：

```js
// 处理对应 api/notes/id 的路由，其中 id 可以为任意字符串
app.get('/api/notes/:id', (req, res) =>
{
  // 从 request 中获得 id 参数，并将 id 更改为 number
  const id = Number(req.params.id)
  const note = notes.find(note => note.id === id)

  if (note)
  {
    res.json(note)
  }
  else
  {
    // 笔记不存在则返回 404 状态码
    res.status(404).end()
  }
})
```

#### Deleting Resources 删除资源

```js
app.delete('/api/notes/:id', (req, res) =>
{
  const id = Number(req.params.id)
  notes = notes.filter(note => note.id !== id)

  // 返回 204 状态码
  res.status(204).end()
})
```

#### Receiving data 接收数据

向 `/api/notes` 发送 HTTP POST 请求，并以 JSON 格式在 request body 中发送新笔记的信息，需要使用到 express json-parser，它与命令 `app.use(express.json())` 一起使用。

```js
const express = require('express')
const app = express()

app.use(express.json())

const generateId = () =>
{
  const maxId = notes.length > 0
    ? Math.max(...notes.map(note => note.id)) // 使用展开语法将 notes 中所有的 id 都展开出来
    : 0
  return maxId + 1
}

// ...

app.post('/api/notes', (req, res) =>
{
  const body = req.body

  if (!body.content)
  {
    // 使用 return 返回 400 状态码以及错误提示并阻止代码继续执行
    return res.status(400).json(
    {
      error: '内容缺失'
    })
  }

  const note =
  {
    content: body.content,
    important: body.important || false,
    data: new Date(),
    id: generateId(),
  }

  notes = notes.concat(note)

  res.json(note)
})
```

像 DELETE、POST 等操作，在浏览器上不是很方便执行，使用 Postman 应用或者 VS Code REST client 插件来操作即可。

如果 POST 请求出现错误，需要检查 Postman 或者 REST client 插件是否将 POST 数据的 `Content-Type` 设置为 `application/json`。

#### About HTTP request types

HTTP 标准讨论了与请求类型相关的两个属性，安全「safety」 和 幂等性「idempotence」 。

##### 安全「safety」

HTTP GET 请求应该是满足安全性的：

> In particular, the convention has been established that the GET and HEAD methods SHOULD NOT have the significance of taking an action other than retrieval. These methods ought to be considered "safe".

安全性意味着执行请求不能在服务器中引起任何**副作用「side effect」**。 副作用是指数据库的状态**不能因请求而改变**，响应**只能**返回服务器上已经存在的数据。

无法保证 GET 请求是安全的，这实际上是 HTTP 标准的建议。通过遵守 API 中的 RESTful 原则，GET 请求实际上总是以一种安全的方式使用。

HTTP 标准还定义了 [HEAD](https://www.w3.org/protocols/rfc2616/rfc2616-sec9.html#sec9.4)， 实际上，HEAD 应该像 GET 一样工作，但是它只返回状态码和响应头。 当您发出 HEAD 请求时，不会返回响应主体。

##### [幂等「idempotence」](https://developer.mozilla.org/zh-CN/docs/Glossary/%E5%B9%82%E7%AD%89)

除了 POST 以外的所有 HTTP 请求都是幂等的，这意味着，如果一个请求有副作用，那么无论发送多少次请求，结果都应该是相同的。

> Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request. The methods GET, HEAD, PUT and DELETE share this property.

POST 是唯一的不安全也不幂等的 HTTP 请求类型。发送多个 POST 请求，可以产生多条内容。

#### Middleware 中间件

中间件是可用于处理请求和响应对象的函数。可以从请求对象中存储的请求中获取原始数据，将其解析为一个 JavaScript 对象，并将其作为一个新的属性、body 分配给请求对象。

可以使用多个中间件，会按照顺序一个一个地执行。

中间件是一个接收三个参数的函数：

```js
const requestLogger = (request, response, next) =>
{
  console.log('Method: ', request.method)
  console.log('Path: ', request.path)
  console.log('Body: ', request.body)
  next() // 调用作为参数传递的下一个函数。 函数将控制权交给下一个中间件。
}

// 使用方法
app.use(express.json()) // 获得 body 内容，保证 requestLogger 能够得到 request.body 内容
app.use(requestLogger)
```

中间件函数按照与 express 服务器对象的使用方法一起使用的顺序调用。 如果我们希望在调用路由事件处理程序之前执行路由，则必须在路由之前使用中间件函数。相反的，中间件函数放在路由之后，使中间件在没有路由处理 HTTP 请求的时候调用。

### 将应用部署到网上

#### Same origin policy and CORS 同源政策和 CORS

> Cross-origin resource sharing (CORS) is a mechanism that allows restricted resources (e.g. fonts) on a web page to be requested from another domain outside the domain from which the first resource was served. A web page may freely embed cross-origin images, stylesheets, scripts, iframes, and videos. Certain "cross-domain" requests, notably Ajax requests, are forbidden by default by the same-origin security policy. Cross-origin resource sharing.
>
> _—— 维基百科_

[同源策略](https://developer.mozilla.org/en-us/docs/web/security/same-origin_policy) 和 [CORS](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS) 是 Web 应用的通用原则。

在 Node 项目中可以使用 `cors` 中间件来允许来自其他的请求。

通过 `npm install cors` 来安装。

```js
const cors = require('cors')

app.use(cors())
```

#### Application to the Internet 将应用部署到网上

使用 [Heroku](https://www.heroku.com/) 来部署应用。

在项目根目录新建 `Procfile` 文件，在文件内填写 `web: npm start` 来告诉 Heroku 如何启动应用。

将应用的 `index.js` 中端口改为环境变量：
```js
const PORT = process.env.PORT || 3001
// ...
```

在应用内创建 git 仓库，并使用 git 方式或者 ssh 方式将应用部署至 Heroku 上，方法同其他 git 方式，VS code 集成了 git 功能，可以直接使用，需要注意的是，`git push` 时候，需要的是 Heroku 的 Access API 而不是用户密码。

#### Frontend production build 前端生产构建

目前为止，应用处于开发模式中运行，简言之就是「人类可读」的环境，在部署应用时，我们需要将应用创建一个[生产构建](https://zh-hans.reactjs.org/docs/optimizing-performance.html#use-the-production-build)或为生产而优化的版本，也就是「机器可读」的环境。

`create-react-app` 创建的应用可以使用 `npm build` 命令来构建。

> 通过在应用根目录运行这个命令，将会创建一个名为build 的目录(其中包含应用中唯一的 HTML 文件index. HTML) ，其中包含目录 static。 应用的 JavaScript 代码的 [Minified](https://en.wikipedia.org/wiki/minification_(programming)) 版本将生成到 static 目录。 即使应用代码位于多个文件中，所有的 JavaScript 都将被缩小到一个文件中。 实际上，来自所有应用依赖项的所有代码也将缩小到这个单一文件中。

#### Serving static files from the backend 从后端服务部署静态文件

将前端构建的 `build` 文件夹复制到后端应用的根目录，通过 express 的内置的 static 中间件来现实前端构建的内容。

```js
app.use(express.static('build'))
```

当 express 收到 HTTP GET 请求时，它会首先检查 build 目录是否包含与请求地址对应的文件，并返回相应的内容。

这样的方式采用了前后端在一起的方式，所以可以将前端的 `baseUrl` 改成相对路径 `/api/notes` 即可。

在确保生产环境在本地正常之后，将生产构建 `git push` 至 Heroku 完成部署。

#### Streamlining deploying of the frontend 流程化前端部署 

通过更改后端 `package.json` 的 scripts 来实现流程化构建，以下直接引用了教材内容，一般来说根据项目实际情况来填写这些 scripts

```json
{
  "scripts": {
     //...
    "build:ui": "rm -rf build && cd ../../osa2/materiaali/notes-new && npm run build --prod && cp -r build ../../../osa3/notes-backend/",
    "deploy": "git push heroku main",
    "deploy:full": "npm run build:ui && git add . && git commit -m uibuild && npm run deploy",    
    "logs:prod": "heroku logs --tail"
  }
}
```

#### Proxy 代理

在前端开发时，因为之前将 `baseUrl` 更改成后端的相对路径，所以在开发模式运行前端内容时，并不能获取到准确的为止，所以此时需要通过代理，将相对路径的请求转发至后端服务器。

在 `package.json` 增加 `proxy` 代理设置来转发。

```json
{
  // ...
  "scripts": {
    // ...
  },
  "proxy": "http://localhost:3001"
}
```

> 方法的一个劣势，是前端部署的复杂程度。 部署新版本需要生成新的前端生产构建并将其复制到后端存储库。 这使得创建一个自动化的[部署管道](https://martinfowler.com/bliki/DeploymentPipeline.html)变得更加困难。 部署管道是指通过不同的测试和质量检查将代码从开发人员的计算机转移到生产环境的自动化控制的方法。

### 将数据存入 MongoDB

#### Debugging Node applications 调试 Node 应用

将数据 `console.log` 到控制台是一种可靠的方法。

使用 VS Code 的代码调试器，在软件的 Run 菜单下选择 Add Configuration... 来配置 debug 的设置，选择代码使用的环境 Node.js 会自动生成配置文件，然后选择 Start Debugging 开始调试。

使用 Google Dev Tools 调试应用，在控制台使用 `node --inspect index.js` 进行调试，也可以在 Console 控制台点击 Node logo 开启。*系统需要全局安装 Node，如果只是在项目中安装 Node 是使用命令，也不显示 Node logo 的*

**质疑一切，出现 bug 时，逐一排除所有的可能性，将 bug 修复后再继续编写代码。**

#### MongoDB

MongoDB 不同于 MySQL 等其他关系数据库，它时[文档数据库](https://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E6%96%87%E6%AA%94%E7%9A%84%E6%95%B8%E6%93%9A%E5%BA%AB)，它通常被归类为 [NoSQL](https://zh.wikipedia.org/wiki/NoSQL)

使用 [MongoDB Atlas](https://www.mongodb.com/cloud/Atlas) 为应用提供数据存储服务，也可以在本地创建 MongoDB。

使用 Mongoose 库来代替 MongoDB 官方的驱动程序操作数据库会非常方便。Mongoose 可以被描述为 `ODM(Object Document Mapper)`，它可以非常简单的将 JavaScript 对象保存为 Mongo 文档。

在项目中使用 `npm install mongoose` 安装 Mongoose。

新增 `mongo.js` 用于与数据库的连接：

```js
const mongoose = require('mongoose')

if ( process.argv.length < 3 )
{
  console.log('请提供连接数据库的密码：node mongo.js <passowrd>')
  process.exit(1)
}

const password = process.argv[2]

const url = `mongodb+srv://fullstack:${password}@cluster0-ostce.mongodb.net/test?retryWrites=true`

mongoose.connect(
  url,
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
    useCreateIndex: true
  }
)
```

可以通过修改 url 中的 test 来更改数据库名称，如果数据库中没有这个数据库，它会在连接成功后自动添加。

#### Schema

以下定义了笔记的 [Schema](http://mongoosejs.com/docs/guide.html) 模式和匹配的 [Model](http://mongoosejs.com/docs/models.html) 模型。

```js
// mongo.js
// ...
const noteSchema = new mongoose.Schema(
  {
    content: String,
    date: Date,
    important: Boolean,
  }
)

const Note = mongoose.model('Note', noteSchema)
```

Schema 告诉 Mongoose 如何将 note 对象存储在数据库中。

> 在 Note 模型定义中，第一个 "Note" 参数是模型的单数名。集合的名称将是小写的复数 notes，因为 Mongoose 约定是当模式以单数（例如 Note）引用集合时自动将其命名为复数（例如 notes）。
>
> 像 Mongo 这样的文档数据库是 schemaaless，这意味着数据库本身并不关心存储在数据库中的数据的结构。 可以在同一集合中存储具有完全不同字段的文档。
>
> Mongoose 背后的思想是，存储在数据库中的数据在 application 级别上被赋予一个 schema，该模式定义了存储在任何给定集合中的文档的形状。

#### Creating and saving objects 创建和保存对象

在 Node Model 模型的帮助下，创建新的 Note 非常简单：

```js
// mongo.js
// ...
const note = new Note(
  {
    content: '新的 note',
    date: new Date(),
    important: false,
  }
)
```

> 模型是所谓的构造函数 constructor function，它根据提供的参数创建新的 JavaScript 对象。 由于对象是使用模型的构造函数创建的，因此它们具有模型的所有属性，其中包括将对象保存到数据库的方法。

通过 `save()` 方法来保存新的 `note`：
```js
// mongo.js
// ...
note.save().then(result =>
{
  console.log('保存成功')
  // 关闭数据库连接，否则程序无法完成它的执行。
  mongoose.connection.close()
})
```

#### Fetching objects from the database 从服务器获取对象

使用 `find()` 方法来获取：

```js
// mongo.js
// ...
Note.find({}).then(result =>
{
  result.forEach(note =>
  {
    console.log(note)
  })
  mongoose.connection.close()
})
```
`find({})` 方法中 `{}` 是空对象，表示没有检索条件，数据库会返回所有结果。

`find({ important: true })` 使用这样的条件将会返回所有重要的笔记。

#### Backend connected to a database 后端连接到数据库

使用与上方相同的方法，将数据库连接的代码添加到 index.js 中去，**注意，若数据库密码不从 `process` 中得到，而直接保存在 `url` 中，切勿将带有密码的文件提交至 Github**，并将路由调整如下：

```js
route.get('/api/notes/',(req, res) =>
{
  Note.find({}).then(notes =>
  {
    res.json(notes)
  })
})
```

如果不想要返回 `_id` 和 `__v` 字段，通过 Schema 的 toJSON 方法来修改，如下：

**注意一点，mongo 的 `_id` 字段看起来像个 `String 类型`，但它其实是个对象**

```js
noteSchema.set('toJSON',
{
  transform: (document, returnedObject) =>
  {
    returnedObject.id = returnedObject._id.toString() // _id 字段是对象，需要 toString 方法来转换成 String
    delete returnedObject._id;
    delete returnedObject.__v;
  }
})
```

#### Database configuration into its own module 数据库逻辑配置到单独的模块

便于维护及管理，将数据库逻辑配置到单独的模块是较好的选择。

在项目根目录下创建 `models` 文件夹，并在该文件夹下新建 `note.js` 文件：

```js
const mongoose = require('mongoose')

const url = process.env.MONGODB_URI

console.log('正在连接', url)

mongoose.connect
(
  url,
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
    useCreateIndex: true
  }
)
.then(result =>
{
  console.log('成功连接 MongoDB')
})
.catch((error)=>
{
  console.log('连接 MongoDB 失败：', error.message)
})

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  important: Boolean,
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

此处定义 `Note` modules 与定义 ES6 模块方式略有不同。

> 模块的公共接口是通过将值设置为 module.exports 变量来定义的。 我们将该值设置为 Note 模型。模块内部定义的其他东西，比如变量 mongoose 和 url 对于模块的用户来说是不可访问的或者不可见的。

在 `index.js` 中导入 `note` 模块即可：

```js
const Note = require('./models/note')
```

将数据库的 `url` 直接编入代码中不是一个安全的选择，将它设置为环境变量 `MONGODB_URI` 传递给应用会更合适。

定义环境变量的方式有很多种：

- 在启动时定义。 *在 `package.json` 中添加 `scripts`*

```json
"mongodb": "MONGODB_URI=address_here npm run dev",
```

- 更复杂的方法是使用 `dotenv` 库

```bash
npm install dotenv
```

在项目根目录新建 `.env` 文件，环境变量在改文件内定义。

```
MONGODB_URI='mongodb+srv://fullstack:passwordhere@icluster.fmdqa.mongodb.net/note-app?retryWrites=true&w=majority'
PORT=3001
```

在 `.gitignore` 中添加 `.env` 条目来避免将环境变量上传至 github，并在 `index.js` 第一行添加 `require('dotenv'.config)` 命令来使用 `.env` 中的环境变量。引用环境变量的方法与普通环境变量一样 `process.env.MONGODB_URI`。

#### Using database in route handlers 在路由处理程序中使用数据库

添加一条新笔记：

```js
app.post('/api/notes', (req, res) =>
{
  const body = req.body

  if (body.content === undefined)
  {
    return res.status(400).json({ error: '提交的内容缺失' })
  }

  // 使用 Note 构造函数创建 Note 对象
  const note = new Note({
    content: body.content,
    important: body.important || false,
    date: new Date(),
  })

  note.save().then(saveNote =>
  {
    // 操作成功时才发送 response
    res.json(saveNote)
  })
})
```

使用 mongoose 的 `findById` 方法获取单独一条笔记：

```js
app.get('/api/notes/:id', (req, res) =>
{
  note.findById(req.params.id).then(note =>
  {
    res.json(note)
  })
})
```

#### Verifying frontend and backend integration 验证前端和后端的集成

使用 POSTMAN 或者其他工具来测试后端。

#### Error handling 错误处理

通过 `.catch` 方法来捕获错误。

```js
app.get('/api/notes/:id', (req, res) =>
{
  Note.findById(req.params.id)
    .then( note =>
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
    .catch(error =>
    {
      console.log(error)
      res.status(500).end()
    })
})
```

当请求指定 id 的笔记不存在时，服务器会返回 404 错误，当给出的 id 是一个奇怪的参数时， `findById` 方法会抛出一个错误，导致 `Promise` 返回 `rejected`，因此会触发 `catch` 代码中的函数，也就是 *500 internal server error*。

实际上当 id 格式不正确会触发 `catch` 的代码，那么将这个错误定义为 `400 bad request` 比较符合对错误的描述。

```js
.catch(error =>
{
  console.log(error)
  res.status(400).send({ error: '请求错误，id 格式不正确' })
})
```

> 调用错误处理程序的原因可能与您预期的完全不同。 如果您将错误记录到控制台，您可以避免冗长和令人沮丧的调试会话。

#### Moving error handling into middleware 将错误处理移入中间件

上述代码是在单个路由内编写了错误处理程序，这是一种方法。在单个位置实现所有的错误处理，是一个比较好的方法。

> 如果我们以后想要将与错误相关的数据报告给外部的错误跟踪系统，比如 [Sentry](https://sentry.io/welcome/)，那么这么做就特别有用。

在路由中，使用 `next` 函数向下传递错误，`next` 函数作为第三个参数传递给处理程序。

```js
app.get('/api/notes/:id', (request, response, next) =>
{
  Note.findById(request.params.id)
  .then(note =>
  {
    if (note)
    {
      response.json(note)
    }
    else
    {
      response.status(404).end()
    }
  })
  .catch(error => next(error))
})
```

> 将向前传递的错误作为参数给 `next` 函数。 如果在没有参数的情况下调用 `next`，那么执行将简单地转移到下一个路由或中间件上。 如果使用参数调用  `next` 函数，那么执行将继续到 error 处理程序中间件。

Express 的 `errorHandler` 是一种中间件，它定义了一个接受 4 个参数的函数：

```js
const errorHandler = (error, request, response, next) =>
{
  console.log(error.message)

  if (error.name === 'CastError' && error.kind === 'ObjectId')
  {
    return response.status(400).send({ error: '请求错误，id 格式不正确' })
  }

  next(error)
}

app.use(errorHandler)
```

`errorHandler` 将会检查错误是否是 *CastError* 错误以及种类是否是 *ObjectId*，如果是将会返回 400 错误以及错误消息。如果不是，中间件将会转发错误至缺省的 Express 错误处理程序。

#### The order of middleware loading 中间件加载的顺序

中间件的执行顺序与通过 `app.use` 函数加载到 Express 顺序相同*「自上而下」*。

```js
app.use(express.static('build'))
app.use(express.json)
app.use(logger)

app.post('api/notes', (req, res) =>
{
  const body = req.body
  // ...
})

// ...

const unknownEndpoint = (req, res) =>
{
  res.status(404).send({ error: '访问的页面不存在' })
}

// 访问不存在页面的处理程序
app.use(unknownEndpoint)

const errorHandler = (err, req, res, next)
{
  // ...
}

app.use(errorHandler)
```

如果 `app.use(express.json())` 放在了 `app.post` 路由下面，那么 `express.json()` 将会在路由之后执行，这意味着 `app.post` 中的 `body` 无法正确地获得数据，因为 `req.body` 是 undefined。

#### Other operations 其他操作

`findByIdAndRemove` 方法及 `findByIdAndUpdate` 方法。

```js
app.put('/api/notes/:id', (request, response, next) => {
  const body = request.body

  const note = {
    content: body.content,
    important: body.important,
  }

  Note.findByIdAndUpdate(request.params.id, note, { new: true })
    .then(updatedNote => {
      response.json(updatedNote)
    })
    .catch(error => next(error))
})
```

在 `findByIdAndUpdate` 方法中使用了可选参数 `{ new: true }`，这样 `updateedNote` 接收到的是修改后的文档。默认情况下 `updatedNote` 接收到的是原始未修改的文档。这点要注意。

### ESLint 与代码检查

先前在路由中检查笔记的有效性，如果没有数据，将返回 400 错误信息以及错误信息「提交的内容缺失」：

```js
app.post('/api/notes', (req, res) =>
{
  const body = req.body

  if (body.content === undefined)
  {
    return res.status(400).json({ error: '提交的内容缺失' })
  }
  // ...
})
```

在数据存储到数据库之前验证数据的一个更好的方法是使用 mongoose 提供的 [validation](https://mongoosejs.com/docs/validation.html) 功能。

```js
const noteSchema = new mongoose.Schema(
  {
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
  }
)
```

> minlength 和 required 验证器是内置的 ，由 Mongoose 提供。如果没有一个内置的验证器满足我们的需求的话，Mongoose 允许我们创建新的验证器自定义验证器。

在 `errorHandler` 中间件增加 `validation` 的判断：

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
```

#### Promise chaining 承诺链

教材所讲的是承诺链的概念，所用的案例并不是很典型，引用作者的话：

> 在这个例子中，承诺链没有提供多少好处。 但要是有许多必须按顺序进行的异步操作，情况就会发生变化……在本课程的下一章节中，我们将学习 JavaScript 中的async/await 语法，这将使编写后续的异步操作变得容易得多。

```js
note.save()
.then()
.then()
.then()
.catch()
```

`.then` 返回了一个 `Promise`，通过多个 `.then` 方法组成了 `Promise Chaining`。

如同 then 的字面意思，代码是依次执行的。

#### Deploying the database backend to production 将数据库后端部署到生产环境

略

#### Lint

Lint 是什么？

>Generically, lint or a linter is any tool that detects and flags errors in programming languages, including stylistic errors. The term lint-like behavior is sometimes applied to the process of flagging suspicious language usage. Lint-like tools generally perform static analysis of source code.
>
> 通常，lint 或 linter 是检测和标记编程语言中的错误，包括文本错误的一种工具。 lint-like 这个术语有时用于标记可疑的语言使用情况。 类似 lint 的工具通常对源代码执行静态分析。

> 在像 Java 这样的编译静态类型语言中，像 NetBeans 这样的 IDE 可以指出代码中的错误，甚至那些不仅仅是编译错误的错误。 执行静态分析的额外工具，如检查样式 ，可以用来扩展 IDE 的功能，也指出与样式有关的问题，如缩进。

>在 JavaScript 的世界里，目前主要的静态分析工具又名 「linting」是 [ESlint](https://eslint.org/)。

安装 ESlint 作为开发依赖：

```bash
npm install eslint --save-dev
```

初始化默认设置：

```bash
node_modules/.bin/eslint --init
```
{{< figure src="https://fullstackopen.com/static/ba1423527692484103dcb2b7374eeb01/5a190/52be.png" >}}

该配置将保存在 `.eslintrc.js` 文件中。

可根据习惯修改规则，比如使用 2 个 space。

检查验证方式：

```bash
node_modules/.bin/eslint index.js
```

为 linting 创建一个单独的脚本

```json
{
  // ...
  "scripts":
  {
    "start": "node index.js",
    "dev": "nodemon index.js",
    // ...
    "lint": "eslint ."
  },
  // ...
}
```

在根目录新建 `.eslintignore` 并增加想要忽略检查的文件。

安装 `eslint-plugin` 来替代使用命令检查代码中的错误，ESlint 插件会将不符合规则的代码使用红色波浪线标记出来。

ESlint 有大量的[规则](https://eslint.org/docs/rules/)， 通过在 `.eslintrc.js` 文件中编辑增加规则即可。

> 许多公司定义了通过 ESlint 配置文件在整个组织中执行的编码标准。 建议不要一遍又一遍地使用重造轮子，从别人的项目中采用现成的配置到自己的项目中可能是一个好主意。 最近，很多项目都采用了 Airbnb 的 Javascript 风格指南，使用了 Airbnb 的 [ESlint](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb) 。
