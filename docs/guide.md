# 指引

## 模式

### 定义模式

在 Mongoose 中，任何事情都是从一个 Schema 开始的，每一个 Schema 映射至了 MongodDB 中的一个 Collection，并定义了数据在该 Collection 中的数据结构。

```javascript
import Mongoose from 'mongoose'

const {Schema} = Mongoose

const BlogSchema = new Schema({
  title: String,
  author: String,
  body: String,
  comments: [
    {
      body: String,
      date: Date
    }
  ],
  date: {
    type: Date,
    default: Date.now
  },
  hidden: Boolean,
  meta: {
    votes: Number,
    favs: Number
  }
})
```

> 如果你在一个模式被定义了之后，想再添加新的键，可以使用 `Schema.add` 方法

每一个定义在 `BlogSchema` 中的键，都将被关联至相应的 `SchemaType`，比如，我们定义了一个 `String` 类型的 `title` 字段，以及一个 `Date` 类型的 `date` 字段，键同样的定义嵌套的键（比如上面的 `meta` 字段）。

Mongoose 允许使用的 `SchemaType` 有：

- `String`：字符
- `Number`：数字
- `Date`：日期
- `Buffer`：流
- `Boolean`：布尔
- `Mixed`：混合
- `ObjectId`：MongoDB ObjectId
- `Array`：数组

更多详情，可以阅读 [SchemaTypes](schematypes.md) 文档。

Schema 不仅仅只是定义了文档的数据结构以及属性的类型，它还可以定义文档实例的方法以及模型的静态方法、索引的设置以及文档生命周期中的钩子（Hooks，称之为 Middleware）。

### 创建一个模型

要使用我们定义的 `BlogSchema`，需要先将该Schema转换成为一个 `Model`：

```javascript
const Blog = Mongoose.model(BlogSchema)
```

### 实例方法

`Models` 的实例就是文档，文档自带了很多内置的实例方法，我们也可以在定义Schema的时候，定义我们自己的实例方法：

```javascript
// 定义一个Schema
const AnimalSchema = new Schema({
  name: String,
  type: String
})

// 关联一个名为 `findSimilarTypes` 的方法至 `methods` 对象
AnimalSchema.methods.findSimilarTypes = function(cb) {
  return this.model('Animal').find({
    type: this.type,
  }, cb)
}
```

现在，所有的 `Animal` 实例都将有一个 `findSimilarTypes` 的方法了：

```javascript
const Animal = Mongoose.model('Animal', AnimalSchema)
const dog = new Animal({
  type: 'dog'
})

dog.findSimilarTypes((error, dogs) => {
  console.log(dogs)
})
```

> *请注意：重写 Mongoose 默认的文档方法，有可能导致一些不可预料的的结果。*
>
> *不要使用 ES6 的箭头函数定义模式的方法，因为箭头函数不会绑定 `this` 变量，这导致你的实例将无法访问到实例本身。*

### 静态方法

要为模型添加静态方法也是很简单的：

```javascript
// 添加方法至 `statics` 对象即可
AnimalSchema.statics.findByName = function (name, cb) {
  return this.find({
    name: new RegExp(name, 'i')
  }, cb)
}

const Animal = Mongoose.model('Animal', AnimalSchema)
Animal.findByName('fido', (error, animals) => {
  console.log(animals)
})
```

> *同样的，请不要使用 ES6 的箭头函数定义静态方法*

### 查询助手工具

你同样可以快速的添加查询方法，只需要将方法添加新 `query` 对象即可：

```javascript
AnimalSchema.query.byName = funciton (name) {
  return this.find({
    name: new RegExp(name, 'i')
  })
}

const Animal = Mongoose.model('Animal', AnimalSchema)
Animal.find().byName('fido').exec((error, animals) => {
  console.log(animals)
})
```

### 索引

MongoDB 支持二级索引，使用 Mongoose，我们可以在 Schema 的路径级别，或者模式级别定义索引，但如果要创建复合索引的话，那么就必须在模式级别定义。

```javascript
// 字段级别
const AnimalSchema = new Schema({
  name: String,
  type: String,
  tags: {
    type: [String],
    index: true
  }
})

// 模式级别
AnimalSchema.index({
  name: 1,
  type: -1
})
```

