# openapi-generator-ts-cli

[![Version](https://img.shields.io/badge/version-v0.1.0-blue.svg)](./package.json)
[![Node.js](https://img.shields.io/badge/Node.js-%3E%3D20.19.0-339933.svg?logo=node.js)](https://nodejs.org/)
[![License](https://img.shields.io/badge/license-Apache--2.0-green.svg)](./package.json)

> 当前版本：v0.1.0

一个轻量级纯 TypeScript OpenAPI/Swagger 客户端生成器

它直接读取 OpenAPI/Swagger 文档并生成 TypeScript API client，不依赖 Java 版 `openapi-generator-cli`，也不绑定任何具体业务项目Swagger/OpenAPI 文档是唯一输入来源；目录、模型、枚举、API 方法和注释都从文档结构中推导

## 特性

- 纯 TypeScript CLI，构建和运行都只需要 Node.js
- 支持单个 OpenAPI 文件、远程 URL 或本地 glob 批量 Swagger 文件
- API 按 Swagger tag 聚合到 `generate/apis`
- Model 按 Swagger 来源文件和 schema namespace 分组
- `enums*` schema 会输出到顶层 `generate/enums`
- `docs` 默认不生成，只有显式传 `--docs` 才输出
- 保留 Swagger/OpenAPI 的 `summary`、`description`、`title` 为 TypeScript JSDoc
- 生成代码支持无参 `new XxxApi()`，也支持传入 Axios-like instance

## 安装与构建

### 从 Git 安装固定分发版本

`main` 保存 TypeScript 源码。以下 Git 安装链接固定到本仓库的预构建分发提交，其 `package.json` 版本为 `0.1.0`。

```bash
npm install -g git+https://github.com/HitGameAI/openapi-generator-ts-cli.git#e0c2d381f14eb373cafda7b795a13cb779d12dc5
```

该提交包含 `dist/cli.js`。安装后可以检查命令版本和帮助：

```bash
openapi-generator-ts-cli --version
openapi-generator-ts-cli --help
```

安装后可以直接使用 `package.json` 中声明的命令：

```bash
openapi-generator-ts-cli generate \
  --swagger-glob "./openapi/**/*.swagger.yaml" \
  --output "./generate" \
  --clean
```

作为开发依赖时，使用同一个分发提交：

```bash
npm install -D git+https://github.com/HitGameAI/openapi-generator-ts-cli.git#e0c2d381f14eb373cafda7b795a13cb779d12dc5
```

需要重新安装时，可以先卸载，再安装同一个分发提交：

```bash
npm uninstall -g openapi-generator-ts-cli
npm install -g --force git+https://github.com/HitGameAI/openapi-generator-ts-cli.git#e0c2d381f14eb373cafda7b795a13cb779d12dc5
```

然后在项目 `package.json` 中配置：

```json
{
  "scripts": {
    "generate:api": "openapi-generator-ts-cli generate --swagger-glob \"./openapi/**/*.swagger.yaml\" --output \"./generate\" --clean"
  }
}
```

### 本地开发安装

```bash
yarn install
yarn build
```

也可以使用 npm：

```bash
npm install
npm run build
```

## 本地开发基本用法

批量读取本地 Swagger 文件：

```bash
node dist/cli.js generate \
  --swagger-glob "../openapi/**/*.swagger.yaml" \
  --output "../generate" \
  --clean
```

读取单个本地文件：

```bash
node dist/cli.js generate \
  --input "../openapi/service.swagger.yaml" \
  --output "../generate" \
  --clean
```

读取远程 OpenAPI URL：

```bash
node dist/cli.js generate \
  --input "https://example.com/openapi.json" \
  --output "./generate"
```

读取需要鉴权的远程 Swagger：

```bash
node dist/cli.js generate \
  --input "https://example.com/openapi.json" \
  --bearer-token "your-token" \
  --output "./generate"
```

也可以直接传任意请求头，适用于 API Key、Cookie 或自定义网关鉴权：

```bash
node dist/cli.js generate \
  --input "https://example.com/openapi.json" \
  --header "X-API-Key: your-api-key" \
  --header "Cookie: session=your-session" \
  --output "./generate"
```

在当前 xxx 工作区中常用命令：

```bash
node dist/cli.js generate \
  --swagger-glob "../xxx-share-proto/proto/**/*.swagger.yaml" \
  --output "../generate" \
  --clean
```

## CLI 参数

- `--input <file-or-url>`：读取一个本地 OpenAPI/Swagger 文件或远程 URL
- `--swagger-glob <glob>`：读取多个本地 OpenAPI/Swagger 文件
- `--output <dir>`：输出目录，默认是当前目录下的 `generate`
- `--header <header>`：远程 OpenAPI URL 的请求头，可重复传入，例如 `--header "Authorization: Bearer token"`
- `--bearer-token <token>`：远程 OpenAPI URL 的 Bearer Token 快捷参数，会生成 `Authorization: Bearer <token>`
- `--docs`：生成模块级 Markdown 文档；默认关闭
- `--clean`：写入前清理旧的生成目录，避免残留过期文件

`--input` 和 `--swagger-glob` 可以同时使用，生成器会把读取到的文档合并到同一套输出中
`--header` 和 `--bearer-token` 只影响远程 `http/https` 输入，本地文件和 glob 不会使用这些请求头

## 输出结构

```text
generate/
  apis/
    <tag-api>.ts
    index.ts
  models/
    <source-dir>/
      <source-file>.ts
      index.ts
    <shared-schema-namespace>/
      common.ts
      index.ts
    index.ts
  enums/
    common.ts
    index.ts
  base.ts
  common.ts
  configuration.ts
  api.ts
  index.ts
```

根入口 `generate/index.ts` 会统一导出：

```ts
export * from './api';
export * from './models';
export * from './enums';
export * from './configuration';
```

## 分组规则

API 文件按 operation 的第一个 `tag` 聚合：

- `Authorization` -> `apis/authorization-api.ts`
- `TwoFA` -> `apis/two-faapi.ts`
- `PresignedURL` -> `apis/presigned-urlapi.ts`

Model 文件按 schema 名和 Swagger 来源文件推导：

- 普通业务 schema 进入 `models/<source-dir>/<source-file>.ts`
- `commonXxx`、`apiCommonXxx` 等共享 schema 会进入 `models/<namespace>/common.ts`
- `enumsXxx` 会进入顶层 `enums/common.ts`

生成器不会写入项目定制映射；如果其他项目使用不同 schema 前缀，也会按同一套通用 namespace 规则处理

## 注释生成

生成器会保留 Swagger/OpenAPI 中的说明：

- API 方法使用 operation 的 `summary` 和 `description`
- 方法参数使用 parameter 的 `description` 或 `title`
- Model、enum、字段使用 schema 的 `description` 或 `title`

示例：

```ts
/**
 * @summary 用户登录 | User login
 * @param body
 * @param options Override http request option.
 */
async authServiceLogin(...)
```

```ts
export interface AccessControlLoginResponse {
  /**
   * 登录结果 | [EN] Login result
   */
  payload?: CommonPayload;
}
```

## API 调用方式

生成的 API class 可以无参创建，默认使用运行环境中的 `fetch`：

```ts
import { AuthorizationApi } from './generate';

const api = new AuthorizationApi();
const result = await api.authServiceJwks();
```

也可以传入配置和 Axios-like 实例：

```ts
import axios from 'axios';
import { AuthorizationApi, Configuration } from './generate';

const api = new AuthorizationApi(
  new Configuration({
    basePath: 'https://api.example.com',
    headers: { Authorization: 'Bearer token' },
  }),
  axios,
);
```

方法最后一个参数是请求覆盖项：

```ts
await api.authServiceJwks({
  headers: { 'X-Trace-Id': 'trace-id' },
});
```

## 验证生成结果

```bash
npx tsc --noEmit \
  --target ES2022 \
  --module NodeNext \
  --moduleResolution NodeNext \
  --skipLibCheck \
  --strict false \
  ../generate/index.ts
```

## 设计约束

- 不依赖 Java openapi-generator
- 不包含业务项目专属目录、前缀或硬编码配置
- 以 Swagger/OpenAPI 为主，不读取 proto 文件
- `docs` 是可选输出，默认不生成
- 生成文件使用 UTF-8 编码
