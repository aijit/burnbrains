# shell 拾遗： 文本数据处理

在 linux shell 中，不同工具、程序间通过文本进行数据交互，即以文本作为输入、输出标准格式。文本成为公共的协议，使得整个 shell 成为一个开放式环境，这给文本数据的处理带来了很大的方便。

## 重定向
程序的输出可通过符号 `>`、`>>` 实现覆盖、追加重定向到文件。我们用 `echo` 命令（回显，将输入参数原样输出到控制台）和`cat` 命令（吐文件，将文件内容输出到控制台显示）演示重定向。

- echo

	`$ echo hello world`

	> hello world

-  重定向 `覆盖` 到文件 test.txt

	$  echo -ne "this\nis\na\n" `> test.txt`
	
	`$ cat test.txt `

	> this
	>
	>	is
	>
	>	a

- 重定向 `追加` 到文件 test.txt

	$ echo -ne "test\ntext.\n" `>> test.txt`
	
	`$ cat test.txt `
	
	> this
	>
	> is
	>
	> a
	>
	> test
	>
	> text.

- `标准输出` 重定向到 std.txt，`标准错误` 重定向到 err.txt

	$ (echo hello world && cat a)  `> std.txt`  `2> err.txt`

	`$ cat std.txt`

	> hello world
	
	`$ cat err.txt`

	> cat: a: No such file or directory

- `标准错误` 重定向到 `标准输出`

	$ (echo hello world && cat a) > std.txt `2>&1`

	`$ cat std.txt`

	> hello world
cat: a: No such file or directory
## 管道
通过管道（使用符号 | 表示），可以把一个程序（cat）的输出，作为另一个程序（echo）的输入。

- `cat` test.txt `|` xargs `echo`
	> this is a test text.

## `grep` 模式搜索

简单的模式搜索可以使用 `grep` 命令来完成。

- Usage: `grep` [OPTION...] PATTERNS [FILE...]

- 常用 OPTION
	> -n 显示行号
	>
	> -H 显示文件名
	>
	> -h 隐藏文件名
	>
	> -r 递归整个目录中的文件
	>
	> -o 只显示匹配的内容
	>
	> -E 使用扩展的正则表达式 PATTERN

## `sed` 匹配替换

- Usage: `sed` OPTIONS... [SCRIPT] [INPUTFILE...]

- 常用 OPTIONS
	> -i 在原文件中 in-place 执行操作
	>
	> -e 指定 SCRIPT，可以有多个，如 `sed -e 'SCRIPT1' -e 'SCRIPT2' [INPUTFILE...]`
	>
	> -f 从文件中加载 SCRIPT
	>
	> -r, -E 使用扩展的正则表达式
	>
	> -u 不使用缓存
	>
	> -n 取消自动输出，只输出 p 操作内容

- [SCRIPT] 格式： `[addr]X[options]`
	
	- `[addr]` 用于定位需要处理的行
	
		| `[addr]` 格式 | 用例 | 说明 |
		|----|----|----|
		| 行号| `1` | 第 `1` 行 |
		| 行 range | `15,30` | 第 `15` ~ `30` 行 |
		| 正则匹配 | `/foo/` | 包含字符串 `foo` 的行 |
		| 取反  |  `1!` | 除第 `1` 行外的所有行 |
	
	- `X` 是操作命令，`[options]` 是 `X` 命令的参数
	
	| `X` | `[options]` | 用例 | 说明 |
	| ---- | ---- | :----- | :---- |
	|  `i`  |  要插入的内容   |  `1i before` | 在第`1`行前插入一行，内容为 `before`   |
	|  `a` |  要追加的内容   |  `1a after` | 在第`1`行后追加一行，内容为 `after`   |
	|  `r` |  file   |  `1r  title.txt` | 读取文件 `title.txt` 中的内容，输出到第`1`行后面   |
	|  `w` |  file   |  `/foo/w  title.txt` | 将包含 `foo` 的行输出到文件 file.txt  |
	|  `s` |  `/`pattern`/`replace`/[g]`   |  `s/foo/bar/g` |  将所有 `foo` 替换为 `bar`，不加 `g` 则只替换每行中第一次出现的 `foo` |
	|  `z` |  -   | `/foo/ z` |  将包含 `foo` 的行清空，保留这个空行 |
	|  `d` |  -   | `/foo/ d` |  删除包含 `foo` 的行 |
	|  `q` | exit-code | /foo/q 2 | 处理完包含 `foo` 的行就退出，返回错误码 `2` 到shell |
	|  `=` |  -   | `=` |  输出所有行的行号 |
		
	- 对匹配的行，执行多个 `SCRIPT` 操作
	
		在一条 `sed` 命令中，可以使用多条用 `;` 间隔开的 `SCRIPT`，形如 `sed '[a1]X1[o1]; [a2]X2[o2]; [a3]X3[o3]' [INPUTFILE…]`。



## 参考

- [GNU grep manual](https://www.gnu.org/software/grep/manual/grep.html)
- [GNU sed, a stream editor](https://www.gnu.org/software/sed/manual/sed.html)
- [GNU Awk User’s Guide](https://www.gnu.org/software/gawk/manual/gawk.html)