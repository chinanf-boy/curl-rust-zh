# alexcrichton/curl-rust [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 libcurl 的 Rust 绑定库 」

[中文](./readme.md) | [english](https://github.com/alexcrichton/curl-rust)

---

## 校对 ✅

<!-- doc-templite START generated -->
<!-- repo = 'alexcrichton/curl-rust' -->
<!-- commit = '3a309647f1ac35ecd749236e14a16479b804bf16' -->
<!-- time = '2018-11-07' -->
翻译的原文 | 与日期 | 最新更新 | 更多
---|---|---|---
[commit] | ⏰ 2018-11-07 | ![last] | [中文翻译][translate-list]

[last]: https://img.shields.io/github/last-commit/alexcrichton/curl-rust.svg
[commit]: https://github.com/alexcrichton/curl-rust/tree/3a309647f1ac35ecd749236e14a16479b804bf16

<!-- doc-templite END generated -->

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[If help, **buy** me coffee —— 营养跟不上了，给我来瓶营养快线吧! 💰](https://github.com/chinanf-boy/live-need-money)

---

# curl-rust

Rust 绑定 libcurl

[![Build Status](https://travis-ci.org/alexcrichton/curl-rust.svg?branch=master)](https://travis-ci.org/alexcrichton/curl-rust)
[![Build status](https://ci.appveyor.com/api/projects/status/lx98wtbxhhhajpr9?svg=true)](https://ci.appveyor.com/project/alexcrichton/curl-rust)

[docs.rs/文档](https://docs.rs/curl)

### 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [快速入门](#%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8)
- [Post / Put 请求](#post--put-%E8%AF%B7%E6%B1%82)
- [自定义 headers](#%E8%87%AA%E5%AE%9A%E4%B9%89-headers)
- [Keep alive](#keep-alive)
- [多 requests](#%E5%A4%9A-requests)
- [绑定的库](#%E7%BB%91%E5%AE%9A%E7%9A%84%E5%BA%93)
- [版本 支持](#%E7%89%88%E6%9C%AC-%E6%94%AF%E6%8C%81)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 快速入门

```rust
extern crate curl;

use std::io::{stdout, Write};

use curl::easy::Easy;

// 打印一个 web页面 到 stdout
fn main() {
    let mut easy = Easy::new();
    easy.url("https://www.rust-lang.org/").unwrap();
    easy.write_function(|data| {
        stdout().write_all(data).unwrap();
        Ok(data.len())
    }).unwrap();
    easy.perform().unwrap();

    println!("{}", easy.response_code().unwrap());
}
```

```rust
extern crate curl;

use curl::easy::Easy;

// 捕获 output(web页面) 到 一个本地变量 `Vec`.
fn main() {
    let mut dst = Vec::new();
    let mut easy = Easy::new();
    easy.url("https://www.rust-lang.org/").unwrap();

    let mut transfer = easy.transfer();
    transfer.write_function(|data| {
        dst.extend_from_slice(data);
        Ok(data.len())
    }).unwrap();
    transfer.perform().unwrap();
}
```

## Post / Put 请求

`Easy`的这个`put`和`post`方法，可以配置 HTTP 请求,然后`read_function`可以用来填充指定的数据。该接口与实现`Read`的类型匹配。

```rust
extern crate curl;

use std::io::Read;
use curl::easy::Easy;

fn main() {
    let mut data = "this is the body".as_bytes();

    let mut easy = Easy::new();
    easy.url("http://www.example.com/upload").unwrap();
    easy.post(true).unwrap();
    easy.post_field_size(data.len() as u64).unwrap();

    let mut transfer = easy.transfer();
    transfer.read_function(|buf| {
        Ok(data.read(buf).unwrap_or(0))
    }).unwrap();
    transfer.perform().unwrap();
}
```

## 自定义 headers

自定义的 headers，可以指定为请求的一部分:

```rust
extern crate curl;

use curl::easy::{Easy, List};

fn main() {
    let mut easy = Easy::new();
    easy.url("http://www.example.com").unwrap();

    let mut list = List::new();
    list.append("Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==").unwrap();
    easy.http_headers(list).unwrap();
    easy.perform().unwrap();
}
```

## Keep alive

`handle`（下面的变量）可以跨多个请求重复使用。Curl 将试图保持连接活着.

```rust
extern crate curl;

use curl::easy::Easy;

fn main() {
    let mut handle = Easy::new();

    handle.url("http://www.example.com/foo").unwrap();
    handle.perform().unwrap();

    handle.url("http://www.example.com/bar").unwrap();
    handle.perform().unwrap();
}
```

## 多 requests

libcurl 库通过"multi"接口，支持同时发送多个请求。位于当前`curl-rust`下的[`multi`模块][multi]中。其提供同时执行多个传输的能力。有关更多信息,请参见[`multi`模块][multi].

[multi]: https://github.com/alexcrichton/curl-rust/blob/master/src/multi.rs

## 绑定的库

默认情况下,此箱子(crate)将尝试动态链接到，在系统范围内的 libcurl 和 SSL 库。一些行为可通过各种 Cargo 功能定制:

| 名            | 曰                                                                                  |
| ------------- | ----------------------------------------------------------------------------------- |
| `ssl`         | 启用 SSL 支持。默认启用.                                                            |
| `http2`       | 通过 libnghttp2 启用 HTTP/2 支持。默认情况下禁用.                                   |
| `static-curl` | 使用捆绑的 libcurl 版本并静态链接到它。默认情况下禁用.                              |
| `static-ssl`  | 使用捆绑的 OpenSSL 版本并静态链接到它。只适用于使用 OpenSSL 的平台。默认情况下禁用. |
| `spengo`      | 启用 SPEGNO 支持。默认情况下禁用.                                                   |

## 版本 支持

此绑定库已经使用 Curl 版本 7.24.0 开发。它们应该与任何新版本的 curl 一起工作,并且可能与旧版本一起工作,但是这还没有经过测试.

## License

这个`curl-rust`,MIT 许可证,见`LICENSE`更多细节.
