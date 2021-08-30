title: HTTP 条件式请求
speaker: Ian Zhang

<slide class="bg-black-blue aligncenter" image="https://source.unsplash.com/C1HhAQrbykQ/ .dark">

# HTTP 条件式请求 {.text-landing.text-shadow}

By Ian Zhang {.text-intro}

[:fa-github: Github](https://github.com/TAUnionOtto/Blog/blob/master/11.HTTP%20-%20%E6%9D%A1%E4%BB%B6%E5%BC%8F%E8%AF%B7%E6%B1%82%E4%B8%8E%E7%BC%93%E5%AD%98.md){.button.ghost}

<slide :class="size-80 aligncenter">

## HTTP 到底传输了什么

---

:::flexblock

Request

```http {.animated.fadeInUp}
GET / HTTP/1.1
Host: www.example.com


```

---

Response

```http {.animated.fadeInUp}
HTTP/1.1 200 OK
Date: Mon, 23 May 2005 22:38:34 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 155
Last-Modified: Wed, 08 Jan 2003 23:11:55 GMT
Server: Apache/1.3.3.7 (Unix) (Red-Hat/Linux)
ETag: "3f80f-1b6-3e1cb03b"
Accept-Ranges: bytes
Connection: close

<html>
  <head>
    <title>An Example Page</title>
  </head>
  <body>
    <p>Hello World, this is a very simple HTML document.</p>
  </body>
</html>

```

:::

<slide :class="size-80">

## 在无状态协议中提供有状态的服务

---

### 无状态的协议{style="font-size: 32px"}

- 对单次请求的处理，不依赖其他请求
- 不会跟踪任何请求端的状态信息
- 历史上的 web 服务本身就是纯静态的
- 便于扩展与高并发，利于缓存和均衡负载，解耦请求与处理请求的过程合……

---

### 有状态的服务{style="font-size: 32px"}

- 请求中携带所有的“前提”
- 对于文件，使用<span style="text-decoration: underline;">条件式请求</span>或类似技术
- 更高要求的动态网站需要更精确、简便的方法描述这个“前提”，比如使用 cookie、token 等

<slide :class="size-80">

## HTTP 条件请求（Conditional Requests）

---

在 HTTP 协议中有一个“条件式请求”的概念，在这类请求中，请求的结果，甚至请求成功的状态，都会随着验证器与受影响资源的比较结果的变化而变化。这类请求可以用来验证缓存的有效性，省去不必要的控制手段，以及验证文件的完整性，例如在断点续传的场景下或者在上传或者修改服务器端的文件的时候避免更新丢失问题。{.text-intro}

- 受影响的资源
- 验证器
- 如何比较
- 请求的结果和状态
- 具体的例子

<slide :class="size-80">

## 受影响的资源

---

- 对于安全（safe）方法来说，例如 `GET`，通常用来获取文件，条件请求可以被用来限定仅在满足条件的情况下返回文件。这样可以节省带宽。{.text-intro}
- 对于非安全（unsafe）方法来说，例如 `PUT` 方法，通常用来上传文件，条件请求可以被用来限定仅在满足文件的初始版本与服务器上的版本相同的条件下才会将其上传。{.text-intro}

<slide :class="size-80">

## 验证器（Validators）

---

请求中会传递一个描述资源版本的值。这些值称为“验证器”，并且分为两大类：

- 文件的最后修改时间，即 `last-modified` （最后修改）时间。
- 一个意义模糊的字符串，指代一个独一无二的版本，称为“实体标签”，或者 `etag` 。

<slide :class="size-80">

## 如何比较

---

一些被称为条件首部（Conditional Header）的 HTTP Header，可以引发条件请求。当条件匹配成功时，服务器返回或操作相应的资源；反之不返回或操作相应资源，而是提示客户端进行其他操作。

- `If-Match` 如果远端资源的实体标签与在 `ETag` 这个首部中列出的某个值相同的话，表示条件匹配成功。
- `If-None-Match` 如果远端资源的实体标签与在 `ETag` 这个首部中列出的值都不相同的话，表示条件匹配成功。
- `If-Modified-Since` 如果远端资源的 `Last-Modified` 首部标识的日期比在该首部中列出的值要更晚，表示条件匹配成功。
- `If-Unmodified-Since` 如果远端资源的 `Last-Modified` 首部标识的日期比在该首部中列出的值要更早或相同，表示条件匹配成功。
- `If-Range` 作用于增量下载时的特殊首部。比较逻辑与 `If-Match` 或 `If-Unmodified-Since` 相似，但只能匹配一个 `etag` 或 `last-modified`。

<slide :class="size-80">

## 应用场景

<slide :class="size-80">

### 缓存更新

---

初始状态，没有缓存

![conditional-requests-refresh-cache-1](./static/conditional-requests-refresh-cache-1.png){style="background: #fff; padding: 32px; border-radius: 32px"}

<slide :class="size-80">

`If-None-Match` 和 `If-Modified-Since` 匹配**失败**，服务器返回 304

![conditional-requests-refresh-cache-2](./static/conditional-requests-refresh-cache-2.png){style="background: #fff; padding: 32px; border-radius: 32px"}

<slide :class="size-80">

`If-None-Match` 或 `If-Modified-Since` 匹配**成功**，服务器返回 200

![conditional-requests-refresh-cache-3](./static/conditional-requests-refresh-cache-3.png){style="background: #fff; padding: 32px; border-radius: 32px"}

<slide :class="size-80">

### 增量下载的完整性

---

第一次发起 `GET /doc` 请求

服务端返回前 2782 个字节，并通过 `Accept-Ranges: bytes` 告诉客户端：“我支持范围请求，范围请求的单位是 bytes”

![conditional-requests-partial-download-1](./static/conditional-requests-partial-download-1.png){style="background: #fff; padding: 32px; border-radius: 32px"}

<slide :class="size-80">

第二次发起 `GET /doc` 请求，携带 `Ranges: 2783-` 请求之后的内容

`If-Match` 和 `If-Unmodified-Since` 匹配**成功**，服务器返回 206，返回第 2783-5678 字节

![conditional-requests-partial-download-2](./static/conditional-requests-partial-download-2.png){style="background: #fff; padding: 32px; border-radius: 32px"}

<slide :class="size-80">

如果 `If-Match` 和 `If-Unmodified-Since` 匹配**失败**，服务器返回 412

通常客户端会重新发起 `GET /doc` 请求，从头开始请求最新的文件（当然客户端也可以选择直接报错，这取决于具体的业务场景）

![conditional-requests-partial-download-3](./static/conditional-requests-partial-download-3.png){style="background: #fff; padding: 32px; border-radius: 32px"}

<slide :class="size-80">

也可以使用 `If-Range`，省略上图中的 `/1/` 和 `/2/`，减少反复通信的开销

匹配**成功**，服务器返回 206；匹配**失败**，服务器返回 200，并携带最新版本的文件

![conditional-requests-partial-download-4](./static/conditional-requests-partial-download-4.png){style="background: #fff; padding: 32px; border-radius: 32px"}

<slide :class="size-80">

### 资源的首次上传

---

![conditional-requests-double-create.png](./static/conditional-requests-double-create.png){style="background: #fff; padding: 32px; border-radius: 32px"}

<slide :class="size-80">

### 并发更新的乐观锁

---

考虑这样一种情况：你维护一个 wiki 系统，用户可以对文件进行读取，然后在本地编辑后，发起 `PUT` 请求修改这一页面

![conditional-requests-lost-update-1](./static/conditional-requests-lost-update-1.png){style="background: #fff; padding: 32px; border-radius: 32px"}

<slide :class="size-80">

那么当两个用户并发的修改同一个页面的时候，一定会产生竞态。要如何解决？

:::div {style="display: flex; flex-direction: row; align-items: center;"}

![conditional-requests-lost-update-2](./static/conditional-requests-lost-update-2.png){style="background: #fff; padding: 32px; margin-right: 128px; border-radius: 32px"}

- `If-Match`
- `If-None-Match`
- `If-Modified-Since`
- `If-Unmodified-Since`

:::

<slide :class="size-80">

答案：锁起来，仅允许第一个请求成功执行

![conditional-requests-lost-update-3](./static/conditional-requests-lost-update-3.png){style="background: #fff; padding: 32px;border-radius: 32px"}

<slide :class="size-80">

## 总结

---

条件式请求是 HTTP 协议中一项非常重要的特性，它使高效复杂的应用系统的构建得以实现。

HTTP 的部分功能本身就依赖条件式请求这项特性，比如缓存或断点续传功能。在某些环境中设置正确的实体标签可能需要一些技巧，不过对站点管理员来说只需要正确配置服务器就可以了。但一旦设置成功，浏览器就能够按照预期地发送条件式请求。

同时开发者也能利用条件式请求完成自己独特的业务逻辑，比如控制并发上传和并发修改的锁机制。

显然在两种情况下，条件式请求都发挥着基础性作用，成为 Web 应用的有力支撑。
