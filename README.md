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

## 中文说明

零依赖的 HTTP `Content-Type` 头解析与序列化库，遵循 RFC 9110。

### `parse(header, options?)`

解析 `Content-Type`（或 `Accept`）头，返回 `{ type, parameters, index }`：

- `type`：media type，解析时统一折叠为小写（如 `IMAGE/SVG+XML` → `image/svg+xml`）。
- `parameters`：参数表（null 原型对象）。参数名折叠为小写（`Charset=UTF-8` 的 key 为 `charset`），参数值保留原始大小写。
- `index`：本次解析停止处的下标。

解析规则要点：

- 重复参数采用 **first-wins**：先出现的参数生效，后续同名（含大小写折叠后同名）参数忽略，quoted/unquoted 混用同样适用。
- quoted-string 中的反斜杠按 quoted-pair 反转义（去掉反斜杠本身，保留下一个字符）；未闭合的引号会忽略该参数。
- 闭合引号后到分隔符之间的非 OWS 垃圾字符被丢弃，后续参数继续解析。
- 解析器是宽容的：畸形输入不抛错；type 两侧、参数名与 `=` 两侧的 OWS（空格/制表符）按规范吞掉。

`ParseOptions`：

- `parameters`（默认 `true`）：设为 `false` 时在首个 `;` 处早退，只返回 `type`，`index` 停在分号处，`parameters` 为空。
- `comma`（默认 `false`）：设为 `true` 时在逗号处早退，可用于逐段解析 `Accept` 这类多值头（引号内的逗号不会触发早退）。
- `start`（默认 `0`）：解析起始下标。

### `format(obj)`

将 `{ type, parameters? }` 序列化为头字符串：

- `type` 与参数名的大小写原样保留（不做折叠）。
- 参数值若不是纯 token（含空格、`"`、`\` 等）会自动加引号；其中 `"` 与 `\` 会以反斜杠转义，空值输出 `""`。
- 非法的 `type`、参数名或参数值抛出带诊断信息的 `TypeError`（如 `Invalid type: ...`、`Invalid parameter name: ...`、`Invalid parameter value: ...`）。

### `isTypeValid(str, start?, end?)` / `isTokenValid(str, start?, end?)`

按 RFC 9110 校验 type（必须恰好含一个 `/` 且两侧均为非空 token）与 token（参数名）字符集；`format` 内部即使用这两个函数。可选的 `start`/`end` 用于只校验字符串的一个切片。

合法输入上 `format(parse(x))` 可安全 round-trip（type 与参数名会等价地归一为小写形式）。

### 测试

`npm test`（ts-scripts：prettier、`tsc` 类型检查、vitest），最近一次真实运行结果：

```
Test Files  3 passed (3)
Tests       131 passed (131)
```

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
