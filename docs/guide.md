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

更多详情，可以阅读 [SchemaTypes](schema-types.md) 文档。

