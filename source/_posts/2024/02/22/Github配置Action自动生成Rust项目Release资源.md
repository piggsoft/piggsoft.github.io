---
title: Github配置Action自动生成Rust项目Release资源
date: 2024-02-22 22:15:00
tags: 
  - Github
  - Github Actions
  - Rust
  - Rust release
categories:
  - ["Github", "Actions"]
  - ["Rust"]
---

如果我们在Github上开源了一款Rust项目，那么我们就可以借助Github Action来完成自动在不同平台上打包，并在创建Release后上传到Github Release中。

<!--more-->

## 找寻合适的Action

在[Github Marketplace](https://github.com/marketplace?type=actions)中寻找我们需要的Action

建议搜索条件为 https://github.com/marketplace?category=&query=rust+sort%3Apopularity-desc&type=actions&verification=

我使用的Action为:

- [create-gh-release-action 创建Release](https://github.com/taiki-e/create-gh-release-action)
- [upload-rust-binary-action 构建程序并上传到Release中](https://github.com/taiki-e/upload-rust-binary-action)

## 配置

在Git根目录下创建`.github/workflows`目录，目录下创建Action配置文件，名称可自定义，我的文件为`.github/workflows/release.yaml`

在文件中填下如下内容
```yaml
name: Release

permissions:
  contents: write

on:
  push:
    tags: [ "v[0-9]+.*" ]

env:
  CARGO_TERM_COLOR: always
  RUST_BACKTRACE: 1

defaults:
  run:
    shell: bash

jobs:
  create-release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: taiki-e/create-gh-release-action@v1
        with:
          branch: master
          # (Optional)
          changelog: CHANGELOG.md
          # (Optional) Format of title.
          # [default value: $tag]
          # [possible values: variables $tag, $version, and any string]
          # title: $version
          # (Required) GitHub token for creating GitHub Releases.
          token: ${{ secrets.GITHUB_TOKEN }}

  upload-assets:
    needs: create-release
    strategy:
      matrix:
        os:
          - ubuntu-latest
          - macos-latest
          - windows-latest
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: taiki-e/upload-rust-binary-action@v1
        with:
          # (required) Comma-separated list of binary names (non-extension portion of filename) to build and upload.
          # Note that glob pattern is not supported yet.
          bin: tools  #需要添加自己的名称，package.name
          # (optional) On which platform to distribute the `.tar.gz` file.
          # [default value: unix]
          # [possible values: all, unix, windows, none]
          tar: unix
          # (optional) On which platform to distribute the `.zip` file.
          # [default value: windows]
          # [possible values: all, unix, windows, none]
          zip: windows
          # (required) GitHub token for uploading assets to GitHub Releases.
          token: ${{ secrets.GITHUB_TOKEN }}

```

这其中需要注意的两项

- `branch: master`需要替换为自己的主分支名
- `bin: tools`需要替换为自己的构建名称，主要就是`Cargo.toml`中的`package.name`

## 触发

Action使用的push tag触发，也就是需要再本地打一个tag，并推到Github才能触发。并且这个Tag有名称要求`tags: [ "v[0-9]+.*" ]`,可以替换为自己的。

识别到这个版本号后，会在根目录找到`CHANGELOG.md`,找到对应的版本号下的日志。

例如打了一个`v0.0.1`的tag，那么`CHANGELOG.md`的内容就为
```markdown
# Changelog

## [Unreleased]

## [0.0.1] - 2024-02-22

- change something
- add something

```

## Demo

[release.yaml](https://github.com/piggsoft/tools/blob/master/.github/workflows/release.yaml)