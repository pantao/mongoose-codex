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
- `bufferCommands`
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

#### option: autoIndex

当应用启动之后，Mongoose 会为每一个 Shema 的 `index` 声明发送一个 `ensureIndex` 命令至数据库，从 `Mongoose v3` 开始，`indexes` 的的创建默认都被设定为后台任务，如果你想禁用自动创建索引的功能而改用手动创建，那可以设置 `autoIndex` 这个选项：

```javascript
const FooSchema = new Schema({...}, {autoIndex: false})
const Foo = Mongoose.model('Foo', FooSchema)
Foo.ensureIndexes(callback)
```

#### option: bufferCommands

默认的，当与数据库的连接断掉之后，数据库驱动在尝试重新连接互数据库的过程中，Mongoose 会先将所有的命令都缓冲起来，当连接重新建立之后，再真实的发送这些命令，如果要关闭这个功能，可以直接设置 `bufferCommands` 功能即可：

```javascript
const FooSchema = new Schema({...}, {bufferCommands: false})
```

#### option: capped

Mongoose 支持 MongoDB 的定容集合（Capped Colleciton），你可以直接通过设置 `capped` 属性为一个以 `bytes` 为单位的数字即可;

```javascript
const FooSchema = new Schema({...}, {capped: 1024})
```

同样的，你还可以设置 `capped` 为一个对象，此时就必须设置 `size` 属性：

```javascript
const FooSchema = new Schema({...}, {capped: {size: 1024, max: 1000, autoIndex: true}})
```

#### option: collection

默认的，Mongoose 会使用 `utils.toCollectionName` 方法获取一个自动根据名称转换得到的集合名称，如果你想使用一个自定义的名称，可以使用该配置选项设置：

```javascript
const FooSchema = new Schema({...}, {collection: 'bar'})
```

#### option: emitIndexErrors

默认的，Mongoose 会在创建索引成功或者失败时，都 `emit` 一个 `index` 事件，如果你想如果 `index` 事件外，在产生错误时， `emit` 一个 `error` 事件，那么可以设置该选项为 `true`

```javascript
MyModel.schema.options.emitIndexErrors // true
MyModel.on('error', error => {
  console.log(error) // 当 index 创建失败时，这里会被执行
})
```

#### option: id

Mongoose 会自动的为每一个文档创建一个 `id` 虚拟属性，映射至 `_id` 字段，该字段只允许被 `get` 不允许被 `set`。如果要禁该功能，设置 `id` 选项为 `false` 即可：

```javascript
// 默认行为
const FooSchema = new Schema({
  name: String
})

const Foo = Mongoose.model('Foo', FooSchema)
const foo = new Foo({
  name: 'Mongoose'
})
console.log(foo.id) // 50341373e894ad16347efe01

// 禁用 id 后的行为
const BarSchema = new Schema({
  name: String
})

const Bar = Mongoose.model('Bar', BarSchema)
const bar = new Bar({
  name: 'Mongoose'
})
console.log(bar.id) // undefined
```

#### option: _id

默认的，Mongoose 会为每一个 Schema 都添加一个 `_id` 属性，类型为 `ObjectID` 类型，如果你不想使用这个功能，那么可以禁用它，但是你 **仅能在子文档中禁用该功能**，因为Mongoose无法保存一个没有索引键的文档：

```javascript
// 默认行为
const PageSchema = new Schema({title: String})
const Page = Mongoose.model('Page', PageSchema)
const page = new Page({
  title: 'Default Behavior'
})

console.log(page) // {_id: '50341373e894ad16347efe01', title: 'Default Behavior'}

// 禁用
const CommentSchema = new Schema({subject: String}, {_id: false})
const PageSchema = new Schema({title: String, comments: [CommentSchema]})
const Page = Mongoose.model('Page', PageSchema)

const page = new Page({
  title: 'Disabled',
  comments: [
    {subject: 'no subject'}
  ]
})

console.log(page.comments[0]._id) // undefined
```

#### option: minimize

