# 文档

在 Mongoose 中，文档表现为一个与数据库中某一条记录一对一映射的模型的实例，每一个文档都是模型的实例。

## 检索

我们提供了很多检索文档的方法，但是这不在本文章内，要了解详情，可以查阅 [查询](queries.md) 章节。

## 更新

Mongoose 提供了很多用于更新文档的方法，我们可以先来看看使用传统方法 `findById` 来更新文档的实现：

```javascript
Tank.findById(id, function (error, tank) {
  if (error) {
    return handleError(error)
  }

  tank.size = 'large';
  tank.save(function (error, updatedTank) {
    if (error) {
      return handleError(error)
    }
    res.send(updatedTank)
  })
})
```

传统的方法是首先从MongoDb中查询出相应ID的文档，然后更新该文档的数据，接着使用 `save` 方法将更新保存至数据库中，这是一个很繁琐的实现，我们可以快速的使用 `Model.update()` 方法即可：

```javascript
Tank.udpate({_id: id}, {
  $set: {
    size: 'large'
  }
}, callback)
```

如果我们需要将更新了的文档返回，则可以使用 `findByIdAndUpdate` 方法：

```javascript
Tank.findByIdAndUpdate(id, {
  $set: {
    size: 'large'
  }
}, {
  new: true
}, function (err, tank) {
  if (err) {
    return handleError(err)
  }

  res.send(tank)
})
```

`findAndUpdate` 以及 `findAndRemove` 等类似 `findAndModify` 的方法，至少会修改一个文档的数据，但是需要注意的是，这些方法并不会在将数据更新保存至数据库前进行任何数据的校验，你可能需要自己调用 `runValidators` 选项以保存前进行有限的数据校验。

## 校验

文档在被保存至数据前都需要通过数据校验。

## 接下来

了角 [子文档](sub-documents.md)