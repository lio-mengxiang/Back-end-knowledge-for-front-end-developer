## Nodejs组成

Node.js主要由V8、Libuv和第三方库组成。

- Libuv：跨平台的异步IO库，但它提供的功能不仅仅是IO，还 包括进程、线程、信号、定时器、进程间通信，线程池等。

- 第三方库：异步DNS解析（cares）、HTTP解析器（旧版使用 http_parser，新版使用llhttp）、HTTP2解析器（nghttp2）、 解压压缩库(zlib)、加密解密库(openssl)等等。

- V8：实现JS解析和支持自定义的功能，得益于V8支持自定义拓展，才有了Node.js。

## Node.js代码架构

