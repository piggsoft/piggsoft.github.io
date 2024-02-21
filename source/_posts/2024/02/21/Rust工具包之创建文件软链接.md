---
title: Rust工具包之创建文件软链接
date: 2024-02-21 23:49:44
tags:
  - Rust
  - Windows创建软链接
categories: 
  - ["IT", "命令", "创建软链接"]
  - ["Rust", "Tools"]
---

在编程的世界里，Rust以其独特的魅力和高效的性能吸引了无数开发者的目光。这门系统编程语言不仅提供了无与伦比的安全性，还拥有着接近C语言的性能表现。对于那些渴望深入底层，同时又不愿放弃现代语言特性的开发者来说，Rust无疑是一座桥梁，连接着传统与未来。今天，我们将一同踏上自学Rust的征程，目标是实现一个看似简单，却充满挑战的任务——在Windows操作系统中创建软链接。

在Windows环境下，软链接（Symbolic Link）是一种特殊的文件指针，它包含了指向另一个文件或目录的路径。软链接的使用，可以让文件系统的组织更加灵活，也让文件的访问和管理变得更加高效。然而，要在Rust中实现这一功能，我们需要深入了解Windows API，掌握Rust的系统调用接口，以及熟练使用Rust的文件系统库。

在这个过程中，我们不仅要学习Rust的基本语法，还要熟悉其类型系统、内存管理机制，以及如何与C语言库进行交互。我们将一步步构建起创建软链接的程序，从解析命令行参数开始，到调用Windows API函数，再到处理可能出现的错误，每一个环节都是对我们Rust知识的考验。

随着程序的逐渐成型，我们不仅能够体会到Rust在系统编程领域的强大能力，还能深刻理解软链接在文件系统中的作用。通过这个具体的例子，我们可以看到Rust如何在保证安全的同时，提供足够的灵活性来操作底层系统。

最终，当我们在命令行中输入一个简单的命令，看到软链接被成功创建时，那份成就感将是无价的。这不仅仅是对Rust知识的一个实践，更是对编程能力的一种提升。自学Rust，实现Windows创建软链接，这不仅是一个技术挑战，也是一次深入探索计算机科学奥秘的旅程。让我们一起开始这段精彩的旅程吧！

<!--more-->

## 设计流程

我这个创建软链接的主要目的是将C盘的部分目录下的内容迁移到D盘，并且尽量不修改应用的配置（有些应用就无法修改）。

将这个需求拆分为如下几步

- 用户输入原始目录和迁移后的目录
- 让用户确认目录信息是否正确
- 将原始目录中的内容迁移到新的目录中
- 删除掉原目录
- 在原目录相同的路径下，以新目录为来源创建一个软链接

## 选择依赖包

- 迁移选择[copy_dir](https://crates.io/search?q=copy_dir)
- 删除选择[rm_rf](https://crates.io/crates/rm_rf)
- 创建链接选择[symlink](https://crates.io/crates/symlink)

## 命令行参数解析

命令行参数解析可以自己来写，也可以使用三方的lib来实现，我这个包的定位是后续的工具，所以我选择[clap](https://crates.io/crates/clap)来实现。

这个工具能很方便的进行子命令的定义，参数的定义，帮助信息等一些方便的脚手架功能。

**注意**: 引入clap需要加入features的定义，不然编译会报错。
```toml
clap = { version = "4.5.1", features = ["derive", "cargo"] }
```

## 核心代码

```rust
pub(crate) fn make_link(source: &str, target: &str) {
    let source_path = &source[..];
    let binding = source
        .replace("C:\\", "D:\\")
        .replace("c:\\", "d:\\")
        .replace("C:/", "D:/")
        .replace("c:/", "d:/");
    let mut target_path = &binding[..];
    if target != "" {
        target_path = &target[..];
    }
    println!("原始目录为：{source_path}");
    println!("软连接迁移后的目录为：{target_path}");
    println!("请确认是否将原始目录: {source_path}，迁移到：{target_path}，并将原始目录删除后建立软连接？ Y or N ？");

    let mut user_input: String = String::new();
    io::stdin()
        .read_line(&mut user_input)
        .expect("读取输入出错！");


    if user_input.trim().to_ascii_uppercase().as_str() != "Y" {
        println!("程序正在退出！");
        return;
    }

    println!("---------------开始目录拷贝---------------");
    let result = copy_dir::copy_dir(source_path, target_path);
    match result {
        Ok(errors) => {
            if !errors.is_empty() {
                errors.iter().for_each(|err| { eprintln!("目录拷贝出错: {}", err) });
                process::exit(-1);
            }
        },
        Err(err) => {
            eprintln!("目录拷贝出错: {:?}", err);
            process::exit(-1);
        },
    }
    println!("---------------结束目录拷贝---------------");

    println!("---------------开始原始目录删除---------------");
    let result = rm_rf::ensure_removed(source_path);
    if let Err(err) = result {
        eprintln!("目录删除出错: {}", err);
        process::exit(-1);
    }
    println!("---------------结束原始目录删除---------------");

    println!("---------------开始建立软链接---------------");
    let result = symlink::symlink_auto(target_path, source_path);
    if let Err(err) = result {
        eprintln!("建立软链接出错: {}", err);
        process::exit(-1);
    }
    println!("---------------结束建立软链接---------------");

    println!("---------------Make Link Done!---------------");
}
```

## 入口代码

```rust
#[derive(Parser)] // requires `derive` feature
#[command(name = "tools", bin_name = "tools", version = "0.1.0", about = "Piggsoft的工具包")]
enum ToolsCli {
    #[command(name = "makeLink", version="0.1.0", about = "将<source>文件迁移到<target>，并将<source>变为软链接")]
    MakeLink(MakeLinkArgs),  //subcommand

    #[command(name = "Host", about = "打印host")]
    Host(MakeLinkArgs),  //subcommand
}


#[derive(clap::Args)]
struct MakeLinkArgs {
    #[arg(required = true, long, short = 's', help = "原始文件或者目录")]
    source: String,
    #[arg(required = false, long, short = 't', default_value = "", help = "迁移的目标文件或者目录，保证在迁移前无该文件或目录", long_help = "不传时，将使用<source_path>，并将其开头的<C:>改成<D:>")]
    target: String,
}


fn main() {
    match ToolsCli::parse() {
        ToolsCli::MakeLink(args) => make_link::make_link(args.source.as_str(), args.target.as_str()),
        _ => println!("不支持的命令"),
    }
}
```

## 完整代码
[Piggsoft Rust Tools](https://github.com/piggsoft/tools)
