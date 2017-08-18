# 快速入门

*在阅读本文档之前，首先，你需要已经安装了 [MongoDB](http://www.mongodb.org/downloads) 以及 [Node.js](http://nodejs.org/).*

在你的工作目录下，创建一个名为 `mongoose-started` 的项目目录，并将该目录初始化为一个项目：

```bash
mkdir mongoose-codex-getting-started
cd mongoose-codex-getting-started
npm init
npm install mongoose --save
touch index.js
```

首先，我们在 `index.js` 文件中添加以下代码，创建一个至MongoDB的数据库连接：

```javascript
// 加载 Mongoose 模块
import Mongoose from 'mongoose'

// 创建数据库连接
Mongoose.connect('mongodb://localhsot/test')
const connection = Mongoose.connection
```

现在，我们已经有了一个等待连接至 `localhost` 服务器中 `test` 数据库的连接，现在我们需要告诉该连接，当它连接至数据库成功或者失败时，该做些什么：

```javascript
// 定义连接错误时的回调
connection.on('error', console.error.bind(console, 'connection error: '))

// 定义连接成功时的回调
connection.once('open', () => {
  // 我们已经连接成功了
})
```

当我们成功连接至数据库时，我们的回调都会被自动执行，接下来的所有代码，我们都将写在该该回调函数内。

对于 Mongoose 来讲，一切都是从一个称之为 **模式（Schema）** 的东西开始：

```javascript
// 定义一个 Cat 模式
const CatSchema = Mongoose.Schema({
  name: String
})
```

至此，我们定义了一个名为 `Cat` 的模式，该模式中只有一个字段 `name`，该字段只接受类型为 `String` 的数据。

```javascript
// 定义一个 Cat 模型
const Cat = Mongoose.model('Cat', CatSchema)
```

一个模型，本质就是一个 **类（Class）**，Mongoose 将使用模型来构建文档的数据结构，在上面的例子中，每一个 `Cat` 的实例都将有 `CatSchema` 模式定义的属性以及由 `Mongoose.Schema` 提供的方法与行为，现在，我们可以创建一个 `Cat` 实例：

```javascript
// 创建一个 Cat 实例
const cat = new Cat({
  name: 'Silence'
})
console.log(cat.name)
```

一只猫是可以叫出声音来的，我们现在可以定义一个猫叫的方法：

```javascript
// 在 Mongoose.model 方法调 Schema 之前，定义 speak 方法
CatSchema.methods.speak = function () {
  const greeting = this.name ? `Meow name is ${this.name}` : 'I don\'t hav a name'
  console.log(greeting)
}
```

添加至 `Schema.methods` 属性的方法，将为被添加至 `Schema` 模型 的 `prototype` 上，这使得每一个实例都将具有该方法：

```javascript
const fluffy = new Cat({
  name: 'fluffy'
})
fluffy.speak(); // Meow name is fluffy
```

至此，我们还没有将数据持久化至数据库，使用 `save` 方法即可：

```javascript
cat.save((error, cat) => {
  if (error) {
    return console.error(error)
  }
  cat.speak()
})
```

当数据都保存之后，我们有可能还需要从数据库中查询所有的 Cat 数据，使用 `find` 方法即可：

```javascript
Cat.find((error, cats) => {
  if (error) {
    return console.log(error)
  }
  console.log(cats)
})
```

查询除了直接查询所有外，我们还可以添加过滤条件：

```javascript
Cat.find({
  name: /^fluff/
}, callback)
```

在上面的查询中，我们只从所有的 Cat 数据中查询 `name` 是以 `fluff` 开头的 Cat。