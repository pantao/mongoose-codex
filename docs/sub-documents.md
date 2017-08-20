# 子文档 

内嵌至其它文档中的文档称之为子文档（Sub Documents），在Mongoose中，即表示你可以在一个 Schema 中内嵌其它的 Schema。Mongoose 有两个方法包含子文档：一个子文档列表的数组或者一个单个子文档：

```javascript
var childSchema = new Schema({ name: 'string' });

var parentSchema = new Schema({
  // Array of subdocuments
  children: [childSchema],
  // Single nested subdocuments. Caveat: single nested subdocs only work
  // in mongoose >= 4.2.0
  child: childSchema
});
```

子文档与文档很相似，内嵌的 Schema 同样可以有`自定义中间介、自定义数据校验逻辑、虚拟字段以及其它顶级文档有的功能，而它们之间最主要的区别在于他们不能单独的进行持久化的控制，他们只能跟随其父文档的持久化而同步进行。

```javascript
var Parent = mongoose.model('Parent', parentSchema);
var parent = new Parent({ children: [{ name: 'Matt' }, { name: 'Sarah' }] })
parent.children[0].name = 'Matthew';

// `parent.children[0].save()` is a no-op, it triggers middleware but
// does **not** actually save the subdocument. You need to save the parent
// doc.
parent.save(callback);
```

子文档的 `save` 与 `validate` `middleware` 与顶层文档是一样的，当在顶层文档调用 `save()` 方法时，会同时触发所有子文档的 `save()` 方法。

```javascript
childSchema.pre('save', function (next) {
  if ('invalid' == this.name) {
    return next(new Error('#sadpanda'));
  }
  next();
});

var parent = new Parent({ children: [{ name: 'invalid' }] });
parent.save(function (err) {
  console.log(err.message) // #sadpanda
});
```

子文档的 `pre('save')` 与 `pre('validate')` 会在父文档的 `pre('save')` 之前执行，但是在父文档的 `pre('validate')` 之后。

```javascript
// Below code will print out 1-4 in order
var childSchema = new mongoose.Schema({ name: 'string' });

childSchema.pre('validate', function(next) {
  console.log('2');
  next();
});

childSchema.pre('save', function(next) {
  console.log('3');
  next();
});

var parentSchema = new mongoose.Schema({
  child: childSchema,
    });
    
parentSchema.pre('validate', function(next) {
  console.log('1');
  next();
});

parentSchema.pre('save', function(next) {
  console.log('4');
  next();
});
```

## 查找子文档

每一个子文档默认都有一个 `_id` 属性，` Mongoose` 文档数组有一个特殊的 `id` 方法，以帮助我们从子文档数组中搜索特定 `_id` 的文档。

```javascript
var doc = parent.children.id(_id);
```

## 添加新的子文档

Mongoose 子文档数组同样具有如 `push`、`unshift`、`addToSet` 等方法：

```javascript
var Parent = mongoose.model('Parent');
var parent = new Parent;

// create a comment
parent.children.push({ name: 'Liesl' });
var subdoc = parent.children[0];
console.log(subdoc) // { _id: '501d86090d371bab2c0341c5', name: 'Liesl' }
subdoc.isNew; // true

parent.save(function (err) {
  if (err) return handleError(err)
  console.log('Success!');
});
```

同样的，子文档可以不需要添加至文档数组，也可以被创建：

```javascript
var newdoc = parent.children.create({ name: 'Aaron' });
```

## 删除子文档

每一个子文档都有其自己的 `remove()` 方法：

```javascript
// Equivalent to `parent.children.pull(_id)`
parent.children.id(_id).remove();
// Equivalent to `parent.child = null`
parent.child.remove();
parent.save(function (err) {
  if (err) return handleError(err);
  console.log('the subdocs were removed');
});
```

## 不同的定义语法

如果你定义一个父Schema某一个字段的Schema为一个数组：

```javascrript
var parentSchema = new Schema({
  children: [{ name: 'string' }]
});
// Equivalent
var parentSchema = new Schema({
  children: [new Schema({ name: 'string' })]
});
```

## 接下来

了解 [查询](queries.md)