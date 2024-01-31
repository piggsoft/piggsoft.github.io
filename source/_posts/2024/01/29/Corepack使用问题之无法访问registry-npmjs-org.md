---
title: Corepack使用问题之无法访问registry.npmjs.org
date: 2024-01-29 13:16:43
keyword: 
- corepack 无法访问 registry.npmjs.org
- corepack 无法使用
- corepack 无法安装pnpm
- corepack 无法安装yarn
- corepack enable后,pnpm无法使用
- corepack enable后,yarn无法使用
tags: corpack无法正常使用
categories: 
- corepack
- pnpm
- yarn 
- nodejs
---

本文主要分享我再使用`corepack`安装`pnpm`或者`yarn`的时候出现无法连接到`registry.npmjs.org`的问题。

其过程中有翻阅`yarn`和`pnpm`的官方文档，也有翻阅`corepack`本地的源代码，以及在各自的github issue，stackoverflow中查找类似问题。

最后终于找到了答案，现将答案分享出现给后续碰到相同问题的同学一起使用。

<!--more-->
## 起始
`Corepack`作为一款默认集成了`pnpm` `npx` `pnpx` `yarn` `npm` `yarnpkg`的管理工具，可在有效对`nodejs`相关的包管理工具进行管理，也已加入到`nodejs`的默认安装内容中进行分发。

作为一名折腾爱好者，对于这种集成管理工具特别亲切，能方便进行各种切换更新。

## 问题

于是抽空对`corepack`进行了体验，体验过程中遇到的第一个问题就是使用命令`corepack enable`后，不论是`pnpm`还是`yarn`都无法正常使用。都会出现报错，`Internal Error: Error when performing the request https://registry.npmjs.org/pnpm`。

```
    at ClientRequest.<anonymous> (/usr/lib/node_modules/corepack/dist/corepack.cjs:42195:20)
    at ClientRequest.emit (node:events:390:28)
    at TLSSocket.socketErrorListener (node:_http_client:447:9)
    at TLSSocket.emit (node:events:390:28)
    at emitErrorNT (node:internal/streams/destroy:157:8)
    at emitErrorCloseNT (node:internal/streams/destroy:122:3)
    at processTicksAndRejections (node:internal/process/task_queues:83:21)
```

看到出现`registry`的问题，第一反应是由于国内墙了，然后访问不了。赶忙确认`~/.npmrc`和`~/.yarnrc`是否已经配置好了。

检查后确认已经配置好，并无问题。尝试用命令`pnpm config get`和`yarn config get`来进行确认，但这两命令也会提示无法连接到`https://registry.npmjs.org/pnpm`。


## 发现

仔细观察报错日志，发现问题来自于`corepack.cjs`。这就又带来一个问题，为什么输入`pnpm`和`yarn`后，会是`corepack`报错呢？

研究后发现`pnpm`是调用的`~/AppData/Roaming/npm/pnpm.ps1`或者是`~/AppData/Roaming/npm/pnpm.CMD`
```powershell
#!/usr/bin/env pwsh
$basedir=Split-Path $MyInvocation.MyCommand.Definition -Parent

$exe=""
if ($PSVersionTable.PSVersion -lt "6.0" -or $IsWindows) {
  # Fix case when both the Windows and Linux builds of Node
  # are installed in the same directory
  $exe=".exe"
}
$ret=0
if (Test-Path "$basedir/node$exe") {
  # Support pipeline input
  if ($MyInvocation.ExpectingInput) {
    $input | & "$basedir/node$exe"  "$basedir/node_modules/corepack/dist/pnpm.js" $args
  } else {
    & "$basedir/node$exe"  "$basedir/node_modules/corepack/dist/pnpm.js" $args
  }
  $ret=$LASTEXITCODE
} else {
  # Support pipeline input
  if ($MyInvocation.ExpectingInput) {
    $input | & "node$exe"  "$basedir/node_modules/corepack/dist/pnpm.js" $args
  } else {
    & "node$exe"  "$basedir/node_modules/corepack/dist/pnpm.js" $args
  }
  $ret=$LASTEXITCODE
}
exit $ret
```

然后`pnpm.js`中更简单
```javascript
#!/usr/bin/env node
require('./lib/corepack.cjs').runMain(['pnpm', ...process.argv.slice(2)]);
```

这个疑问就算解决了。那么回归正题，corepack如何设置npmmirror呢？关于这个问题我到[corepack的issue列表中去查是否已有类似的问题](https://github.com/nodejs/corepack/issues)

还真发现有人在讨论该问题，并已有人提交了`PR`并被合并了。

[问题讨论](https://github.com/nodejs/corepack/issues/92), [问题讨论](https://github.com/nodejs/corepack/issues/66)

[PR](https://github.com/nodejs/corepack/pull/186)

[对比](https://github.com/nodejs/corepack/pull/186/files)

## 结论

通过对比不难发现，__新加入了一个系统环境变量`COREPACK_NPM_REGISTRY`__,通过这个环境变量就可以为`corepack`制定`mirror`，后续的就回到了`yarn`和`pnpm`自己的配置文件上了。

![readme](1.png)

![impl](2.png)

![env](3.png)

设置之后就可以正常使用`pnpm`和`yarn`了。


