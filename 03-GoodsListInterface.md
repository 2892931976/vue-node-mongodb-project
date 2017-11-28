## 基于Node.js开发商品列表接口

> #### 一、node的启动调试方式
[http://www.cnblogs.com/ccyinghua/p/7889320.html](http://www.cnblogs.com/ccyinghua/p/7889320.html)

> #### 二、基于Express实现商品列表查询接口

mongodb安装与环境搭建，建立dumall数据库，创建goods集合，导入数据文件 （resource/dumall-goods），也可以自己手动插入。

mongodb安装与环境搭建： [http://www.cnblogs.com/ccyinghua/p/7887713.html](http://www.cnblogs.com/ccyinghua/p/7887713.html)

建立数据库和插入数据，可以进入mongo操作数据库，以命令行形式插入操作，即成功启动MongoDB后，再打开一个命令行窗口输入mongo,就可以进行数据库的一些操作；
也可以下载mongovue3.4.4（64位）客户端进行操作，客户端可以手动输入插入数据，也可以导入文件插入数据。

![image](https://github.com/ccyinghua/vue-node-mongodb-project/blob/master/resource/readme/09/1.png?raw=true)

数据库建立完成，然后就是项目与mongodb连接：

```javascript
* 安装mongoose链接数据库mongodb
* 创建model,mongoose创建了model就是一个实体，可以跟mongodb进行关联。
* 创建路由。
* 基于mongoose,实现商品列表的查询功能。
```
###### 1、安装mongoose(操作mongodb数据库的对象模型工具)

API文档： [http://mongoosejs.com/docs/guide.html](http://mongoosejs.com/docs/guide.html)

```javascript
cnpm install mongoose --save

```

###### 2、创建model

[models/goods.js](https://github.com/ccyinghua/vue-node-mongodb-project/blob/master/server/models/goods.js)--定义输出一个商品的模型model.


```javascript
var mongoose = require('mongoose');
var Schema = mongoose.Schema;
// 定义一个Schema
var produtSchema = new Schema({
	'productId':String,   // 或者 'productId':{type:String}
	'productName':String,
	'salePrice':Number,
	'productImage':String
})

// 输出(导出)
module.exports = mongoose.model('good',produtSchema); // 定义一个good商品模型，可以根据这个商品模型调用其API方法。
// 这个模型定义的是数据库dumall的goods集合数据，所以这个model取名good是对应这个集合，连接数据库之后，这个模型会根据名字的复数形式"goods"来查找数据集合。
```
###### 3、创建路由

server/routers文件夹内定义其路由，在routers/goods.js;同时server/app.js要增添上这个goods的路由

```javascript
>> app.js

var goods = require('./routes/goods');
app.use('/goods', goods);

```

```javascript
>> routers/goods.js

var express = require('express');
var router = express.Router();  // 拿到express框架的路由
var mongoose = require('mongoose');
var Goods = require('../models/goods');

// 链接MongoDB数据库,数据库的名称叫dumall
mongoose.connect('mongodb://127.0.0.1:27017/dumall');   // 若是带账号密码的：'mongodb://root:123456@127.0.0.1:27017/dumall'

// 连接成功操作
mongoose.connection.on("connected",function(){
    console.log("MongoDB connected success.")
})

// 连接失败操作
mongoose.connection.on("error",function(){
    console.log("MongoDB connected fail.")
})

// 连接断开操作
mongoose.connection.on("disconnected",function(){
    console.log("MongoDB connected disconnected.")
})

// 二级路由
/* GET goods page. */
router.get('/', function(req, res, next) {
    res.send('hello,goods list');  // 测试路由，连接成功页面出现'hello,goods list'
});

module.exports = router;


// 启动express
// node server/bin/www 或 pm2方式 或 webstorm 等
// localhost:3000/goods/    // '/goods'是app.js中的一级路由，'/'是本页的二级路由;链接成功页面出现'hello,goods list'

```
![image](https://github.com/ccyinghua/vue-node-mongodb-project/blob/master/resource/readme/09/2.png?raw=true)

路由连接成功之后，用model的good商品模型查询到数据库的goods集合。

```javascript
// 二级路由
/* GET goods page. */
router.get('/', function(req, res, next) {
    // res.send('hello,goods list');  // 测试路由，连接成功页面出现'hello,goods list'
  
    Goods.find({},function(err, doc){   // Goods来自models/goods.js;导出的是mongoose的商品模型，可使用mongoose的API方法
        if(err) {
            res.json({
                status:'1',
                msg:err.message
            })
        }else{
            res.json({
                status:'0',
                msg:'',
                result:{
                    count:doc.length,
                    list:doc
                }
            })
        }
    })

});

// 重新启动express
// 页面出现mongodb数据库里的goods集合数据信息json

```
![image](https://github.com/ccyinghua/vue-node-mongodb-project/blob/master/resource/readme/09/3.png?raw=true)