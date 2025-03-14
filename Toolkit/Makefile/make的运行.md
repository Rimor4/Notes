## 指定目标[¶](https://seisman.github.io/how-to-write-makefile/invoke.html#id2 "Link to this heading")

- all:这个伪目标是所有目标的目标，其功能一般是编译所有的目标。
    
- clean:这个伪目标功能是删除所有被make创建的文件。
    
- install:这个伪目标功能是安装已编译好的程序，其实就是把目标执行文件拷贝到指定的目标中去。
    
- print:这个伪目标的功能是例出改变过的源文件。
    
- tar:这个伪目标功能是把源程序打包备份。也就是一个tar文件。
    
- dist:这个伪目标功能是创建一个压缩文件，一般是把tar文件压成Z文件。或是gz文件。
    
- TAGS:这个伪目标功能是更新所有的目标，以备完整地重编译使用。
    
- check和test:这两个伪目标一般用来测试makefile的流程。

## 检查规则[¶](https://seisman.github.io/how-to-write-makefile/invoke.html#id3 "Link to this heading")
`-n`, `--just-print`, `--dry-run`, `--recon`

不执行参数，这些参数只是打印命令，不管目标是否更新，把规则和连带规则下的命令打印出来，但不执行，这些参数对于我们调试makefile很有用处。

`-t`, `--touch`

这个参数的意思就是把目标文件的时间更新，但不更改目标文件。也就是说，make假装编译目标，但不是真正的编译目标，只是把目标变成已编译过的状态。

`-q`, `--question`

这个参数的行为是找目标的意思，也就是说，如果目标存在，那么其什么也不会输出，当然也不会执行编译，如果目标不存在，其会打印出一条出错信息。

`-W <file>`, `--what-if=<file>`, `--assume-new=<file>`, `--new-file=<file>`

这个参数需要指定一个文件。一般是是源文件（或依赖文件），Make会根据规则推导来运行依赖于这个文件的命令，一般来说，可以和“-n”参数一同使用，来查看这个依赖文件所发生的规则命令。

## make的参数[¶](https://seisman.github.io/how-to-write-makefile/invoke.html#id4 "Link to this heading")
> GNU make 3.80

`-b`, `-m`

这两个参数的作用是忽略和其它版本make的兼容性。

`-B`, `--always-make`

认为所有的目标都需要更新（重编译）。

`-C <dir>`, `--directory=<dir>`

指定读取makefile的目录。如果有多个“-C”参数，make的解释是后面的路径以前面的作为相对路径，并以最后的目录作为被指定目录。如：“make -C ~hchen/test -C prog”等价于“make -C ~hchen/test/prog”。

`-debug`[=_<options>_]

输出make的调试信息。它有几种不同的级别可供选择，如果没有参数，那就是输出最简单的调试信息。下面是<options>的取值：

- a: 也就是all，输出所有的调试信息。（会非常的多）
    
- b: 也就是basic，只输出简单的调试信息。即输出不需要重编译的目标。
    
- v: 也就是verbose，在b选项的级别之上。输出的信息包括哪个makefile被解析，不需要被重编译的依赖文件（或是依赖目标）等。
    
- i: 也就是implicit，输出所有的隐含规则。
    
- j: 也就是jobs，输出执行规则中命令的详细信息，如命令的PID、返回码等。
    
- m: 也就是makefile，输出make读取makefile，更新makefile，执行makefile的信息。


`-d`

相当于“–debug=a”。

`-e`, `--environment-overrides`

指明环境变量的值覆盖makefile中定义的变量的值。

`-f`=_<file>_, `--file`=_<file>_, `--makefile`=_<file>_

指定需要执行的makefile。

`-h`, `--help`

显示帮助信息。

`-i` , `--ignore-errors`

在执行时忽略所有的错误。

`-I` _<dir>_, `--include-dir`=_<dir>_

指定一个被包含makefile的搜索目标。可以使用多个“-I”参数来指定多个目录。

`-j` [_<jobsnum>_], `--jobs`[=_<jobsnum>_]

指同时运行命令的个数。如果没有这个参数，make运行命令时能运行多少就运行多少。如果有一个以上的“-j”参数，那么仅最后一个“-j”才是有效的。（注意这个参数在MS-DOS中是无用的）

`-k`, `--keep-going`

出错也不停止运行。如果生成一个目标失败了，那么依赖于其上的目标就不会被执行了。

`-l` _<load>_, `--load-average`[=_<load>_], `-max-load`[=_<load>_]

指定make运行命令的负载。

`-n`, `--just-print`, `--dry-run`, `--recon`

仅输出执行过程中的命令序列，但并不执行。

`-o` _<file>_, `--old-file`=_<file>_, `--assume-old`=_<file>_

不重新生成的指定的<file>，即使这个目标的依赖文件新于它。

`-p`, `--print-data-base`

输出makefile中的所有数据，包括所有的规则和变量。这个参数会让一个简单的makefile都会输出一堆信息。如果你只是想输出信息而不想执行makefile，你可以使用“make -qp”命令。如果你想查看执行makefile前的预设变量和规则，你可以使用 “make –p –f /dev/null”。这个参数输出的信息会包含着你的makefile文件的文件名和行号，所以，用这个参数来调试你的 makefile会是很有用的，特别是当你的环境变量很复杂的时候。

`-q`, `--question`

不运行命令，也不输出。仅仅是检查所指定的目标是否需要更新。如果是0则说明要更新，如果是2则说明有错误发生。

`-r`, `--no-builtin-rules`

禁止make使用任何隐含规则。

`-R`, `--no-builtin-variabes`

禁止make使用任何作用于变量上的隐含规则。

`-s`, `--silent`, `--quiet`

在命令运行时不输出命令的输出。

`-S`, `--no-keep-going`, `--stop`

取消“-k”选项的作用。因为有些时候，make的选项是从环境变量“MAKEFLAGS”中继承下来的。所以你可以在命令行中使用这个参数来让环境变量中的“-k”选项失效。

`-t`, `--touch`

相当于UNIX的touch命令，只是把目标的修改日期变成最新的，也就是阻止生成目标的命令运行。

`-v`, `--version`

输出make程序的版本、版权等关于make的信息。

`-w`, `--print-directory`

输出运行makefile之前和之后的信息。这个参数对于跟踪嵌套式调用make时很有用。

`--no-print-directory`

禁止“-w”选项。

`-W` _<file>_, `--what-if`=_<file>_, `--new-file`=_<file>_, `--assume-file`=_<file>_

假定目标<file>;需要更新，如果和“-n”选项使用，那么这个参数会输出该目标更新时的运行动作。如果没有“-n”那么就像运行UNIX的“touch”命令一样，使得<file>;的修改时间为当前时间。

`--warn-undefined-variables`

只要make发现有未定义的变量，那么就输出警告信息。
