# SchemaTypes

ShemaTypes 用于处理路径的 **默认值**、**校验**、**getters**、**setters**、**在查询中字段默认是否提供** 以及一些其它特性等。

下面是所有被Mongoose支持的ShemaTypes：

- String
- Number
- Date
- Buffer
- Boolean
- Mixed
- ObjectID
- Array

## 示例

```javascript
var schema = new Schema({
  name:    String,
  binary:  Buffer,
  living:  Boolean,
  updated: { type: Date, default: Date.now },
  age:     { type: Number, min: 18, max: 65 },
  mixed:   Schema.Types.Mixed,
  _someId: Schema.Types.ObjectId,
  array:      [],
  ofString:   [String],
  ofNumber:   [Number],
  ofDates:    [Date],
  ofBuffer:   [Buffer],
  ofBoolean:  [Boolean],
  ofMixed:    [Schema.Types.Mixed],
  ofObjectId: [Schema.Types.ObjectId],
  ofArrays:   [[]]
  ofArrayOfNumbers: [[Number]]
  nested: {
    stuff: { type: String, lowercase: true, trim: true }
  }
})

// example use

var Thing = mongoose.model('Thing', schema);

var m = new Thing;
m.name = 'Statue of Liberty';
m.age = 125;
m.updated = new Date;
m.binary = new Buffer(0);
m.living = false;
m.mixed = { any: { thing: 'i want' } };
m.markModified('mixed');
m._someId = new mongoose.Types.ObjectId;
m.array.push(1);
m.ofString.push("strings!");
m.ofNumber.unshift(1,2,3,4);
m.ofDates.addToSet(new Date);
m.ofBuffer.pop();
m.ofMixed = [1, [], 'three', { four: 5 }];
m.nested.stuff = 'good';
m.save(callback);
```

## SchemaType 选项

你可以使用一个直接指定某一个字段的类型，或者使用一个对象表示字段的SchmeType定义：

```javascript
var schema1 = new Schema({
  test: String // `test` is a path of type String
});

var schema2 = new Schema({
  test: { type: String } // `test` is a path of type string
});
```

通过一个对象定义一个字段的ShemaType，`type` 属性设置了该字段的类型，还可以使用其它的字段来定义这个字段更多的属性，比如你想定义一个字段在保存前都需要处理成小写字母：

```javascript
var schema2 = new Schema({
  test: {
    type: String,
    lowercase: true // Always convert `test` to lowercase
  }
});
```

`lowercase` 仅对类型为 `String` 的字段有效

### 所有可用的字段 SchemaTypes 设置选项：

- `required`：Boolean 类型，表示是否必须存在
- `default`：任何符合 `type` 字段定义的类型的值作为默认值，如果该项设置为一个 `function`，那么该函数的返回值将作为默认值
- `select`：Boolean类型，表示查询时，该字段是否用于文档的 `projection`
- `validate`：为该字段添加一个 `validator function`
- `get`：通过 `Object.defineProperty()` 方法定义一个自定义的 `getter` 方法
- `set`：通过 `Object.defineProperty()` 方法定义一个自定义的 `setter` 方法
- `alias`：在 `Mongoose >= 4.10.0` 版本之后，可以使用该设置为字段定义一个别名

```javascript
var numberSchema = new Schema({
  integerOnly: {
    type: Number,
    get: v => Math.round(v),
    set: v => Math.round(v),
    alias: 'i'
  }
});

var Number = mongoose.model('Number', numberSchema);

var doc = new Number();
doc.integerOnly = 2.001;
doc.integerOnly; // 2
doc.i; // 2
doc.i = 3.001;
doc.integerOnly; // 3
doc.i; // 3
```

### 索引

你还可以定义 MongoDB的索引。

- `index`：布尔值，表示是否在当前字段上定义一个索引
- `unique`：布尔值，是否唯一
- `sparse`：布尔值，是否为 `sparse index`

```javascript
var schema2 = new Schema({
  test: {
    type: String,
    index: true,
    unique: true // Unique index. If you specify `unique: true`
    // specifying `index: true` is optional if you do `unique: true`
  }
});
```

### String

- `lowercase`：boolean，是否总是使用 `.toLowerCase()`
- `uppercase`：boolean，是否总蛤歌坛 `.toUpperCase()`
- `trim`：是否总是使用 `.trim()`
- `match`：RegExp，创建一个正则表达式的 `validator`
- `enum`：Array，枚举值列表

