# 概况

## 快速使用

**Mongoose** 提供了一个直观的，基于模式（Schema）的方案来解决应用程序数据的持久化任务，它包含了内置的类型转换、数据校验、查询、业务钩子（Hooks）等开箱即用的封装。

```javascript
// 加载 Mongoose 模块 load mongoose module
import Mongoose from 'mongoose'

// 连接至数据库 connect to mongodb server
Mongoose.connect('mongodb://localhost/test')

// 声明一个模型 declare Model
const Cat = Mongoose.model('Cat', {
  name: String
})

// 创建一个新的实例 create new Model instance
const kitty = new Cat({
  name: 'Zildjian'
})

// 持久化该实例 persisting an instance
kitty.save( err => {
  if (err) {
    console.log(err)
  } else {
    console.log('meow')
  }
})
```

## 开始 Mongoose 之旅

- [快速入门](docs/index.md)