> 当你的系统启动之后，Mongoose 会自动的呼叫 `ensureIndex` 方法来创建你在Schema中定义的索引，它会调用索引序列中的每一个索引定义，同时会
> 在某一个模型的所有 `index` 定义都 `ensuured` 之后，在该模型上 `emit` 一个 `index` 事件，所以，如果是在开发过程中，为了开发效率，建
> 议将 `autoIndex` 选项设置为 `false`。

```javascript
Mongoose.connect('mongodb://user:pass@localhost:port/database', {
  config: {
    autoIndex: false
  }
})

// 或者

Mongoose.createConnection('mongodb://user:pass@localhost:port/database', {
  config: {
    autoIndex: false
  }
})

// 或者

AnimalSchema.set('autoIndex', false)

// 或者

new Schema({
  ...
}, {
  autoIndex: false
})
```

当索引创建成功或者产生错误时，都会 `emit` 一个 `index` 事件：

```javascript
AnimalSchema.index({
  _id: 1
}, {
  sparse: true
})

const Animal = Mongoose.model('Animal', AnimalSchema)

Animal.on('index', error => {
  // _id index cannot be sparse
  console.log(error.message)
})
```

### 虚拟属性

虚拟属性（Virtuals）是一些你仅仅可以获取或者设置的属性，但是他们并不会被持久化至MongoDB，这些对于一些需要复合多个字段的值组成一个值的获取与设置将十分有用：

```javascript
// 定义一个模式
const PersonSchema = new Schema({
  name: {
    first: String,
    last: String
  }
})

// 创建模型
const Person = Mongoose.model('Person', PersonSchema)

// 创建一个文档
const axl = new Person({
  name: {
    first: 'Axl',
    last: 'Rost'
  }
})
```

想象一下，你希望打印出这个人的全名：

```javascript
console.log(axl.name.first + ' ' + axl.name.last)
```

如果每一次需要使用全名的时候都这样去将两个字段的值合并，那将是十分糟糕的，这个时候，我们可以使用虚拟属性来创建一个更加简单的获取方法：

```javascript
PersonSchema.vurtial('fullName').get(function () {
  return this.name.first + ' ' + this.name.last
})
```

现在，你就可以直接使用 `fullName` 这个字段来获取他的全名了：

```javascript
console.log(axl.fullName)
```

但是如果你使用 `toJson()` 或者 `toObject()` 方法，将一个实例格式化为一个 `JSON` 字符串或者一个纯对象，默认情况下，Mongoose 并不会在结果中包含虚拟属性，但是你可以在这些方法中添加人上 `{virtuals: true}` 参数：

你同样的还可以添加一个 `setter` 方法来设置虚拟字段：

```javascript
PersonSchema.virtual('fullName').get(function () {
  return this.name.first + ' ' + this.name.last
}).set(function (value) => {
  this.name.first = value.substr(0, value.indexOf(' '))
  this.name.last = value.substr(value.indexOf(' ') + 1)
})

axl.fullName = 'William Rost'
```

虚拟字段的 `setter` 方法会在 `validation` 方法执行前先执行，所以，当你设置了 `fullName` 之后，即使 `first` 与 `last` 两个字段都为空，这也都是合法的数据。

#### 别名

有时候为了节省网络传输所占用的流量，我们可能还会使用一些别名来减少传输过程中数据的大小，比如 `name` 我们可能会使用 `n` 代替，这个时候，你可以快速的创建这些虚拟字段：

```javascript
const PersonSchema = new Schema({
  n: {
    type: String,
    alias: 'name'
  }
})

const Person = Mongoose('Person', PersonSchema)

// 设置 Person  的 name 属性，会自动的映射至 n 上
const person = new Person({
  name: 'Val'
})

console.log(person) // { n: 'Val' }
console.log(person.toObject({virtuals: true}) // { n: 'Val', name: 'Val'})
console.log(person.name) // Val

person.name = 'Not Val'
console.log(person) // { n: 'Not Val' }
```

### 选项

Schema 在定义是，有一些可用的配置选项，它可以在构架函数中或者 `set` 方法中定义：

```javascript
new Schema({...}, options)

const schema = new Schema({...})
schema.set(option, value)
```

可以的配置选项有：

- `autoIndex`
- `capped`
- `collection`
- `emitIndexErrors`
- `id`
- `_id`
- `minimize`
- `read`
- `safe`
- `shardKey`
- `strict`
- `toJSON`
- `toObject`
- `typeKey`
- `validateBeforeSave`
- `versionKey`
- `skipVersioning`
- `timestamps`
- `retainKeyOrder`