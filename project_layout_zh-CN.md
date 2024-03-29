# Go Project Layout

到目前为止，Go 语言官方还没有一个标准或者推荐的项目布局，但是经过多年的发展，社区还是形成了一些通用的布局风格。通常来讲，一个 Go 项目通常包含以下目录：

**cmd**: 这个目录存放了该项目需要导出的所有应用的 main 入口。比如一个叫做 `my-app` 的项目，需要编译生成两个二进制文件 `myclient` 和 `myserver`，那么这个项目至少存在这样的文件 `my-app/cmd/myclient/main.go` 和 `my-app/cmd/myserver/main.go`。

> 注意：
> - 文件名 `main.go` 不是强制的，但是这个文件的包名必须是 main，而且必须定义一个 mian 函数。
> - `main.go` 所包含的内容应该尽可能少，通常只是 import 了一些东西，然后执行一个函数。
> - 就算项目只生成一个应用，也请以定义这样的结构。

**pkg**: 这个目录是项目主要代码的入口，几乎所有与功能相关的代码都放在这个目录下的相应子目录下。关于是否需要这个目录，社区有很多讨论 [[1](https://github.com/golang-standards/project-layout/issues/10)]。我们决定保留这个目录，并作如下约束：
- 对于简单项目（功能单一，除必要目录以外，项目根目录下其他目录（模块）数量小于6个），可以不使用这个目录。
- 对于主要作为库函数供其他项目调用而存在的项目，可以不使用这个目录，但是根目录下的目录结构应该按照功能来划分。比如不应该出现 models，types 或者 interfaces 等目录来存放所有的 model，类型定义或者接口，这些内容应该存放在相应的功能所在的目录下（注意：这同样适用于保留 pkg 目录的项目）[[2](https://rakyll.org/style-packages/)]。
- 对于其他项目，都必须存在这个目录，同时 pkg 的下级目录也应该按照功能来划分。

**internal**: 所有存放在公共代码仓库中的Go项目都可以作为库函数被其他项目 import，如果该项目有一些不想被外部引用的包，可以把它们放到这个目录下。

> 注意：
> - 大部分情况下都不需要这个目录，除非确实有这个需求。
> - 除了出现在项目的根目录下，internal 目录还可以出现在其他任何子目录下，只有和 internal 目录拥有相同父目录的包才能import这个目录下的包。比如包 `a/b/c/internal/d/e/f` 只能被目录 `a/b/c` 下的其他目录下的包 import，而不能被 `a/b/d` 下的包 import [[3](https://golang.org/doc/go1.4#internalpackages)]。

**scripts**: (也可以定义为 `hack`)这个目录存放了所有用于代码分析，检查和构建等的脚本。这些脚本可以被开发人员执行，也可以被项目根目录下的 `Makefile` 调用。

**build**: 这个目录存放了一些 build 相关的脚本和文件。比如 `dockerfile` 和 `docker-entrypoint.sh` 等。这个目录中还包含两个子目录，一个是 ci，用来存放 CI 相关的脚本，比如执行单元测试的脚本，用来构建和上传 image 的脚本和 CI 系统（比如 Jenkins，Travis）相关的脚本。如果某些 CI 系统对脚本目录有特殊要求，比如必须存放在根目录下，可以适当调整，或者以软连接的形式进行关联。这个目录还需要包含的一个子目录是 bin，这是一个临时目录，用来存放构建生成的二进制文件，可能会被某些 dockerfile 引用，需要写入 `.gitignore` 来避免提交到代码库中。
