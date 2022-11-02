# 消息段 (Segment)

**消息段** 的概念类似于 HTML 中的标签，用于描述具有特定语义的内容。Koishi 提供了两种方式处理消息：字符串和消息段数组。它们之间可以相互转换。

```ts
// 会话属性
session.content                             // 字符串
session.elements                            // 消息段数组

// 发送消息
session.send('<at id="123"/>')              // 发送字符串
session.send(segment('at', { id: 123 }))    // 发送消息段
```

我们为常见的消息段提供了静态方法，以方便使用：

```ts
session.send(segment.image(url))            // 发送图片
```

对于复杂的消息段，我们也提供了 `h` 的别名。下面的写法是等价的：

```ts
session.send(h('image', { url }))           // 发送图片
```

## 实例属性

一个消息段对象的结构如下：

```ts
interface segment {
  type: string
  attrs: object
  children: segment[]
}
```

## 静态方法

### segment(type, attrs?, children?)

- **type:** `string` 消息段类型
- **attrs:** `object` 消息段属性
- **children:** `segment[]` 子消息段
- 返回值: `segment` 生成的消息段

构造一个消息段对象。

### segment.escape(source, inline?)

- **source:** `string` 源文本
- **inline:** `boolean` 在属性内部转义 (会额外处理引号)
- 返回值: `string` 转义过后的文本

转义一段文本到消息段格式。

### segment.unescape(source)

- **source:** `string` 源文本
- 返回值: `string` 转义前的文本

取消一段文本的消息段转义。

### segment.parse(source)

- **source:** `string` 源文本
- 返回值: `segment[]` 消息段数组

解析一段文本内的全部消息段。其中的纯文本将会解析成 text 类型。

### segment.select(source, query)

- **source:** `string | segment[]` 源文本或消息段数组
- **query:** `string` 选择器
- 返回值: `segment[]` 消息段数组

使用选择器在一段文本或消息段数组中查找。支持的语法包括：

- 通配选择器 `*`
- 元素选择器 `type`
- 选择器列表 `sel1, sel2`
- 组合器
  - 后代组合器 `sel1 sel2`
  - 直接子代组合器 `sel1 > sel2`
  - 一般兄弟组合器 `sel1 ~ sel2`
  - 紧邻兄弟组合器 `sel1 + sel2`

如果传入了字符串，则会先使用 [`segment.parse()`](#segment-parse-source) 进行解析。

### segment.transform(source, rules)

- **source:** `string | segment[]` 源文本或消息段数组
- **rules:** `Dict<Transformer>` 转换规则，以消息段类型为键
- 返回值: `string | segment[]` 转换后的文本或消息段数组

将一段文本或消息段数组中的所有消息段按照规则进行转换。转换规则的定义方式如下：

```ts
type Content = string | segment | (string | segment)[]
type Transformer =
  | boolean
  | Content
  | ((attrs: Dict, index: number, array: segment[]) => Content)
```

返回值会与传入的参数保持相同类型。如果传入了字符串，则会先使用 [`segment.parse()`](#segment-parse-source) 进行解析，并在转换完成后重新序列化。

### segment.transformAsync(source, rules)

- **source:** `string | segment[]` 源文本或消息段数组
- **rules:** `Dict<AsyncTransformer>` 转换规则，以消息段类型为键
- 返回值: `Promise<string | segment[]>` 转换后的文本或消息段数组

与 [`segment.transform()`](#segment-transform-source-rules) 类似，但转换规则可以是异步的，同一层级的各元素的转换将同时进行。转换规则的定义方式如下：

```ts
type AsyncTransformer =
  | boolean
  | Content
  | ((attrs: Dict, index: number, chain: segment[]) => Awaitable<Content>)
```