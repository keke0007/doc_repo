网络请求库－axios库

## 目录

认识Axios库 content axios发送请求 axios创建实例 axios的拦截器 axios请求封装

## 认识axios

1为什么选择axios？作者推荐和功能特点 尤小右★ +关注 2016-11-322:38来自微博weibo.com 公告一下：以后vue-resource 不再是官方推荐的 ajax库了，推荐用 axios。详见 英文博客：网页链接 ☆收藏 凸61 功能特点：

- 在浏览器中发送XMLHttpRequests请求
- 在 node.js 中发送http请求
- 支持PromiseAPI 补充：axios名称的由来？个人理解
- 拦截请求和响应 没有具体的翻译.
- 转换请求和响应数据 axios: ajax i/o system.
- 等等

## axios请求方式

支持多种请求方式：

```javascript
axios(config)
axios.request(config)
axios.get(url[, config])
axios.delete(url[, config])
axios.head(url[, config])
axios.post(url[, data[, config]l)
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])
```

1有时候，我们可能需求同时发送两个请求

- 使用axios.all，可以放入多个请求的数组

## 常见的配置选项

请求地址 查询对象序列化函数

```javascript
url: '/user', paramsSerializer: function(params){ }
```

请求类型 request body

```javascript
method: 'get', data: { key: 'aa'},
```

请根路径 超时设置 baseURL: 'http://www.mt.com/api', timeout: 1000, 请求前的数据处理

```javascript
transformRequest:[function(data)0],
```

请求后的数据处理

```javascript
transformResponse: [function(data)0],
```

自定义的请求头

```javascript
headers:{'x-Requested-With':'XMLHttpRequest'},
```

URL查询对象

```javascript
params:{id: 12 },
```

## axios的创建实例

1为什么要创建axios的实例呢？

- 当我们从axios模块中导入对象时，使用的实例是默认的实例；
- 当给该实例设置一些默认配置时，这些配置就被固定下来了.
- 但是后续开发中，某些配置可能会不太一样；
- 比如某些请求需要使用特定的baseURL或者timeout等
- 这个时候，我们就可以创建新的实例，并且传入属于该实例的配置信息
```javascript
const instance = axios.create({
```

baseURL::"http://123.207.32.32:1888"

```javascript
instance.post("/o2_param/postjson",°{
```

name:."why", age:18

```javascript
}).then(res => {
console.log("res:", res)
```

## 请求和响应拦截器

axios的也可以设置拦截器：拦截每次请求和响应

```javascript
axios.interceptors.request.use(请求成功拦截，请求失败拦截)
```

- axios.interceptors.response.use(响应成功拦截，响应失败拦截)
```javascript
axios.interceptors.request.use((config) => {
console.log("请求成功拦截")
return config
err =>{
console.log("请求失败拦截")
return err
axios.interceptors.response.use((res) => {
console.log("响应成功拦截")
return·res.data
err =>{
console.log("响应失败失败")
```

## axios请求库封装(简洁版)

```javascript
class HYRequest {
constructor(baseURL)·
this.instance = axios.create({
})
request(config)·{
return new Promise((resolve,·reject) => {
this.instance.request(config).then(res => {
resolve(res.data)
}).catch(err => {
console.log("request err:", err)
reject(err)
```

H

```javascript
get(config)·{
return this.request({ ...config, method: "get" })
post(config)·{
return this.request(f ...config, method: "post" })
```
