---
title: corepack设置npmmirror出现无法302的提示
date: 2024-01-29 15:31:14
keyword:
- core设置COREPACK_NPM_REGISTRY后出现302
tags:
- corpack无法正常使用
categories:
  - [Nodejs, Corepack]
  - [Nodejs, pnpm]
  - [Nodejs, yarn]
---

在上篇对`corepack`的`npmmirror`修改后，corepack的安装能够正常的进行了。

在其他的mirror可以正常的情况下，尝试了`https://registry.npmmirror.com`,发现这个mirror内容会有302跳转，但corepack当前设置禁止302，导致这个mirror无法使用。

本文就是我对探究无法解决这个问题的分享。

<!--more-->

## 分析
接上篇对`corepack`配置`npmmirror`为`https://registry.npmmirror.com`后,对pnpm进行设备更新时`corepack use pnpm@latest` 出现如下错误
```powershell
Internal Error: Server answered with HTTP 302 when performing the request to https://registry.npmmirror.com/pnpm/-/pnpm-8.15.0.tgz; for troubleshooting help, see https://github.com/nodejs/corepack#troubleshooting
    at ClientRequest.<anonymous> (C:\Users\xxxx\AppData\Roaming\nvm\v20.11.0\node_modules\corepack\dist\lib\corepack.cjs:42192:21)
    at Object.onceWrapper (node:events:633:26)
    at ClientRequest.emit (node:events:518:28)
    at HTTPParser.parserOnIncomingClient (node:_http_client:693:27)
    at HTTPParser.parserOnHeadersComplete (node:_http_common:119:17)
    at TLSSocket.socketOnData (node:_http_client:535:22)
    at TLSSocket.emit (node:events:518:28)
    at addChunk (node:internal/streams/readable:559:12)
    at readableAddChunkPushByteMode (node:internal/streams/readable:510:3)
    at Readable.push (node:internal/streams/readable:390:5)
```

直接在浏览器点开这个链接[https://registry.npmmirror.com/pnpm/-/pnpm-8.15.0.tgz](https://registry.npmmirror.com/pnpm/-/pnpm-8.15.0.tgz),会发现云端返回302后跳转到了cdn链接[https://cdn.npmmirror.com/packages/pnpm/8.15.0/pnpm-8.15.0.tgz]

但`corepack`内部将大于300的code全部定义为错误
```javascript
async function fetchUrlStream(url, options = {}) {
  if (process.env.COREPACK_ENABLE_NETWORK === `0`)
    throw new UsageError(`Network access disabled by the environment; can't reach ${url}`);
  const { default: https } = await import("https");
  const { ProxyAgent } = await Promise.resolve().then(() => __toESM(require_dist12()));
  const proxyAgent = new ProxyAgent();
  return new Promise((resolve, reject) => {
    const request = https.get(url, { ...options, agent: proxyAgent }, (response) => {
      const statusCode = response.statusCode;
      if (statusCode != null && statusCode >= 200 && statusCode < 300)
        return resolve(response);
      return reject(new Error(`Server answered with HTTP ${statusCode} when performing the request to ${url}; for troubleshooting help, see https://github.com/nodejs/corepack#troubleshooting`));
    });
    request.on(`error`, (err) => {
      reject(new Error(`Error when performing the request to ${url}; for troubleshooting help, see https://github.com/nodejs/corepack#troubleshooting`));
    });
  });
}
```

还是老样子去github上看看是否已有类似的issue和PR，经过一番搜索，找到如下的讨论

[add http redirect support #306](https://github.com/nodejs/corepack/pull/306)

发现在#302已经解决，找到302
[install from npmmirror.com failed (302 redirect) #302](https://github.com/nodejs/corepack/issues/302)

在23年12月29日，有人已提交PR修复[feat: Support redirections #341](https://github.com/nodejs/corepack/pull/341)

我们来看看修复类容(PR 文件变更)(https://github.com/nodejs/corepack/pull/341/files)

变更也很简单，当`code`符合定义时，讲取出`header.location`进行第二次访问
```javascript
        if ([301, 302, 307, 308].includes(statusCode as number) && response.headers.location)
          return createRequest(response.headers.location as string);
```

## 结论
那我们要做的就很简单，升级到`v0.24.0`及以上。简单的升级办法是下载`release`里面版本符合的包，替换整个`C:\Users\xxxx\AppData\Roaming\nvm\v20.11.0\node_modules\corepack`目录即可
