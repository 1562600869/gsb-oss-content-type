> Init snapshot for GSB from upstream https://github.com/jshttp/content-type (MIT). Slim for GSB: dropped size-limit from npm test (Node20 incompat); removed index.bench.ts.

# content-type

[![NPM version][npm-image]][npm-url]
[![NPM downloads][downloads-image]][downloads-url]
[![Build status][build-image]][build-url]
[![Build coverage][coverage-image]][coverage-url]
[![License][license-image]][license-url]

Create and parse HTTP `Content-Type` header.

## Installation

```sh
npm install content-type
```

## API

```js
import * as contentType from "content-type";
```

### contentType.parse(string, options?)

```js
const obj = contentType.parse("image/svg+xml; charset=utf-8");
```

Parse a `Content-Type` header. This will return an object with the following properties (examples are shown for the string `'image/svg+xml; charset=utf-8'`):

- `type`: The media type (always lower case). Example: `'image/svg+xml'`.
- `parameters`: An object of the parameters in the media type (parameter name is always lower case). Example: `{charset: 'utf-8'}`.
- `index`: The index where parsing stopped. Example: `33`.

The parser is lenient and does not validate or throw on malformed input.

#### Options

- `parameters` (default: `true`): Set to `false` to skip parameters.
- `comma` (default: `false`): Set to `true` to stop on a comma. This can be used to parse the media range in an `Accept` header.
- `start` (default: `0`): Set index to start parsing from.

### contentType.format(obj)

```js
const str = contentType.format({
  type: "image/svg+xml",
  parameters: { charset: "utf-8" },
});
```

Format an object into a `Content-Type` header. This will return a string of the content type for the given object with the following properties (examples are shown that produce the string `'image/svg+xml; charset=utf-8'`):

- `type`: The media type. Example: `'image/svg+xml'`.
- `parameters`: An optional object of the parameters in the media type. Example: `{charset: 'utf-8'}`.

Throws a `TypeError` if the object contains an invalid type or parameter names.

### Validation

This package exposes the validation functions used by `format`:

- `isTypeValid(str, start?, end?)` Validates the MIME type against RFC 9110.
- `isTokenValid(str, start?, end?)` Validates a token against RFC 9110 (used for the parameter name).

Passing `start` and `end` allows for validating a subset of a string, instead of using `str#slice`.

## License

[MIT](LICENSE)

[npm-image]: https://img.shields.io/npm/v/content-type
[npm-url]: https://npmjs.org/package/content-type
[downloads-image]: https://img.shields.io/npm/dm/content-type
[downloads-url]: https://npmjs.org/package/content-type
[build-image]: https://img.shields.io/github/actions/workflow/status/jshttp/content-type/ci.yml?branch=master
[build-url]: https://github.com/jshttp/content-type/actions/workflows/ci.yml?query=branch%3Amaster
[coverage-image]: https://img.shields.io/codecov/c/gh/jshttp/content-type
[coverage-url]: https://codecov.io/gh/jshttp/content-type
[license-image]: http://img.shields.io/npm/l/content-type.svg?style=flat
[license-url]: LICENSE

## 中文说明

创建与解析 HTTP `Content-Type` 头（RFC 9110），零运行时依赖。

### API

- `contentType.parse(header, options?)`：解析 `Content-Type`（或 `Accept` 中的 media range），返回 `{ type, index, parameters }`。
  - `type`：媒体类型，解析时归一为小写（类型大小写不敏感）。
  - `parameters`：参数表；参数名归一为小写，参数值保留原始大小写；同名参数（大小写折叠后）采用 first-wins，后出现的忽略；quoted-string 中的 `\x` quoted-pair 会去掉转义反斜杠。
  - `index`：解析停止位置的下标。
  - 解析器是宽容的：非法输入不抛异常（如未闭合引号的参数会被忽略）。
- `contentType.format(obj)`：把 `{ type, parameters }` 序列化为头部字符串。非法 type / 参数名 / 参数值会抛出可诊断的 `TypeError`；含空格、`"`、`\` 的值自动加引号并转义，空值输出 `""`。
- `contentType.isTypeValid(type, start?, end?)`：按 RFC 9110 校验 `type/subtype`。
- `contentType.isTokenValid(token, start?, end?)`：按 RFC 9110 校验 token（用于参数名）。

### ParseOptions

- `parameters`（默认 `true`）：设为 `false` 时在首个 `;` 处早退，只返回 type，`index` 停在分号处。
- `comma`（默认 `false`）：设为 `true` 时在逗号处早退，用于解析 `Accept` 等多值头。
- `start`（默认 `0`）：起始解析下标，配合 `index` 可逐个解析多值头。

### 测试

`npm test`（ts-scripts：tsc 构建 + prettier + vitest + 覆盖率）。最近一次真实运行结果：**3 个测试文件、129 个测试全部通过**（`Test Files 3 passed (3)`，`Tests 129 passed (129)`）。
