# vue-cli-mock
vue-cli 添加本地mock服务框架

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# run mock serve localhost:3000
npm run mock

# run serve with mock serve
npm run mockdev
```

# mock目录
```
.
	└── mock/					# mock配置目录
    └── db.json					# mock数据配置
    └── faker-data.js			# 批量生成伪数据
    └── post-to-get.js			# post映射为get中间件
```

# 说明
[JSON Server](https://github.com/typicode/json-server) 是一个创建伪RESTful服务器的工具，具体使用方法可以看官方文档，这里直接讲在vue-cli 中的用法。
### 配置流程
- 全局安装 ``$ npm install -g json-server``
- 项目目录下创建 ``mock`` 文件夹
- ``mock`` 文件夹下添加 ``db.json`` 文件，内容如下

```
{
  "posts": [
    { "id": 1, "title": "json-server", "author": "typicode" }
  ],
  "comments": [
    { "id": 1, "body": "some comment", "postId": 1 }
  ],
  "profile": { "name": "typicode" }
}
```
- ``package.json`` 添加命令
```
"mock": "json-server --watch mock/db.json",
    "mockdev": "npm run mock & npm run dev"
```
![](http://upload-images.jianshu.io/upload_images/1651860-419846b1d54d3d24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 启动 mock 服务器
``$ npm run mock`` 命令 运行 mock server
访问 http://localhost:3000/
发现 ``db.json `` 下第一级 json 对象被解析成为可访问路径

![](http://upload-images.jianshu.io/upload_images/1651860-349bfc482aba7065.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

GET请求具体路径 如：http://localhost:3000/posts 可获取数据

![](http://upload-images.jianshu.io/upload_images/1651860-8150da6fd1f6bd54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### faker.js 批量生成伪数据
如果需要大量的伪数据，手动构造比较费时费力，可以使用 [faker.js](https://github.com/Marak/faker.js) 批量生成。faker.js 的具体使用参见官方文档，这里做一个示例。
- ``$ cnpm install faker -G`` 全局安装 faker
-  ``mock`` 目录下创建 ``faker-data.js``，内容如下
```
module.exports = function () {
    var faker = require("faker");
    faker.locale = "zh_CN";
    var _ = require("lodash");
    return {
        people: _.times(100, function (n) {
            return {
                id: n,
                name: faker.name.findName(),
                avatar: faker.internet.avatar()
            }
        }),
        address: _.times(100, function (n) {
            return {
                id: faker.random.uuid(),
                city: faker.address.city(),
                county: faker.address.county()
            }
        })
    }
}
```
- ``$ json-server mock/faker-data.js`` 在 json server 中使用 faker
请求 http://localhost:3000/address 可以获取到随机生成的100组伪数据

![](http://upload-images.jianshu.io/upload_images/1651860-3a978cbe0d6f3c11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 添加中间件
json server 使用 [RESTful 架构](http://www.ruanyifeng.com/blog/2011/09/restful)，GET请求可以获取数据，POST 请求则是添加数据。
在开发过程中，有时候想直接模拟获取POST请求返回结果，可以添加 express 中间件 将POST请求转为GET请求。
- ``mock`` 目录下创建 ``post-to-get.js``，内容如下
```
module.exports = function (req, res, next) {
  req.method = "GET";
  next();
}
```
- 启动命令添加运行中间件 ``--m mock/post-to-get.js``
```
"mock": "json-server --watch mock/db.json --m mock/post-to-get.js",
```

重新启动服务，POST请求就被转换为GET请求了

![](http://upload-images.jianshu.io/upload_images/1651860-d62321826379a90a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其他需求也可以通过添加不同的中间件实现。

### 代理设置
在 ``config/index.js`` 的 ``proxyTable`` 将请求映射到 http://localhost:3000

![](http://upload-images.jianshu.io/upload_images/1651860-1629801bae740557.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码中添加请求以测试效果

![](http://upload-images.jianshu.io/upload_images/1651860-0206a52db3368cfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


``$ npm run mockdev`` 启动带mock 数据的本地服务

结果如下：
![](http://upload-images.jianshu.io/upload_images/1651860-da5fcadfbd4a7f7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)