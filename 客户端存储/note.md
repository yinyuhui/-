## 1.0 前言

本文是《客户端存储技术》阅读笔记，标题与该书基本保持一致

## 1.1 客户端存储优缺点

优点：

-   直接访问数据，比从服务端获取更快
-   节省网络流量，一次请求可以使用多次
-   减轻服务器压力，避免重复多次请求
-   创建离线应用程序成为可能

缺点：

-   数据同步问题
-   存储限制模糊，无法确定可使用资源量
-   没有专业数据库功能强大

## 1.2 Cookie

Cookie 会被带在请求头和响应头在客户端和服务端来回传递，因此存放几十个键值对即可，不建议大数据量

不需要服务端知晓的数据不建议存放在 Cookie 中，可以存用户偏好和基础信息等

Cookie 自身使用方法：

-   存。键值对、其他设置如过期时间、可用域名等，分号分隔

    ```JS 
    document.cookie = "name=yyh; expires=Fri, 31 Dec 9999 23:59:59 GMT; domain=app.foo.com"
    ```

-   取 。获取的是所有 Cookie 内容，需自行解析

    ```
    document.cookie
    ```

-   删。将过期时间修改为过去即可

    ```JS 
    document.cookie = "name=yyh; expires=Thu, 01 Jan 1970 00:00:00 GMT"
    ```

