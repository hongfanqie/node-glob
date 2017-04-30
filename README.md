# Glob

通过星号等 shell 所用的模式匹配文件。

这是一个 JavaScript 版本的 glob 实现。使用 [minimatch](https://github.com/isaacs/minimatch) 执行匹配操作。

译注：早期 Unix （第 1-6 版，1969-1975）的命令行解释器依赖独立程序 */etc/glob* 展开参数中的通配符。这个程序会展开通配符并把展开后的文件列表传给命令。它的名字是 "global command" 的简称。后来这个功能由工具函数 glob() 提供，被 shell 等程序使用。（译自 [WikiPedia](https://en.wikipedia.org/wiki/Glob_%28programming%29#Origin)。）

## 用法

用 npm 安装：

```
npm i glob
```

```javascript
var glob = require("glob")

// options 可选
glob("**/*.js", options, function (er, files) {
  // files 是一个文件名数组。
  // 如果设置了选项 `nonull` 并且没有找到匹配，则 files 是 ["**/*.js"]
  // er 是一个错误对象或 null。
})
```

## Glob 基础

"Globs" 是像这样的模式：用在命令行中的 `ls *.js`, 用在 `.gitignore` 文件中的 `build/*`。

在解析路径中的模式前，先展开大括号部分。大括号部分以 `{` 开始，以 `}` 结束，里面是一个逗号分隔列表。大括号部分可以包含斜杠，例如 `a{/b/c,bcd}` 将展开为 `a/b/c` 与 `abcd`。

译注：比如路径 "/foo/bar", 其中 "foo" 与 "bar" 是路径片段(path portion)。

下面这些字符在路径片段中有特别的意义：

* `*` 匹配路径片段中零个或多个字符。
* `?` 匹配一个字符。
* `[...]` 匹配一个字符集合，类似于正则表达式的字符集合。如果第一个字符是 `!` 或 `^` 那么它匹配一个不在这个字符集合内的字符。
* `!(pattern|pattern|pattern)` 匹配不匹配模式的文件。
* `?(pattern|pattern|pattern)` 匹配这些模式零次或一次。
* `+(pattern|pattern|pattern)` 匹配这些模式一次或多次。
* `*(a|b|c)` 匹配这些模式零次或多次。
* `@(pattern|pat*|pat?erN)` 匹配这些模式一次。
* `**` 即 globstar 模式，如果这是单独的一个路径片断，则匹配零级或多级目录，但不会搜索符号链接目录。

### 点号

如果文件或目录路径片段的第一个字符是点号（`.`），那么它将不匹配任何 glob，除非 glob 相应的路径片段的第一个字符也是 `.`。

例如，`a/.*/c` 匹配 `a/.b/c`，但是 `a/*/c` 不匹配，因为 `*` 第一个字符不是 `.`。

可以设置选项 `dot:true`，将 `.` 当作普通的字符。

译注：点文件（dot file），名字以 `.` 开始，在 Unix 下是隐藏文件。即使使用 globstar 模式，`a/**/c` 也不会匹配 `a/.b/c`。

### 匹配基本名

如果设置选项 `matchBase:true`，并且模式不包含斜杠，那么将搜索目录树下任意地方的匹配基本名（basename）的文件。例如 `*.js` 匹配 `test/simple/basic.js`。

### 空集

如果没有找到匹配的文件，那么返回一个空的数组。这跟 shell 不同，shell 会返回模式。例如：

```bash
$ echo a*s*d*f
a*s*d*f
```

想与 shell 一致，设置选项 `nonull:true`。

### 其它参考

* `man sh`
* `man bash` (搜索 "Pattern Matching")
* `man 3 fnmatch`
* `man 5 gitignore`
* [minimatch 文档](https://github.com/isaacs/minimatch)

## glob.hasMagic(pattern, [options])

如果模式包含特殊的字符则返回 `true`，否则返回 `false`。

注意选项会影响结果。如果设置了选项 `noext:true`，则 `+(a|b)` 不会视为魔法模式。如果模式包含大括号展开式，比如 `a/{b/c,x/y}`，则认为是魔法的，除非设置了选项 `nobrace:true`。

## glob(pattern, [options], cb)

* `pattern` `{String}` 待匹配的模式
* `options` `{Object}`
* `cb` `{Function}`
  * `err` `{Error | null}`
  * `matches` `{Array<String>}` 匹配模式的文件名

进行一个异步的 glob 搜索。

## glob.sync(pattern, [options])

* `pattern` `{String}` 待匹配的模式
* `options` `{Object}`
* return: `{Array<String>}` 匹配模式的文件名

进行一个同步的 glob 搜索。

## Class: glob.Glob

实例化 `glob.Glob` 类，创建一个 Glob 对象。

```javascript
var Glob = require("glob").Glob
var mg = new Glob(pattern, options, cb)
```

这是一个 EventEmitter 对象，立刻开始遍历文件系统搜索匹配。

### new glob.Glob(pattern, [options], [cb])

* `pattern` `{String}` 待匹配的模式
* `options` `{Object}`
* `cb` `{Function}` 当遇到错误或找到匹配时调用
  * `err` `{Error | null}`
  * `matches` `{Array<String>}` 匹配模式的文件名

注意，如果设置了选项 `sync`，则匹配将立即添加到 `g.found`。

### 属性

* `minimatch` glob 所用的 minimatch 对象。
* `options` 传入的选项。
* `aborted` 布尔值，当调用 `abort()` 时设为 true。取消之后不能继续 glob 搜索，不过可以通过重用 statCache 避免重复调用 syscall。
* `cache` 缓存。每个字段都可以取下面的值：
  * `false` - 路径不存在
  * `true` - 路径存在
  * `'FILE'` - 路径存在，并且不是目录
  * `'DIR'` - 路径存在，并且是目录
  * `[file, entries, ...]` - 路径存在, 并且是目录，数组值是 `fs.readdir` 的结果
* `statCache` 缓存 `fs.stat` 的结果，阻止多次读取同一路径的信息。
* `symlinks` 记录哪些路径是符号链接，与 `**` 解析相关。
* `realpathCache` 可选对象，传给 `fs.realpath`，以减少不必要的 syscall。它保存在 Glob 实例上，可以重用。

### 事件

* `end` 当结束搜索匹配时触发此事件，包含所有的匹配。如果设置了选项 `nonull`，并且没有找到匹配，则 `matches` 包含原来的模式。匹配经过排序，除非设置了选项 `nosort`。
* `match` 每当找到一个匹配时以这个匹配触发此事件，匹配没有去重，也没有解析为真实路径。
* `error` 当遇到一个异常时, 或者在设置了 `options.strict` 的情况下遇到 fs 错误时触发此事件。
* `abort` 当调用 `abort()` 时触发此事件。

### 方法

* `pause` 暂停搜索。
* `resume` 继续搜索。
* `abort` 取消搜索。

### 选项

所有可以传给 Minimatch 的选项也可以传给 Glob，选项会改变匹配行为。有些选项是新加的，有些选项是 glob 特定选项。

所有选项默认是 false, 除非特别说明。

所有选项也会添加给 Glob 对象。

如果运行多个 `glob` 操作，可以将一个 Glob 对象作为 `options` 参数传递给后面的操作，以简化一些 `stat` 和 `readdir` 的调用。在最新的版本里，你可以传入共享的 `symlinks`, `statCache`, `realpathCache`, `cache` 选项，这样并行的 glob 操作将因为共享文件系统的信息而提速。

* `cwd` String，搜索的工作目录，默认为 `process.cwd()`。
* `root` String，以 `/` 开始的模式的挂载目录，默认为 `path.resolve(options.cwd, "/")` (Unix 系统下为 `/`，Windows 系统下为 `C:\` 或其它磁盘根目录。)
* `dot` 在常规匹配与 `globstar` 匹配中包含点文件。注意，`.` 在模式片断中始终匹配点文件。
* `nomount` 以 `/` 开始的模式默认挂载到 root 选项设置的目录上，因而返回一个合法的文件系统路径。设置此选项禁止此行为。
* `mark` 给匹配的目录添加一个 `/` 字符。注意这会调用 `stat`。
* `nosort` 不排序结果。
* `stat` stat 所有的结果。这多少会降低性能，完全没必要，除非认为 `readdir` 不能作为文件存在的可靠指示。
* `silent` 当读取目录时遇到一个不常见的错误，将打印一条警告到 stderr。设置此选项可取消打印。
* `strict` 当读取目录时遇到一个不常见的错误，进程将继续搜索其它匹配。设置此选项可抛出错误。
* `cache` Object, 见上文。传入之前生成的缓存对象可以节省一些 fs 调用。
* `statCache` Object, 匹配结果的文件系统信息的缓存，用来阻止不必要的 `stat` 调用。通常不需要设置此选项。不过如果知道文件系统在不同的 glob() 调用之间不会变化，可以将一个 glob() 调用的 statCache 传给另一个调用的选项（见下面“竞态条件”）。
* `symlinks` Object, 已知的符号链接的缓存。可以传入一个之前生成的 `symlinks` 对象，在匹配 `**` 时节省 `lstat` 调用。
* `sync` 废弃，可以用 `glob.sync(pattern, opts)`。
* `nounique` 在有些情况下，大括号展开式会导致在结果里面同一文件出现多次。本实现默认阻止结果里面出现重复。此选项禁止此行为。
* `nonull` 不返回空集，返回一个包含模式的集合，这是 glob(3) 的默认行为。
* `debug` 启用 minimatch 和 glob 调试。
* `nobrace` 不展开 `{a,b}` 和 `{1..3}` 这样的集合。
* `noglobstar` 不支持 "globstar" 模式，这时 `**` 不匹配多级文件名，像普通的 `*` 一样对待。
* `noext` 不支持 "extglob" 模式，比如 `+(a|b)`。
* `nocase` 匹配不区分大小写。注意：在不区分大小写的系统里，默认匹配没有特殊字符的模式，因为 `stat` 和 `readdir` 不会抛出异常。
* `matchBase` 如果模式不包含斜杠则匹配基名字。例如 `*.js` 将视为 `**/*.js`，匹配所有目录下的 js 文件。
* `nodir` 不匹配目录，只匹配文件。注意如果只匹配目录，简单地在模式的末尾放一个 `/` 即可。
* `ignore` 添加一个模式或一个 glob 模式数组，用来排除匹配。注意：`ignore` 模式**始终**认为 `dot:true`，不管其它的配置是怎样的。
* `follow` 在展开 `**` 时追踪符号链接目录。注意这可能导致大量重复的引用（循环链接）。
* `realpath` 在所有的结果上调用 `fs.realpath`，在不能解析符号链接的情况下，返回匹配文件的全路径，不过它常常是一个损坏的符号链接。
* `absolute` 设为 true 时始终得到匹配文件的绝对地址。不同于 `realpath`，这同时影响 `match` 事件的返回值。

## 与其它 fnmatch/glob 实现的比较

严格地兼容现实规范是值得追求的目标，不过 node-glob 与其它实现之间存在差异，并且是有意的。

默认支持 `**`，除非设置了选项 `noglobstar`。这也是 bsdglob 和 bash 4.3 的方式。只有当 `**` 是单独的一个路径片段时它才有这种特殊意义。例如 `a/**/b` 匹配 `a/x/y/b`，但是 `a/**b` 不会。

注意，`**` 不会搜索符号链接目录，尽管它们可能匹配模式的其它片断。这可以防止无限循环、重复等。

如果转义的模式没有找到匹配，并且设置了选项  `nonull`，则 glob 原样返回模式，而不是转义后的模式。例如 `glob.match([], "\\*a\\?")` 返回 `"\\*a\\?"` 而不是 `"*a?"`。默认的行为类似于在 bash 里设置 `nullglob` 选项，除了 bash 不会解析转义的模式。

如果没有禁止展开大括号，则在解析 glob 的其它模式之前先展开它。例如 `+(a|{b),c)}`，在 bash 或 zsh 下面是无效的，在这儿会先展开为 `+(a|b)` 和 `+(a|c)`，再检查这两个模式的有效性。既然它们是有效的，则进行匹配。

### 注释与排除

在之前的版本中，如果模式以 `#` 开始则它是一个注释。标记为注释。如果模式以 `!` 开始则它是一个排除模式。

v5 已经废弃了选项 `nonegate` 和 `nocomment`。v6 则删除了这两个选项。

如果想排除某些文件，可以使用选项 `ignore` 。

## Windows

**请在 glob 表达式里只使用斜杠。**

译注：斜杠（forward-slashe "/"）是顺时针方向，反斜杠（backward-slashe "\"）是逆时针方向。

尽管 Windows 可以用 `/` 或 `\` 作为路径分隔符，但是本实现只使用 `/`。在 glob 表达式里必须只使用斜杠。反斜杠始终视为转义符，而不是路径分隔符。

绝对路径模式比如 `/foo/*`，匹配结果挂载到选项 root 设置的目录上（使用 `path.join()`）。 在 Windows 下，在默认的情况下，`/foo/*` 可以匹配到 `C:\foo\bar.txt`（译注：此时 cwd 在 C 盘下）。

## 竞态条件

Glob 搜索本质上容易受竞态条件（race conditions）的影响，因为它建立在目录遍历等上面。

因此，当 glob 搜索某个文件时它是存在的，然后在返回结果时它可能被删除或被修改。

作为内部实现的一部分，为了降低系统开销，此实现缓存了所有的 stat 和 readdir 的结果。但是，这也导致它更加容易受竞态条件的影响，特别是在多个 glob 调用之间重用 cache 或 statCache 对象时。

在面对快速的变化时，建议用户不要将 glob 结果作为文件系统状态的担保。对于绝大多数的操作，这绝不会是一个问题。

## 贡献

对程序行为的任何改变（包含补丁）必须同时提交测试。

测试失败的或降低性能的补丁将被拒绝。

```
# to run tests
npm test

# to re-generate test fixtures
npm run test-regen

# to benchmark against bash/zsh
npm run bench

# to profile javascript
npm run prof
```

## 翻译

[本文](https://github.com/isaacs/node-glob#readme) 由 [Ivan Yan](http://yanxyz.net/) 翻译，
译文采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>。

译文[更新](update.md)，意见[反馈](https://github.com/hongfanqie/node-glob/issues)。
