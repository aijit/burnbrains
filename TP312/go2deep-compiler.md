# go 编译选项、伪指令、代码生成

golang 是易用的，在构建代码时，go 程序为我们自动完成对 compile、link 工具的配合调用流程，构建命令十分简单，形如：

`go build [-o output] [build flags] [packages]`

golang 易用的同时，也保留了一定的灵活性，如通过 [build flags] 对编译、链接过程做个性化定制。

## 1 build flags

* -gcflags
	向 go tool compile 传递 **编译选项** 列表，各选项用空格分开
* -ldflags
	向 go tool link 传递 **链接选项** 列表，各选项用空格分开
* -x
	打印构建过程中调用的命令
* -n
	打印构建过程中调用的命令，但不执行它们
* -a
	即使可执行文件已经是最新的，也强制执行重新构建的过程
* -work
	打印并保留构建过程的临时目录


## 2 编译选项

* -m
	打印编译器所做的优化
* -N
	禁止优化
* -l
	禁止内联
* -race
	编译过程中做竞态检测
* -e
	清除报错数量上限，默认是10
* -h
	遇到第一个错就停止，并打印堆栈

## 3 编译器伪指令
编译选项是针对所有 go 文件生效的，控制粒度比较粗。go 提供了另一种在代码行级别干预编译器行为的方式，即编译器伪指令（Compiler Directives）。这种机制有点类似 gcc 的 attribute。编译器伪指令以注释的形式写在代码中，形如 `//go:directive` 。

* noinline

写在函数定义前一行，确保这个函数不被内联调用。优先级高于编译选项指定的规则。
```go
//go:noinline
	func DonnotInlineMe(){
}
```

* linkname

`linkname` 用于把在一个未导出的变量或函数，导出到一个指定的包中（从而可在定义包外部引用）。由于 linkname 颠覆了包的模块化规则，所以需要引入 unsafe 包 `import _ "unsafe"` 才能使用。

使用格式：
`//go:linkname localname [importpath.name]`
```go
// A/f.go
package A;
func af(){
}
```

```go
// B/f.go
package B;
//go:linkname Af B.af
func Af()
```

## 4 链接选项

* -X importpath.name=value
	初始化变量值
* -buildmode mode
	构建模式，如 exe(默认)/plugin(动态链接库)/c-archive
* -s
	删除符号表、调试信息
* -w
	删除 DWARF 符号表
* -E entry
	指定程序入口符号

## 5 cgo 选项

当 go 构建 c 代码时，可以通过多种方式传递编译选项给 c 编译器：

* 选项写在 c 代码注释中
	* 编译 `// #cgo CFLAGS:`
	* 链接 `// #cgo LDFLAGS:`
* 选项定义在环境变量中
	* 编译 `export CGO_CFLAGS=`
	* 链接 `export CGO_LDFLAGS=`

## 6 代码生成

在 go 文件中以注释的形式 `//go:generate cmd` 声明对 shell 命令 cmd 的调用。我们称这个 cmd 为 **生成命令**。通常在 go build 前，先执行一次 go generate 触发 **生成命令** 执行，来做具体的代码生成工作。

* 设置生成命令别名

```go
// src.go
// 声明 cmd [parameters] 命令别名为 aliasCmd 
//go:generate -command aliasCmd cmd [parameters]

// 调用命令 aliasCmd
//go:generate aliasCmd
```
执行 `go generate src.go` 则触发执行 `cmd [parameters]` 命令，cmd 即可以是单纯的 shell 命令，也可以是执行某个 shell 脚本。

* 生成命令中可用的环境变量

> $GOFILE
>
>	文件名，不包含路径和扩展名
>
> $GOLINE
>
>	`//go:generate cmd` 在文件中的行号
>
> $GOPACKAGE
>
>	包名，含完整路径
>
> $GOOS
>
>	操作系统名，如 windows, linux 等


## 参考
[cmd: go build](https://pkg.go.dev/cmd/go#hdr-Compile_packages_and_dependencies)

[cmd: go tool compile](https://pkg.go.dev/cmd/compile#hdr-Compiler_Directives)

[cmd: go tool link](https://pkg.go.dev/cmd/link)

[cmd: cgo](https://pkg.go.dev/cmd/cgo)

[cmd: go generate](https://pkg.go.dev/cmd/go#hdr-Generate_Go_files_by_processing_source)

[https://go.dev/blog/generate](https://go.dev/blog/generate)

[https://learnku.com/go/](https://learnku.com/go/)
[gcc attribute syntax](https://gcc.gnu.org/onlinedocs/gcc/Attribute-Syntax.html)