是否最小化文档，默认情况下，在保存至数据库，Mongoose都会检查文档的属性是否为空，如果为空，则会在保存前删除该字段，以保证文档占用的空间总是最小的，但是你也可以禁用该功能，只需要设置 `minimize` 选项为 `false` 即可：

```javascript
var schema = new Schema({ name: String, inventory: {} });
var Character = mongoose.model('Character', schema);

// 仅在 `inventory` 不为空的前提下才保存该字段的值
var frodo = new Character({ name: 'Frodo', inventory: { ringOfPower: 1 }});
Character.findOne({ name: 'Frodo' }, function(err, character) {
  console.log(character); // { name: 'Frodo', inventory: { ringOfPower: 1 }}
});

// 若 `inventory` 为空，则不会被保存
var sam = new Character({ name: 'Sam', inventory: {}});
Character.findOne({ name: 'Sam' }, function(err, character) {
  console.log(character); // { name: 'Sam' }
});
```

禁用该功能之后：

```javascript
var schema = new Schema({ name: String, inventory: {} }, { minimize: false });
var Character = mongoose.model('Character', schema);

// 不管 `inventory` 是否为空，都将被保存
var sam = new Character({ name: 'Sam', inventory: {}});
Character.findOne({ name: 'Sam' }, function(err, character) {
  console.log(character); // { name: 'Sam', inventory: {}}
});
```

#### opiton: read

允许我们设置MongoDB的 `ReadPreferences` 属性：

```javascript
var schema = new Schema({..}, { read: 'primary' });            // also aliased as 'p'
var schema = new Schema({..}, { read: 'primaryPreferred' });   // aliased as 'pp'
var schema = new Schema({..}, { read: 'secondary' });          // aliased as 's'
var schema = new Schema({..}, { read: 'secondaryPreferred' }); // aliased as 'sp'
var schema = new Schema({..}, { read: 'nearest' });            // aliased as 'n'
```

#### option: safe

#### option: shardKey

#### option: strict

默认值为 `true`，它表示是否严格验证属性及其值，如果该项设置为 `true`，则未在 Schema 中定义的字段将不会被保存：

```javascript
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing({ iAmNotInTheSchema: true });
thing.save(); // iAmNotInTheSchema 将不会被保存

// set to false..
var thingSchema = new Schema({..}, { strict: false });
var thing = new Thing({ iAmNotInTheSchema: true });
thing.save(); // iAmNotInTheSchema 将会被保存
```

该选项同样会影响  `doc.set()` 方法：

```javascript
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing;
thing.set('iAmNotInTheSchema', true);
thing.save(); // iAmNotInTheSchema 将不会被保存
```

该项设置可以通过模型实例化方法的第二参数复写：

```javascript
var Thing = mongoose.model('Thing');
var thing = new Thing(doc, true);  // enables strict mode
var thing = new Thing(doc, false); // disables strict mode
```

`strict` 还可以设置为 `throw`，此时，如果数据格式不匹配时，并不会直接丢掉数据，而是会抛出异常。

- *注意*：如果你没有一个好的理由，请不要设置为 `false`
- *注意*：在 Mongoose v2 中，该值默认为 `false`
- *注意*：使用在 Schema 中未定义的字段，在使用 `set` 方法设置时，都会被直接抛弃

```javascript
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing;
thing.iAmNotInTheSchema = true;
thing.save(); // iAmNotInTheSchema is never saved to the db
```

#### option: toJSON

控制 `toJSON` 的行为：

```javascript
var schema = new Schema({ name: String });
schema.path('name').get(function (v) {
  return v + ' is my name';
});
schema.set('toJSON', { getters: true, virtuals: false });
var M = mongoose.model('Person', schema);
var m = new M({ name: 'Max Headroom' });
console.log(m.toObject()); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom' }
console.log(m.toJSON()); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }
// since we know toJSON is called whenever a js object is stringified:
console.log(JSON.stringify(m)); // { "_id": "504e0cd7dd992d9be2f20b6f", "name": "Max Headroom is my name" }
```

#### opiton: toObject

每一个实例都有一个 `toObject` 方法，用于将实例转换为纯 JavaScript 对象，该配置用于控制 `toObject` 的行为：