### Number

- `min`：最小值
- `max`：最大值

### Date

- `min`：最小日期 
- `max`：最大日期

## 使用

### Dates

内置的方法，没有进行过任何修改，所以，当你使用类似 `setMonth()` 的方法时，它的值并不会真的被设置或者存储，如果你需要修改该值，那么，需要手工的调用 `doc.markModified('pathToYourDate')` 以表示该值被修改过。

```javascript
var Assignment = mongoose.model('Assignment', { dueDate: Date });
Assignment.findOne(function (err, doc) {
  doc.dueDate.setMonth(3);
  doc.save(callback); // THIS DOES NOT SAVE YOUR CHANGE
  
  doc.markModified('dueDate');
  doc.save(callback); // works
})
```

### Mixed

这是一个 **什么值都可以的** 类型，但是它却比较难去管理，它可以通过指定字段的类型为 `Schema.Types.Mixed` 或者直接设置类型为 `{}`，下面这些都是等价的：

```javascript
var Any = new Schema({ any: {} });
var Any = new Schema({ any: Object });
var Any = new Schema({ any: Schema.Types.Mixed });
```

你可以传递任何类型的数据给 `Mixed` 类型的字段，但是由于它是不受 Mongoose 跟踪的，所以你必须手动的标记它的值是否被修改过，使用 `doc.markModified('anything')` 即可：

```javascript
person.anything = { x: [3, 4, { y: "changed" }] };
person.markModified('anything');
person.save(); // anything will now get saved
```

### ObjectIds

你可以使用 `Schema.Types.ObjectId` 标记一个字段为 `MongoDB.ObjectId` 类型：

```javascript
var mongoose = require('mongoose');
var ObjectId = mongoose.Schema.Types.ObjectId;
var Car = new Schema({ driver: ObjectId });
// or just Schema.ObjectId for backwards compatibility with v2
```

### Arrays

定义字段类型为多个 `SchemaTypes` 或者 `SubSchema` 数组：

```javascript
var ToySchema = new Schema({ name: String });
var ToyBox = new Schema({
  toys: [ToySchema],
  buffers: [Buffer],
  string:  [String],
  numbers: [Number]
  // ... etc
});
```

但是需要你注意的是，如果你定义一个字段为一个空数组，那等同于定义它为 `Mixed`，下面的所有定义都是创建一个 `Mixed` 数组：

```javascript
var Empty1 = new Schema({ any: [] });
var Empty2 = new Schema({ any: Array });
var Empty3 = new Schema({ any: [Schema.Types.Mixed] });
var Empty4 = new Schema({ any: [{}] });
```

所有 `Array` 类型的字段都会自动的有一个默认的值：`[]`

```javascript
var Toy = mongoose.model('Test', ToySchema);
console.log((new Toy()).toys); // []
```

如果你要修改它的默认值，那么你需要明确的指定它：

```javascript
var ToySchema = new Schema({
  toys: {
    type: [ToySchema],
    default: undefined
  }
});
```

如果你一个数组被标记为 `required`，那么它必须至少包含一个元素：

```javascript
var ToySchema = new Schema({
  toys: {
    type: [ToySchema],
    required: true
  }
});
var Toy = mongoose.model('Toy', ToySchema);
Toy.create({ toys: [] }, function(error) {
  console.log(error.errors['toys'].message); // Path "toys" is required.
});
```

## 创建自定义类型

Mongoose 同样的允许你创建自己的数据类型，比如 [mongoose-long](https://github.com/aheckmann/mongoose-long)、[mongoose-int32](https://github.com/vkarpov15/mongoose-int32)以及 [其它](https://github.com/aheckmann/mongoose-function) [类型](https://github.com/OpenifyIt/mongoose-types)，如果你需要创建你自己的数据类型，可以阅读 [《自定义 Schema Type》](custom-schematypes.md)。

## `schema.path()` 函数

`schema.path()` 函数会返回给定的字段路径的 `SchemaType`：

```javascript
var sampleSchema = new Schema({ name: { type: String, required: true } });
console.log(sampleSchema.path('name'));
// Output looks like:
/**
 * SchemaString {
 *   enumValues: [],
 *   regExp: null,
 *   path: 'name',
 *   instance: 'String',
 *   validators: ...
 */
 ```

 ## 接下来

 了解 [模型](models.md)