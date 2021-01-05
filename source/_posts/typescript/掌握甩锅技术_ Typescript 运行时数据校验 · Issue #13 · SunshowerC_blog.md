---
title: 掌握甩锅技术: Typescript 运行时数据校验 · Issue #13 · SunshowerC/blog
categories: ['typescript']
tags: ['typescript', '工程化']
---


[![](https://user-images.githubusercontent.com/13402013/56365999-b36b2900-6224-11e9-8444-a8e1f30900bd.png#align=left&display=inline&height=582&margin=%5Bobject%20Object%5D&originHeight=582&originWidth=658&status=done&style=none&width=658)<br />](https://user-images.githubusercontent.com/13402013/56365999-b36b2900-6224-11e9-8444-a8e1f30900bd.png)<br />

- [背景](#背景)
- [为什么要运行时校验数据？](#为什么要运行时校验数据)
- [io-ts 解决方案？](#io-ts-解决方案)
- [理想方案探索](#理想方案探索)
  - [JSON schema](#json-schema)
  - [typescript -> json-schema](#typescript---json-schema)
  - [json-schema 校验库](#json-schema-校验库)
  - [commit 时自动更新 json-schema](#commit-时自动更新-json-schema)
- [总结](#总结)



<a name="8e1b944f"></a>
# 背景

<br />大家出来写  代码的，难免会出 Bug。<br />
<br />文章背景就发生在一个 Bug 身上，<br />
<br />有一天，测试慌张中带着点兴奋冲过来：<br />测试："xxx 系统前端线上出 Bug 了，点进 xx 页面一片空白啊"。<br />我："纳尼？我写的 Bug 怎么会出现代码呢？"。<br />[![](https://user-images.githubusercontent.com/13402013/56365430-6d619580-6223-11e9-82b2-2a91a1abddb0.png#align=left&display=inline&height=337&margin=%5Bobject%20Object%5D&originHeight=337&originWidth=415&status=done&style=none&width=415)<br />](https://user-images.githubusercontent.com/13402013/56365430-6d619580-6223-11e9-82b2-2a91a1abddb0.png)<br />
<br />虽然大脑一片空白，但是锅还是要背的。<br />进入页面一看，哦豁，完蛋，`cannot read the property 'xx' of undefined`。确实是前端常见的报错呀。<br />
<br />**背锅王，我当定了？未必。**<br />
<br />我眉头一皱，发现事情并不是那么简单，经过一番猛如虎的操作之后，最终定位到问题是：后端接口响应的 JSON 数据中，一个嵌套比较深的字段没有返回，即前端只读到了 `undefined`。<br />
<br />咱按章程办事，**后端提供的接口文档指定了数据结构，那你没有返回正确数据结构，这就是你后端的锅**，虽然严谨点前端也能捕获到错误进行处理，但归根到底，是你后端数据接口处理有问题，**这锅，我不背。**<br />
<br />甩锅又是一门扯皮的事情，杀敌一千自伤八百，锅已经扣下来了，想甩出去就难咯，。<br />
<br />唉，要是在接口出错的时候，能立刻知道接口数据出问题，先发制人，马上把锅甩出去那就好咯。<br />
<br />这就是本文即将要讲述的 "Typescript 运行时数据校验"。<br />

<a name="871b2e68"></a>
# 为什么要运行时校验数据？

<br />众所周知，`Typescript` 是 `JavaScript` 超集，可以给我们的项目代码提供静态类型检查，避免因为各种原因而未及时发现的代码错误，在**编译时**就能发现隐藏的代码隐患，从而提高代码质量。<br />
<br />但是，`TypeScript` 项目的一个常见问题是: 如何验证来自外部源的数据并将验证的数据与 TypeScript 类型联系起来。 即，如何避免**后端 API 返回的数据与 `Typescript` 类型定义不一致导致的运行时错误。**<br />
<br />`Typescript` 能用于运行时校验数据类型，那么有没有一种方法，能让我们在 **运行时** 也进行 `Typescript` 数据类型校验呢？<br />

<a name="cbb2e88e"></a>
# io-ts 解决方案？

<br />业界开源了一个运行时校验的工具库：[io-ts](https://github.com/gcanti/io-ts)。<br />

```typescript
//  io-ts 例子
import * as t from 'io-ts'

// ts 定义
interface Category {
  name: string
  categories: Array<Category>
}

// 对应上述ts定义的 io-ts 实现
const Category: t.Type<Category> = t.recursion('Category', () =>
  t.type({
    name: t.string,
    categories: t.array(Category)
  })
)
```

<br />**但是**，如上面的代码所示，这工具看起来就**有点啰嗦有点难用**，对代码的**侵入性非常强**，要全盘依据它的语法来重写代码。这对于一个团队来说，存在一定的迁移成本。<br />
<br />而我们更希望做到的理想方案是：<br />
<br />**写好接口的数据结构 `typescript` 定义，不需要做太多的额外变动，直接就能校验后端接口响应的数据结构是否符合 `typescript` 接口定义**<br />

<a name="17d1af5d"></a>
# 理想方案探索

<br />首先，我们了解到，后端响应的数据接口一般为 `JSON`，那么，抛开 `Typescript`，如果要校验一个 JSON 的数据结构，我们可以怎么做到呢？<br />
<br />**答案是[`JSON schema`](http://json-schema.org)。**<br />

<a name="16f89caa"></a>
## JSON schema

<br />JSON schema 是一种描述 JSON 数据格式的模式。<br />
<br />例如 typescript 数据结构：<br />

```typescript
type TypeSex = 1 | 2 | 3
interface UserInfo {
  name: string
  age?: number
  sex: TypeSex
}
```

<br />等价于以下的 json schema ：<br />

```json
{
  "$id": "api",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "definitions": {
    "UserInfo": {
      "properties": {
        "age": {
          "type": "number"
        },
        "name": {
          "type": "string"
        },
        "sex": {
          "enum": [
            1,
            2,
            3
          ],
          "type": "number"
        }
      },
      "required": [
        "name",
        "sex"
      ],
      "type": "object"
    }
  }
}
```

<br />根据已有 `json-schema` 校验库，即可校验数据对象<br />

```javascript
someValidateFunc(jsonSchema, apiResData)
```

<br />这里大家可能就又会困惑：这`json-schema`写起来也太费劲了？还不一样要学习成本，那和 `io-ts` 有什么区别。<br />
<br />但是，既然我们同时知道 `typescript` 和 `json-schema` 的语法定义规则，那么就两者必然能够互相转换。<br />
<br />也就是说，即便我们不懂 `json-schema` 的规范与语法，我们也能通过`typescript` 转化生成 `json-schema`。<br />
<br />那么，在以上的前提下，我们的思路就是：**既然 `typescript` 本身不支持运行时数据校验，那么我们可以将 `typescript` 先转化成 `json schema`, 然后用 `json-schema` 校验数据结构**<br />

<a name="63a9c47a"></a>
## typescript -> json-schema

<br />要将 `typescript` 声明转换成 `json-schema` ，这里推荐使用 [typescript-json-schema](https://github.com/YousefED/typescript-json-schema)。<br />
<br />我们可以直接使用它的命令行工具，这里就不仔细展开说明了，感兴趣的可以看下官方文档：<br />

```
Usage: typescript-json-schema <path-to-typescript-files-or-tsconfig> <type>

Options:
  --refs                Create shared ref definitions.                               [boolean] [default: true]
  --aliasRefs           Create shared ref definitions for the type aliases.          [boolean] [default: false]
  --topRef              Create a top-level ref definition.                           [boolean] [default: false]
  --titles              Creates titles in the output schema.                         [boolean] [default: false]
  --defaultProps        Create default properties definitions.                       [boolean] [default: false]
  --noExtraProps        Disable additional properties in objects by default.         [boolean] [default: false]
  --propOrder           Create property order definitions.                           [boolean] [default: false]
  --required            Create required array for non-optional properties.           [boolean] [default: false]
  --strictNullChecks    Make values non-nullable by default.                         [boolean] [default: false]
  --useTypeOfKeyword    Use `typeOf` keyword (https://goo.gl/DC6sni) for functions.  [boolean] [default: false]
  --out, -o             The output file, defaults to using stdout
  --validationKeywords  Provide additional validation keywords to include            [array]   [default: []]
  --include             Further limit tsconfig to include only matching files        [array]   [default: []]
  --ignoreErrors        Generate even if the program has errors.                     [boolean] [default: false]
  --excludePrivate      Exclude private members from the schema                      [boolean] [default: false]
  --uniqueNames         Use unique names for type symbols.                           [boolean] [default: false]
  --rejectDateType      Rejects Date fields in type definitions.                     [boolean] [default: false]
  --id                  Set schema id.                                               [string] [default: ""]
```

<br />github 上也有所有类型转换的 [测试用例](https://github.com/YousefED/typescript-json-schema/tree/master/test/programs)，可以对比看看 `typescript` 和 转换出的 `json-schema` 结果<br />

<a name="4abf6b15"></a>
## json-schema 校验库

<br />利用 `typescript-json-schema` 工具生成了 `json-schema` 文件后，我们需要根据该文件进行数据校验。<br />
<br />`json-schema` 数据校验的库很多，[ajv](https://github.com/epoberezkin/ajv)，[jsonschema](https://github.com/tdegrunt/jsonschema) 之类的，这里用 `jsonschema` 作为示例。<br />

```typescript
import { Validator } from 'jsonschema'

import schema from './json-schema.json'

const v = new Validator()
// 绑定schema，这里的 `api` 对应 json-schema.json 的 `$id`
v.addSchema(schema, '/api')


const validateResponseData = (data: any) => {
  // 校验响应数据
  const result = v.validate(data, {
    // SomeInterface 为 ts 定义的接口
    $ref: `api#/definitions/SomeInterface`
  })

  // 校验失败，数据不符合预期
  if (!result.valid) {
    console.log('data is ', data)
    console.log('errors', result.errors.map((item) => item.toString()))
  }

  return data
}
```

<br />当我们校验以下数据时：<br />

```typescript
// 声明文件
interface UserInfo {
  name: string
  sex: string
  age: number
  phone?: number
}

// 校验结果
validateResponseData({
  name: 'xxxx',
  age: 'age应该是数字'
})
// 得出结果
// data is  { name: 'xxxx', age: 'age应该是数字' }
// errors [ 'instance.age is not of a type(s) number',
//   'instance requires property "sex"' ]
```

<br />[完全例子请看 github](https://github.com/Weiyu-Chen/ts-runtime-json-schema)<br />
<br />配合上前端上报系统，当线上系统接口返回了非预料的数据，导致出 bug，就可以实时知道到底错在哪了，并且及时甩锅给后端啦。<br />

<a name="09904c98"></a>
## commit 时自动更新 json-schema

<br />前面提到，我们需要执行 `typescript-json-schema <path-to-typescript-files-or-tsconfig> <type>` 命令来声明 typescript 对应的 `json-schema` 文件。<br />
<br />那么，这里就有个问题，接口数量有可能增加，接口数据也有可能变动，那也就代表着，我们每次变更接口数据结构，都要重新跑一下 `typescript-json-schema` ，时刻保持 `json-schema` 和 typescript 一一对应。<br />
<br />这我们就可以用 [husky](https://github.com/typicode/husky) 的 `precommit` ， 加上 [lint-staged](https://github.com/okonet/lint-staged) 来实现每次更新提交代码时，自动执行 `typescript-json-schema`，无需时刻关注 typescript 接口定义的变更。<br />
<br />[完全例子请看 github](https://github.com/Weiyu-Chen/ts-runtime-json-schema)<br />

<a name="25f9c7fa"></a>
# 总结

<br />综上，我们实现了<br />

1. `typescript` 声明文件 转换生成 `json-schema` 文件
2. 代码接口层拦截校验数据，如校验失败，通过前端上报系统 (如：[sentry](https://sentry.io/)) 进行相关上报
3. 通过 `husky` + `lint-staged` 每次提交代码自动执行 步骤 1，保持 git 仓库的代码 `typescript` 声明 和 `json-schema` 时刻保持一致。


<br />那么，当 Bug 出现的时候，你甚至可以在测试都还没发现这个 Bug 之前，就已经把锅甩了出去。<br />
<br />**只要你跑得足够快，Bug 就会追不上你。**<br />
<br />[![](https://user-images.githubusercontent.com/13402013/56365999-b36b2900-6224-11e9-8444-a8e1f30900bd.png#align=left&display=inline&height=582&margin=%5Bobject%20Object%5D&originHeight=582&originWidth=658&status=done&style=none&width=658)<br />](https://user-images.githubusercontent.com/13402013/56365999-b36b2900-6224-11e9-8444-a8e1f30900bd.png)<br />[https://github.com/SunshowerC/blog/issues/13](https://github.com/SunshowerC/blog/issues/13)
