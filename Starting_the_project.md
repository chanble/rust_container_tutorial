开始
===================================================

*[用rust写个容器](./README.md)第二篇*

本篇文章将做一些准备。

每个程序员都有自己的编程习惯，有些人喜欢上来开始写核心代码。我比较喜欢从从外围写起，搭好架子然后实现。我发现这样更容易看懂，并且使实现代码更清晰。
对于一个rust新手也正好可以学习一下rust语言的参数解析、错误处理、日志记录等等。

*这里跳过rust的安装， 请自行查阅[rust文档](https://kaisery.github.io/trpl-zh-cn/)*

## 创建一个工程

rust语言的logo是一个小螃蟹，接下来我们就把小螃蟹装进容器里。

我们创建一个名为`crabcan`的项目，我们将严格按照不同功能进行拆分，以便清晰的阅读、修改代码。甚至几个月后依然很快的明白代码。

执行`cargo new --bin crabcan`创建项目
将自动生成一个`Cargo.toml`文件， 这个文件包含项目的项目名称、描述、依赖包和编译配置等(*译者注： 类似nodejs中的package.json或php中的composer.json文件*)。 你可以修改这个文件中的内容，或者增加依赖。

`src`文件夹中放源代码。目前应该有一个`mian.rs`文件，这个文件还有一个`hello world`程序。

## 解析命令行参数

现在让我们开始写代码，首先我们需要解析命令行参数。
如：
```sh
crabcan --mount ./mountdir/ --uid 0 --debug --command "bash"
```
这里有4个参数：
- `mount`映射容器的根目录在现实主机的位置是`./mountdir/`
- `uid` UID是0
- `debug ` 是否打印调试信息
- `bash ` 进入容器中执行的命令

### structopt库说明

structopt是一个解析命令行参数很好的库(基于`clap`)，使用非常简单。
我们只需定义一个包含所有参数的结构体。
```rust
#[derive(Debug, StructOpt)] // 标注是StructOpt的结构体
#[structopt(name = "example", about = "An example of StructOpt usage.")] // 
struct Opt {
    /// 是否开启debug模式
    // 自动生成短参数（-d）和长参数(--debug)
    #[structopt(short, long)]
    debug: bool

    // 省略其他参数
}
```
了解更多`StructOpt`的用法点[这里](https://docs.rs/structopt/latest/structopt/)。 另外： 上面有个用`///`注释的内容，这里的内容将在帮助文档(`--help`)中显示出来

### 解析参数

在`src`中创建一个`cli.rs`文件。这个文件负责解析命令行。

在`mian.rs`如下写：
```rust
// 引入cli.rs
mod cli;

fn main() {
    let args = cli::parse_args();
}
```
从上面的代码可以看出，我们希望cli.rs文件中有一个`parse_args`函数，并且解析所有的参数。
*注意： 上面代码编译会出现`args`变量未使用的警告*

下面我们实现一下cli.rs文件
```rust
use std::path::PathBuf;
use structopt::StructOpt;

#[derive(Debug, StructOpt)]
#[structopt(name = "crabcan", about = "A simple container in Rust.")]
pub struct Args {
    /// Activate debug mode
    #[structopt(short, long)]
    debug: bool,

    /// Command to execute inside the container
    #[structopt(short, long)]
    pub command: String,

    /// User ID to create inside the container
    #[structopt(short, long)]
    pub uid: u32,

    /// Directory to mount as root of the container
    #[structopt(parse(from_os_str), short = "m", long = "mount")]
    pub mount_dir: PathBuf,
}

pub fn parse_args() -> Args {
    let args = Args::from_args();

    // 判断参数是否解析正确，这里先不管
    // todo
    // If args.debug: Setup log at debug level
    // Else: Setup log at info level

    // Validate arguments

    args
}

```

从代码(`use structopt::StructOpt`)可以看出，我们需要依赖`structopt`类库。另外一个PathBuf(`use std::path::PathBuf;`)是标准库包含的，不需要额外引入。
