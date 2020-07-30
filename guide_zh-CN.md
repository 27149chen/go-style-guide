# Go Style Guide

## Go Project Layout

Please refer to [Go Project Layout](project_layout_zh-CN.md)

## Naming
1. package的名字必须是全小写的形式，不能出现 "-"，"_" 等分隔符。[[1](https://golang.org/doc/effective_go.html#package-names)] [[2](https://blog.golang.org/package-names)] [[3](https://rakyll.org/style-packages/)] [[4](https://github.com/kubernetes/community/blob/master/contributors/guide/coding-conventions.md)] [[5](https://github.com/uber-go/guide/blob/master/style.md#package-names)]
2. package的名字必须是名词，而且应该是单数，包括那些集合性质的package，比如"net/url"，"example"，"image"，"player"。 [[1](https://dmitri.shuralyov.com/idiomatic-go#use-singular-form-for-collection-repo-folder-name)] [[2](https://rakyll.org/style-packages/)] [[3](https://github.com/uber-go/guide/blob/master/style.md#package-names)]
3. package的名字应该简洁且能体现代码的内容，但需要尽量避免无意义的名称，比如util，common，misc，api，types和interfaces等。 [[1](https://github.com/golang/go/wiki/CodeReviewComments#package-names)] [[2](https://golang.org/doc/effective_go.html#package-names)] [[3](https://blog.golang.org/package-names)] [[4](https://rakyll.org/style-packages/)] [[5](https://github.com/uber-go/guide/blob/master/style.md#package-names)]
4. package的命名和type的命名应该尽量避免冗余。比如应该叫做"controllers/autoscaler"而不是"controllers/autoscalercontroller"，应该叫做"chubby.File"而不是"chubby.ChubbyFile"，应该叫做"storage.Interface"而不是"storage.StorageInterface"。比如您有一个意义很明确的timer模块，应该定义"timer.New"而不是"timer.NewTimer"。[[1](https://github.com/kubernetes/community/blob/master/contributors/guide/coding-conventions.md)] [[2](https://github.com/golang/go/wiki/CodeReviewComments#package-names)] [[3](https://blog.golang.org/package-names)]
5. 除非有特殊的理由，go文件中的package的名字应该与他所在的路径的名字一致。比如文件`foo/bar.go`中的第一行，应该是`package foo`。
6. 在import一个package时，尽量不要定义别名，除非出现冲突。在出现两个包名相同时，优先为the most local or project-specific的package定义别名。[[1](https://github.com/golang/go/wiki/CodeReviewComments#imports)]
7. 导入是按组组织的，它们之间有空白行。 标准库软件包始终在第一组中。import应该按照相似原则分组，中间用空行分隔。比如标准库为一组，第三方库为一组，本地库为一组。标准库始终在第一组中（[goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) 和一些IDE会帮您完成这一工作）。 [[1](https://github.com/golang/go/wiki/CodeReviewComments#imports)]
8. Go的命名风格是MixedCaps和mixedCaps，不使用下划线式的命名风格。其中前者可在package外被访问，后者只能在package内被访问。
9. 上一条的命名风格，同样适用于定义常量。比如您应该定义maxLength或者MaxLength，而不是MAX_LENGTH。[[1](https://github.com/golang/go/wiki/CodeReviewComments#mixed-caps)]
10. 对于存在两个以上大写字母的缩写词或者组合词，在需要大写时应该使用惯用写法（而不是首字母大写的写法），在需要小写时应该所有字母都小写。 [[1](https://github.com/golang/go/wiki/CodeReviewComments#initialisms)] [[2](https://dmitri.shuralyov.com/idiomatic-go#for-brands-or-words-with-more-than-1-capital-letter-lowercase-all-letters)]

    比如这些都是推荐写法：
    - "URLPolicy"，"ServeHTTP"，"XMLHTTPRequest"，"appID"，"OAuthEnabled"，"GitHubToken"
    - "urlPolicy"，"xmlHTTPRequest"，"oauthEnabled"，"githubToken"

    而这些则不推荐：
    - "UrlPolicy"，"ServeHttp"，"XmlHTTPRequest"，"appId"，"oAuthEnabled"，"gitHubToken"
11. 变量的名称应该尽可能简短。基本规则：变量的使用离声明越远，取名就应该越具有描述性。越近，取名就应该越简短，尤其是那些作用域很小的变量，一个或两个字母就足够了。对于方法接收者，一个或两个字母就足够了。诸如循环索引和读取器之类的常见变量可以是单个字母（i，r）。 [[1](https://github.com/golang/go/wiki/CodeReviewComments#variable-names)]
> 注意：请不要滥用这条规则，简短的前提是可读性。
12. 对某些单词使用一致的拼写。 [[1](https://github.com/golang/go/wiki/Spelling)]
    
    应该这样写：
    - "marshaling"，"unmarshaling"，"canceling"，"canceled"，"cancellation"
    
    不应该这样写：
    - "marshalling"，"unmarshalling"，"cancelling"，"cancelled"，"cancelation"

## Style
1. 对于需要使用`context.Context`作为参数的函数，尽量把他作为第一个参数，而且如果没有冲突的话，名字必须叫做`ctx`。 [[1](https://github.com/golang/go/wiki/CodeReviewComments#contexts)]

    比如：
    ```
    func F(ctx context.Context, /* other arguments */) {}
    ```
2. 代码应通过尽可能早地处理错误或者特殊情况并尽早返回或继续循环来减少嵌套。 

    应该：
    ```go
    for _, v := range data {
        if v.F1 != 1 {
            log.Printf("Invalid v: %v", v)
            continue
        }
  
        v = process(v)
        if err := v.Call(); err != nil {
            return err
        }
        v.Send()
    }
    ```

    不应该：
    ```go
    for _, v := range data {
        if v.F1 == 1 {
            v = process(v)
            if err := v.Call(); err == nil {
                v.Send()
            } else {
                return err
            }
        } else {
            log.Printf("Invalid v: %v", v)
        }
    }
    ```
3. 避免不必要的`else`。

    应该：
    ```go
    a := 10
    if b {
        a = 100
    }
    ```

    不应该：
    ```go
    var a int
    if b {
        a = 100
    } else {
        a = 10
    }
    ```
4. 使用字段名初始化Struct。初始化Struct时，应该始终指定字段的名称。 您可以通过[`go vet`](https://golang.org/cmd/vet/)来执行这项检查。
5. 声明一个空的Slice时不需要初始化。大多数情况下，在定义一个空的Slice时不需要进行初始化，只需保持默认值nil。 [[1](https://github.com/golang/go/wiki/CodeReviewComments#declaring-empty-slices)]

     应该：
    ```go
    var t []string
    ```

    不应该：
    ```go
    t := []string{}
    ```
6. Go中的变量名应该短而不是长。 对于范围有限的局部变量尤其如此。 将c首选为lineCount。 我更喜欢sliceIndex。

基本规则：名称声明越远，使用的名称就越具有描述性。 对于方法接收者，一个或两个字母就足够了。 诸如循环索引和读取器之类的常见变量可以是单个字母（i，r）。 更多不寻常的事物和全局变量需要更多描述性名称。
7. 命令行的flag如果由多个词组成，应该用"-"，而不是"_"。 [[1](https://github.com/kubernetes/community/blob/master/contributors/guide/coding-conventions.md)]

## Guidelines
1. 函数的Receiver究竟是用值还是用指针（`func (t T) foo()` or `func (t *T) foo()`），请遵循这个规范： https://github.com/golang/go/wiki/CodeReviewComments#receiver-type
2. Map，Slice等常见类型都不是并发安全的，要保证并发安全，请使用lock，channel等手段。您也可以使用`sync.Map`这个并发安全的Map，和`sync/atomic`这个包提供的原子方法。
3. 当您使用goroutine时，您必须清楚它是否会退出，以及什么时候退出。一般情况下，您应该通过WaitGroup或者channel等方式，在函数返回前等待里面的goroutine先退出。如果您想在函数返回后使里面的goroutine保持运行，您必须通过context或者channel等方式，确保在需要关闭这个goroutine时，可以控制它的关闭。否则，容易造成资源泄露。
4. 在大多数情况下，请不要使用`_`来忽略一个error，您应该处理或者返回这个error。 [[1](https://github.com/golang/go/wiki/CodeReviewComments#handle-errors)]

## Performance
> 注意：对于使用频率较低，性能要求不高或者逻辑比较简单的代码，应该优先遵循风格方面的要求，性能方面的要求不是必需的。性能优化只有在出现性能瓶颈时才需要重点考虑。届时，请着重关注这里列的注意点。

1. 优先选用strconv而不是fmt来做字符串转换，因为前者性能更好。

    应该：
    ```go
    for i := 0; i < b.N; i++ {
        s := strconv.Itoa(rand.Int())
    }
    ```

    不应该：
    ```go
    for i := 0; i < b.N; i++ {
        s := fmt.Sprint(rand.Int())
    }
    ```

2. 指定Slice或者Map的capacity。指定Slice的capacity可以在不断地append时有效地减少后台Array的扩容，比如：
    ```go
    make([]T, 0, 10)
    ```
    一般情况下，您不需要指定Map的capacity，但是如果您事先已经可以确定Map中需要存放的key的数量或者需要存放的数量较大，可以指定capacity，比如：
    ```go
    make(map[T1]T2, 100)
    ```

## Comments
1. comments应该是完整的句子，以句号结尾。这样做可以在使用godoc生成文档时呈现良好的格式。
2. 对于struct或者function，comments应该以结构名或者函数名开头，作为整个comment的主语，就算他是个动词形式的。 [[1](https://github.com/golang/go/wiki/CodeReviewComments#comment-sentences)]

    比如：
    ```
    // Request represents a request to run a command.
    type Request struct { ...

    // Encode writes the JSON encoding of req to w.
    func Encode(w io.Writer, req *Request) { ...
    ```
3. 对于package，comments可以放在任意一个同package下的go文件中，以"Package {pkgname}"开头。对于main包，则应该描述这个二进制包的作用。 [[1](https://rakyll.org/style-packages/)] [[2](https://github.com/golang/go/wiki/CodeReviewComments#package-comments)]

    比如：
    ```
    // Package ioutil implements some I/O utility functions.
    package ioutil

    // Binary seedgen ...
    package main

    // Command seedgen ...
    package main

    // Program seedgen ...
    package main

    // The seedgen command ...
    package main

    // The seedgen program ...
    package main

    // Seedgen ...
    package main

    // Sample helloworld demonstrates how to use x.
    package main
    ```
