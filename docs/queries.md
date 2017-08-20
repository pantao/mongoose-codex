# 查询

我们可以使用模型的各种内置的静态方法完成数据的查询。

任何模型的方法，都能通过两种方式执行特定的查询：

1. 如果传递了回调函数，那么查询会被立即执行，并将结果传递给回调函数
2. 如果没有传递回调函数，那么会返回一个查询对象，该对象提供了一个特定的查询接口

> 在 Mongoose 4 中，查询对象还带有一个 `then()` 方法。

在查询过程中，使用 `JSON` 类型的文档来描述查询，JSON 文档的语法类似于 `MongoDB Shell`。

```javascript
var Person = mongoose.model('Person', yourSchema);

// find each person with a last name matching 'Ghost', selecting the `name` and `occupation` fields
Person.findOne({ 'name.last': 'Ghost' }, 'name occupation', function (err, person) {
  if (err) return handleError(err);
  console.log('%s %s is a %s.', person.name.first, person.name.last, person.occupation) // Space Ghost is a talk show host.
})
```

执行上面的代码我们可以看到，查询立即被执行了，并且，结果被返回给了 `callback` 函数，所有的 `callback` 函数的格式都为 `callback(error, result)`，如果查询的过程中产生的错误，那么 `error` 将是一个包含错误信息的对象，同时 `result` 将为 `null`，如果查询成功，那么 `error` 将为 `null`，同时， `result` 将会是结果。

现在让我们来看一下下如果不传递 `callback` 函数时，会发生什么：

```javascript
// find each person with a last name matching 'Ghost'
var query = Person.findOne({ 'name.last': 'Ghost' });

// selecting the `name` and `occupation` fields
query.select('name occupation');

// execute the query at a later time
query.exec(function (err, person) {
  if (err) return handleError(err);
  console.log('%s %s is a %s.', person.name.first, person.name.last, person.occupation) // Space Ghost is a talk show host.
})
```

在上面的代码中，`query` 就是一个 `Query` 对象，`Query` 对象允许你链式的创建查询：

```javascript
// With a JSON doc
Person.
  find({
    occupation: /host/,
    'name.last': 'Ghost',
    age: { $gt: 17, $lt: 66 },
    likes: { $in: ['vaporizing', 'talking'] }
  }).
  limit(10).
  sort({ occupation: -1 }).
  select({ name: 1, occupation: 1 }).
  exec(callback);
  
// Using query builder
Person.
  find({ occupation: /host/ }).
  where('name.last').equals('Ghost').
  where('age').gt(17).lt(66).
  where('likes').in(['vaporizing', 'talking']).
  limit(10).
  sort('-occupation').
  select('name occupation').
  exec(callback);
```

需要注意的是， `setters` 在查询对象中，默认并不会被执行，所以下面这样的Schema，如果我们sjyr `Person.find({email: 'Val@karpov.io'})` 的话，将查询不到任何内容。

```javascript
var personSchema = new Schema({
  email: {
    type: String,
    lowercase: true
  }
});

```

此时，我们可以在 `Schema` 定义时，设置 `runSettersOnQuery` 为 `true`即可：

```javascript
var personSchema = new Schema({
  email: {
    type: String,
    lowercase: true
  }
}, { runSettersOnQuery: true });
```

## 引用其它文档

在 MongoDB 中没有 `join` 操作，但有时候我们同样希望关联于其它 `collection` 中的文档，这可以通过 [population](population.md) 实现。

## 流

如果你想使用 `stream` 你的结果集，那么使用 `Query#cursor()` 方法替代 `Query#exec` 方法，它会返回一个 `QueryCursor` 对象：

```javascript
var cursor = Person.find({ occupation: /host/ }).cursor();
cursor.on('data', function(doc) {
  // Called once for every document
});
cursor.on('close', function() {
  // Called when done
});
```

## 接下来

了解 [校验](validation.md)