使用时上述方式存取 Cookie 很不方便，因此项目中可以使用[ js-cookie](https://github.com/js-cookie/js-cookie) 库

## 1.3 Web 存储

Web 存储指的是 Local Storage（本地存储） 和 Session Storage（会话存储），可存储较大数据量，5-10M

不支持复杂数据，只支持字符串（数组或对象可编码为 Json 后存储），适用于存储用户偏好等简单信息

数据变化时会触发 window 上的 storage 事件，可获取 key、newValue、oldValue 等**（必须是两个同源 tab，一个 tab 中修改数据，另一个 tab 中触发！同一个 tab 中监听不到变化）**

本地存储只要用户不删除就会一直存在

会话存储只要浏览器关闭就会清除

使用方法（Local Storage 和 Session Storage 相同）：

-   存

    ```JS 
    localStorage.setItem('name', 'yyh')
    sessionStorage.setItem('name', 'yyh')
    ```

-   取

    ```JS 
    localStorage.getItem('name')
    ```

-   删

    ```JS 
    localStorage.removeItem('name')
    ```

-   清除所有键值对

    ```
    localStorage.clear()
    ```

## 1.4 indexedDB

indexedDB 是一个存储系统，可以存储想要存储的任意数据，移动端支持不够友好

### 1.4.1 关键术语

-   数据库： 存储数据的地方，一般情况下，一个网站创建一个数据库即可
-   对象存储：无严格数据格式要求，add 时必须是个对象，至于对象的值可以是任意类型
-   索引：通过该索引过滤所需数据

### 1.4.2 操作数据前准备

1. 使用前需先检测浏览器是否支持

```JS 
if('indexedDB' in window)
```

2. 如果支持，就可以打开数据库了

```JS
var openResult = indexedDB.open(dbName, dbVersionNumber)

// 首次访问数据库或版本号变化时触发
openResult.onupgradeneeded = (e) => {
  // 创建对象存储
}

// 成功
openResult.onsuccess = (e) => { db = e.target.result }

// 失败
openResult.onerror = (e) => {}
```

3. 创建对象存储

```JS 
openResult.onupgradeneeded = (e) => {
  // 数据库对象
  db = e.target.result
  if(!db.objectStoreNames.contains('Person')) {
    // 创建对象存储，第二个参数是配置属性可不传，此处将自增值赋给 id 作为主键
    let PersonOS = db.createObjectStore('Person', { keyPath: 'id', autoIncrement: true })
    // 创建索引 [索引名，数据属性，options（指定如何操作属性）]，也可以不创建索引
    PersonOS.createIndex('age', 'age', { unique: false })
  }
}
```

如果需要更新对象存储，则需先更改版本号，再在这个事件触发时更新

### 1.4.3 操作数据

#### 1.4.3.1 创建（新增）数据（add）

创建数据需在 `onsuccess` 事件中，获取的 db 对象可以作为全局变量，便于后续添加数据使用

```JS
openResult.onsuccess = (e) => {
	// 监听事件并创建数据（此处仅展示添加数据的代码）
  const db = e.target.result
  // 获取事务
  const transaction = db.
  ('Person', 'readwrite')
  // 获取事务

  对象存储
  const store = transaction.objectStore('Person')
  const person = {
    name: 'yyh',
    age: 18
  }
  const request = store.add(person)
  request.onsuccess = e => {}
  request.onerror = e => {}
}
```

#### 1.4.3.2 读取数据（get）

读取一条数据，本方法中 db 为上述存储的全局变量，get 方法参数是之前设置的主键的值，本例中是 id 的值

```JS 
getPersonById() {
	const transaction = db.transaction('Person', 'readonly')
  const store = transaction.objectStore('Person')
  const request = store.get(1) // 主键的值
  request.onsuccess = e => {
    const result = e.target.result  // result 就是查询的结果
  }
  request.onerror = e => {}
}
```

#### 1.4.3.3 更新数据（put）

更新数据与创建数据基本是一样的，更新某一项时，必须要把整个对象传入 put 方法，不能只传一个字段

如果没有对应的主键就会新增一条

```JS
update() {
  const transaction = db.transaction('Person', 'readwrite')
  const store = transaction.objectStore('Person')
  const person = {
    ...getPersonResult, // 读取数据的结果，包含了主键 id 的值
    age: 16， // 这是要更新的字段
  }
  const request = store.put(person)
  request.onsuccess = e => {}
  request.onerror = e => {}
}
```

#### 1.4.3.4 删除数据（delete）

传入要删除的主键值即可，本例中要删除的是 id = 2 那条

```JS
const transaction = db.transaction('Person', 'readwrite')
const store = transction.objectStore('Person')
const request = store.delete(2)
request.onsuccess = e => {}
request.onerror = e => {}
```

### 1.4.4 获取所有数据

在对象存储上打开游标（cursor）来获取所有数据

```JS
getAll() {
  const transaction = db.transaction('Person', 'readonly')
  const store = transaction.objectStore('Person')
  // 打开游标，注意是在对象存储上打开的
  const cursor = store.openCursor()
  // 结果列表
  let resultList = []
  cursor.onsuccess = e => {
    const cursor = e.target.result
    // cursor.value 中就是一条查询结果
    resultList.push(cursor.value)
    // 如果数据没读取结束，继续读下一条
    cursor.continue()
  }
  transaction.oncomplete = () {
    // 数据获取结束，可以对结果操作了
  }
}
```

### 1.4.5 获取部分数据

在索引上打开游标来获取部分数据

范围由 IDBKeyRange API 创建，提供四个函数：lowerBound、upperBound、bound、only，includes 也提供了但 IE 不支持。默认是闭区间，默认遍历顺序是 forward

-   lowerBound。参数：[lower, boolean = false] ，boolean = true 则为开区间
-   upperBound。参数：[upper, boolean = false]
-   bound。参数：[lower, upper, lowerbool, upperbool]

```js
filter() {
	// 假设lower、upper都通过页面输入获取了
  // 如果没有输入范围，不过滤
  if(!lower && !upper) return

  // 确定范围
  let range
  // 如果上下限都输入了
  if(lower && upper) {
    range = IDBKeyRange.bound(lower, upper)
  } else if(lower) {
    // 只输入了下限
    range = IDBKeyRange.lowerBound(lower)
  } else {
    // 只输入了上限
    range = IDBKeyBound.upperBound(upper)
  }

  const transaction = db.transaction('Person', 'readonly')
  const store = transaction.objectStore('Person')
  const index = store.index('age') // age 是之前创建好的索引

  // 结果列表
  let resultList = []

  // 打开游标 第二个参数确定遍历顺序，默认顺序，逆序传'prev'
  const cursor = index.openCursor(range)
  cursor.onceccess = e => {
    const cursor = e.target.result
    if(cursor) {
      resultList.push(cursor.value)
      cursor.continue()
    }
  }

  // 查询结束
  transaction.oncomplete = e => {
    console.log(resultList)
  }
}
```

### 1.4.6 补充

如果要在一个值为数组的字段上建立索引，options 中加上 multiEntry: true 即可，如

```js
db.createIndex('hobbies', 'hobbies', { unique: false, multiEntry: true })
```

如果要获取对象存储中的数据量，直接通过 count()即可

```js
const count = db.transaction('Person', 'readonly').objectStore('Person').count()
count.onsuccess = (e) => {
	let count = e.target.result // 这就是数据量
}
```
