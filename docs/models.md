# 模型

模型（Model）是一些由 Schema 编译而成的各种数据类型的构造器，任何模型的实例都称之为文档，它们将被保存至数据库中，所有的数据的创建与持久化都是通过它所属的模型进行的。

## 编译你的第一个模型 

```javascript
var schema = new mongoose.Schema({ name: 'string', size: 'string' });
var Tank = mongoose.model('Tank', schema);
```

第一个参数表示的是你的模型的名称为 `Tank`，**Mongoose 会自动的转换名称的复数，并将其设置为模型的 collection 名称**，在上面的示例中，数据库中的数据集合的名称将为 `tanks`，`.model()` 方法会创建一个 `schema` 的复本，所以，在调用 `.model()` 方法前，请确定你已经设定好了 `schema` 所需要的所有内容。

## 构建文档

任何文档都是模型的实例，创建并保存一个文档至数据库是非常简单的：

```javascript
var Tank = mongoose.model('Tank', yourSchema);

var small = new Tank({ size: 'small' });
small.save(function (err) {
  if (err) return handleError(err);
  // saved!
})

// or

Tank.create({ size: 'small' }, function (err, small) {
  if (err) return handleError(err);
  // saved!
})
```

需要注意的是，在你的模型关联的数据库连接打开之前，不会有任何数据从数据库中删除或者被插入到数据库中，每一个模型在其被定义时，就已经关联上了一个默认的数据库连接，如果你想一个模型保存在某一个非默认连接的数据库中的话，请使用连接的 `.model()` 方法来创建模型 ：

```javascript
mongoose.connect('localhost', 'gettingstarted');
var connection = mongoose.createConnection('mongodb://localhost:27017/test');
var Tank = connection.model('Tank', yourSchema);
```

## 查询

Mongoose 支持富查询语法，文档可以通过模型的 `find`、`findById`、`findOne`、或者 `where` 静态方法查询：

```javascript
Tank.find({ size: 'small' }).where('createdDate').gt(oneYearAgo).exec(callback);
```

关于查询的更多详情内容，可以查阅 [查询](queries.md) 文档。

## 删除

模型都有一个静态方法 `remove` 用户删除所有文档或者删除匹配的文档：

```javascript
Tank.remove({ size: 'large' }, function (err) {
  if (err) return handleError(err);
  // removed!
});
```

## 更新

每一个模型都有其自己的 `update` 方法用于更新数据，但是它本身并不会返回被更新的结果，如果你想更新并返回已更新之后的数据，可以使用 `findOneAndUpdate` 方法。

关于更新的更多内容，可以查看 [接口](api.md) 文档中的 `Query` 章节。

## 更多

[API 文档](api.md) 有更多模型具有的方法的说明，包括 `count`、`mapReduce`、`aggregate` 等。

## 接下来

了解一下 [文档](documents.md)