```javascript
var schema = new Schema({ name: String });
schema.path('name').get(function (v) {
  return v + ' is my name';
});
schema.set('toObject', { getters: true });
var M = mongoose.model('Person', schema);
var m = new M({ name: 'Max Headroom' });
console.log(m); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }
```

#### option: typeKey

默认情况下，当你在定义 Schema 时，`type` 表示的是当前定义的字段是什么类型的数据，但是如果你有一个字段名称就想叫 `type`，为了避免歧义，我们可以修改 **定义当前字段为什么类型的属性名称**，这个时候就可能使用这个选项来配置：

```javascript
// Mongoose 认为 'loc 是一个字符串'
var schema = new Schema({ loc: { type: String, coordinates: [Number] } });
```

但是如果我们的 `geoJSON` 有一个 `type` 属性的话，那可以像下面这样：

```javascript
var schema = new Schema({
  // Mongoose interpets this as 'loc is an object with 2 keys, type and coordinates'
  loc: { type: String, coordinates: [Number] },
  // Mongoose interprets this as 'name is a String'
  name: { $type: String }
}, { typeKey: '$type' }); // A '$type' key means this object is a type declaration
```

#### option: validateBeforeSave

默认的，Mongoose 会在文档保存前，自动对文档进行数据校验，但是如果你想手工校验数据，那么可以设置此项为 `false`：

```javascript
var schema = new Schema({ name: String });
schema.set('validateBeforeSave', false);
schema.path('name').validate(function (value) {
    return v != null;
});
var M = mongoose.model('Person', schema);
var m = new M({ name: null });
m.validate(function(err) {
    console.log(err); // Will tell you that null is not allowed.
});
m.save(); // Succeeds despite being invalid
```

#### option: versionKey

用于修改文档版本号字段的名称：

```javascript
var schema = new Schema({ name: 'string' });
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'mongoose v3' });
thing.save(); // { __v: 0, name: 'mongoose v3' }

// customized versionKey
new Schema({..}, { versionKey: '_somethingElse' })
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'mongoose v3' });
thing.save(); // { _somethingElse: 0, name: 'mongoose v3' }
```

该项默认值为 `__v`，如果设置该项的值为 `false`，则会直接关闭文档的版本化功能：

```javascript
new Schema({..}, { versionKey: false });
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'no versioning please' });
thing.save(); // { name: 'no versioning please' }
```

但是，你要明确的知道你正在做什么，否则，请不要关闭版本管理功能。

#### option: skipVersioning

#### option: timestamps

用于修改 `updatedAt` 与 `createdAt` 的字段名称：

```javascript
var thingSchema = new Schema({..}, { timestamps: { createdAt: 'created_at' } });
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing();
thing.save(); // `created_at` & `updatedAt` will be included
```

#### option: useNestedStrict

在 Mongoose 4中，`update()` 和 `findOneAndUpdate()` 默认只对顶层Shema进行检测：

```javascript
var childSchema = new Schema({}, { strict: false });
var parentSchema = new Schema({ child: childSchema }, { strict: 'throw' });
var Parent = mongoose.model('Parent', parentSchema);
Parent.update({}, { 'child.name': 'Luke Skywalker' }, function(error) {
  // Error because parentSchema has `strict: throw`, even though
  // `childSchema` has `strict: false`
});

var update = { 'child.name': 'Luke Skywalker' };
var opts = { strict: false };
Parent.update({}, update, opts, function(error) {
  // This works because passing `strict: false` to `update()` overwrites
  // the parent schema.
});
```

如果设置  `useNestedStrict` 为 `true`，则Mongoose 会使用子Schema的 `strict` 选项进行设置：

```javascript
var childSchema = new Schema({}, { strict: false });
var parentSchema = new Schema({ child: childSchema },
  { strict: 'throw', useNestedStrict: true });
var Parent = mongoose.model('Parent', parentSchema);
Parent.update({}, { 'child.name': 'Luke Skywalker' }, function(error) {
  // Works!
});
```

#### option: retainKeyOrder

## 接下来

我们来了解一下 [SchemaTypes](schematypes.md)