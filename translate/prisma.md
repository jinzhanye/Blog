Prisma 取代了传统的 ORM 并简化了数据库工作流：

- 访问：使用自动生成的 Prisma 客户端（in JavaScript, TypeScript, Go）访问类型安全的数据库
- 迁移：声名式的数据建模和迁移（可选）
- 管理：使用 Prisma 管理端进行可视化数据管理

它用来构建 GraphQL、REST、gRPC 和更多种类的API。目前 Prisma 支持MySQL, PostgreSQL, MongoDB.

## 内容
- 快速开始
- 例子
- 数据库连接
- 社区
- 贡献
- Prisma 2 预览


## 快速开始
从零开始 Prisma（或使用你现有的数据库）：

### 1.使用 Homebrew 安装 Prisma

```shell
brew tap prisma/prisma
brew install prisma
```

### 2.连接 Prisma 到数据库
要设置 Prisma，需要先安装 docker。运行以下命令开始使用 Prisma：

```
prisma init hello-world
```

交互式的 CLI 向导现在可以帮助你完成必要的设置
- 选择**创建新的数据库**（你也可以使用[已有的数据库](https://www.prisma.io/docs/-t003/)或部署好的[数据库 demo](https://www.prisma.io/docs/?redirect=%2Fdocs%2F-t001%2F)）
- 选择数据库类型：**MySQL** 或 **PostgreSQL**
- 为生成的 Prisma 客户端选择语言：Typescript、Flow、Javascript 或 Go

向导终止后，运行以下命令设置 Prisma：
```shell
cd hello-world
docker-compose up -d
```

## 3.定义你的数为模型
使用 [SDL](https://www.prisma.io/blog/graphql-sdl-schema-definition-language-6755bcb9ce51/) 编辑 `dtatmodle.prisma` 定义你的数据模型。每个模型映射一个数据表

```
type User {
  id: ID! @id
  email: String @unique
  name: String!
  posts: [Post!]!
}

type Post {
  id: ID! @id
  title: String!
  published: Boolean! @default(value: false)
  author: User
}
```

## 部署部署模型 & 迁移数据库
运行以下命令部署你的 Prisma API
```
prisma deploy
```
Prisma API 基于数据模型为部署的，并公开该文件的每个模型的 CRUD 和实时操作

## 使用 Prisma 客户端（Node.js）
Prisma 客户端连接 Prisma API，允许你对数据库进行执行读写操作。
本节介绍如何使用 Node.js 中的 Prisma 客户端。 

在 hello-world 目录中安装 `prisma-client-lib` 依赖
```
npm install --save prisma-client-lib
```

运行以下命令生成 Prisma 客户端
```
prisma generate
```

在 hello-world 目录内创建 Node 脚本
```
touch index.js
```

在脚本里添加以下代码
```js
const { prisma } = require('./generated/prisma-client')

// A `main` function so that we can use async/await
async function main() {
  // Create a new user with a new post
  const newUser = await prisma.createUser({
    name: 'Alice',
    posts: {
      create: { title: 'The data layer for modern apps' }
    }
  })
  console.log(`Created new user: ${newUser.name} (ID: ${newUser.id})`)

  // Read all users from the database and print them to the console
  const allUsers = await prisma.users()
  console.log(allUsers)

  // Read all posts from the database and print them to the console
  const allPosts = await prisma.posts()
  console.log(allPosts)
}

main().catch(e => console.error(e))
```

最后，使用以下命令来执行脚本代码
```
node index.js
```

## 生词
wizard 向导
exposes 公开/暴露
