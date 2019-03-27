---
title: NODE中的全局对象
date: 2019-03-26 11:13:07
categories:
- Node
tags:
---
在浏览器中全局对象是**window**，在Node.js中全局对象是**global**。
所有**全局变量**都是**全局对象**的属性。
# __filename
__filename 表示当前正在执行的脚本的文件名，它将输出文件所在位置的绝对路径。
~~~ js
// app.js
console.log(__filename);
~~~
~~~ sh
E:\hexo>node app.js
E:\hexo\app.js
~~~
# __dirname
__dirname 表示当前执行脚本所在的目录。
~~~ js
// app.js
console.log(__dirname);
~~~
~~~ sh
E:\hexo>node app.js
E:\hexo
~~~
# process 进程
用于描述当前Node.js 进程状态的对象，提供了一个与操作系统的简单接口。
## process事件
### beforeExit事件
当 Node.js 的事件循环数组已经为空，并且没有额外的工作被添加进来，事件 'beforeExit' 会被触发。
### disconnect事件
当 IPC 通道关闭时，会触发'disconnect'事件。
### exit事件
当进程准备退出时触发。
两种情况下 'exit' 事件会被触发：
- 显式调用 process.exit() 方法，使得 Node.js 进程即将结束；
- Node.js 事件循环数组中不再有额外的工作，使得 Node.js 进程即将结束。

**exit**事件监听器的回调函数，只允许包含同步操作。所有监听器的回调函数被调用后，任何在事件循环数组中排队的工作都会被强制丢弃，然后 Nodje.js 进程会立即结束。
~~~ js
process.on('exit', (code) => {
    setTimeout(() => {
        console.log('该函数不会被执行');
    }, 0);
  console.log(`即将退出，退出码：${code}`);
});
~~~
### message事件
当子进程收到父进程发送的消息时(消息通过 childprocess.send() 发送），会触发 'message' 事件。
### rejectionHandled事件
如果有 Promise 被 rejected，并且此 Promise在 Node.js 事件循环的下次轮询及之后期间，被绑定了一个错误处理器（例如使用 promise.catch()），会触发 'rejectionHandled' 事件。
### uncaughtException事件
如果Javascript有未捕获的异常，沿着代码调用路径反向传递回事件循环，会触发 'uncaughtException' 事件。 
Node.js 默认情况下会将这些异常堆栈打印到 stderr 然后进程退出。 为 'uncaughtException' 事件增加监听器会覆盖上述默认行为。
~~~ js
process.on('uncaughtException', (err) => {
  fs.writeSync(1, `捕获到异常：${err}\n`);
});

setTimeout(() => {
  console.log('这里仍然会运行。');
}, 500);

// 故意调用一个不存在的函数，应用会抛出未捕获的异常。
nonexistentFunc();
console.log('这里不会运行。');
~~~
正确使用uncaughtException事件的方式，是用它在进程结束前执行一些已分配资源（比如文件描述符，句柄等等）的同步清理操作。 触发uncaughtException事件后，用它来尝试恢复应用正常运行的操作是不安全的。
想让一个已经崩溃的应用正常运行，更可靠的方式应该是启动另外一个进程来监测/探测应用是否出错， 无论uncaughtException事件是否被触发，如果监测到应用出错，则恢复或重启应用。
### Signal事件
当Node.js进程接收到一个信号时，会触发信号事件。例如SIGINT, SIGHUP等。
## process.abort()
process.abort()方法会使Node.js进程立即结束，并生成一个core文件。
## process.arch
process.arch属性返回一个表示操作系统CPU架构的字符串，Node.js二进制文件是为这些架构编译的。
例如：'arm', 'arm64', 'ia32', 'mips', 'mipsel', 'ppc', 'ppc64', 's390', 's390x', 'x32', 或 'x64'。
## process.argv
process.argv返回一个数组，里面包含了启动Node.js进程时的命令行参数。
~~~ js
console.log(process.argv)
~~~
~~~ sh
E:\hexo>node app.js -p 80 12=333
[ 'C:\\Program Files\\nodejs\\node.exe',
  'E:\\hexo\\app.js',
  '-p',
  '80',
  '12=333' ]
~~~
## process.channel
process.channel属性保存IPC channel的引用。 如果IPC channel不存在，此属性值为undefined。
## process.chdir(directory)
process.chdir()方法变更Node.js进程的当前工作目录，如果变更目录失败会抛出异常(例如，如果指定的目录不存在)。
~~~ js
console.log(`Starting directory: ${process.cwd()}`);
try {
  process.chdir('/tmp');
  console.log(`New directory: ${process.cwd()}`);
} catch (err) {
  console.error(`chdir: ${err}`);
}
~~~
## process.config
process.config 属性返回一个Javascript对象。此对象描述了用于编译当前Node.js执行程序时涉及的配置项信息。 这与执行./configure脚本生成的config.gypi文件结果是一样的。
## process.connected
只要IPC channel保持连接，process.connected属性就会返回true。 process.disconnect()被调用后，此属性会返回false。
## process.cpuUsage()
process.cpuUsage()方法返回包含当前进程的用户CPU时间和系统CPU时间的对象。
此对象包含user和system属性，属性值的单位都是微秒(百万分之一秒)。 user和system属性值分别计算了执行用户程序和系统程序的时间。
## process.cwd()
process cwd() 方法返回 Node.js 进程当前工作的目录。
## process.disconnect()
process.disconnect()函数会关闭到父进程的IPC频道，以允许子进程一旦没有其他链接来保持活跃就优雅地关闭。
调用process.disconnect()的效果和父进程调用ChildProcess.disconnect()的一样。
## process.emitWarning(warning[, options])
process.emitWarning()方法可用于发出定制的或应用特定的进程警告。
## process.env
process.env属性返回一个包含用户环境信息的对象。
## process.execArgv
process.execArgv 属性返回当Node.js进程被启动时，Node.js特定的命令行选项。
## process.execPath
process.execPath 属性，返回启动Node.js进程的可执行文件所在的绝对路径。
## process.exit([code])
process.exit()方法以结束状态码code指示Node.js同步终止进程。
## process.exitCode
当进程正常结束，或通过process.exit()结束但未传递参数时，此数值标识进程结束的状态码。
给process.exit(code)指定一个状态码，会覆盖process.exitCode的原有值。
## process.getegid()
process.getegid()方法返回Node.js进程的有效数字标记的组身份。
PS：这个函数只在POSIX平台有效(在Windows或Android平台无效)。
## process.geteuid()
process.geteuid()方法返回Node.js进程的有效数字标记的用户身份。
PS：这个函数只在POSIX平台有效(在Windows或Android平台无效)。
## process.getgid()
process.getgid()方法返回Node.js进程的数字标记的组身份。
PS：这个函数只在POSIX平台有效(在Windows或Android平台无效)。
## process.getgroups()
process.getgroups()方法返回数组，其中包含了补充的组ID。 如果包含有效的组ID，POSIX会将其保留为未指定状态，但 Node.js 会确保它始终处于状态。
PS：这个函数只在POSIX平台有效(在Windows或Android平台无效)。
## process.getuid()
process.getuid()方法返回Node.js进程的数字标记的用户身份。
PS：这个函数只在POSIX平台有效(在Windows或Android平台无效)。
## process.kill(pid[, signal])
signal <string> | <number> 将发送的信号，类型为string或number。默认为'SIGTERM'。
process.kill()方法将signal发送给pid标识的进程。
信号名称是如'SIGINT' 或 'SIGHUP'的字符串。
## process.mainModule
process.mainModule属性提供了一种获取require.main的替代方式。 
区别在于，若主模块在运行时中发生改变， require.main可能仍然指向变化之前所依赖的模块 一般来说，假定require.main和process.mainModule引用相同的模块是安全的。
## process.memoryUsage()
process.memoryUsage()方法返回Node.js进程的内存使用情况的对象，该对象每个属性值的单位为字节。
## process.nextTick(callback[, ...args])
process.nextTick()方法将 callback 添加到"next tick 队列"。 一旦当前事件轮询队列的任务全部完成，在next tick队列中的所有callbacks会被依次调用。
这种方式不是setTimeout(fn, 0)的别名。它更加有效率。事件轮询随后的ticks 调用，会在任何I/O事件（包括定时器）之前运行。
## process.pid
process.pid属性返回进程的PID。
## process.platform
process.platform属性返回字符串，标识Node.js进程运行其上的操作系统平台。
## process.ppid
process.ppid 属性返回当前父进程的进程ID。
## process.release
process.release 属性返回与当前发布相关的元数据对象，包括源代码和源代码头文件 tarball的URLs。
## process.send(message[, sendHandle[, options]][, callback])
如果Node.js进程是通过进程间通信产生的，那么，process.send()方法可以用来给父进程发送消息。 接收到的消息被视为父进程的ChildProcess对象上的一个'message'事件。
## process.setegid(id)
id <string> | <number> 一个用户组名或用户组ID
process.setegid()方法为进程设置有效的用户组ID。(请看 setegid(2).) id可以传一个数值ID或传一个用户组名称字符串。如果传了后者的话，会解析成一个相关的数值ID， 解析的时候，这个方法方法是阻塞的。
PS: 这个方法只在POSIX平台可用(换句话说，Windows或Android不行)。
## process.seteuid(id)
id <string> | <number> A user name or ID
process.seteuid()方法为进程设置有效的用户ID。(请看 seteuid(2).) id可以传一个数值ID或传一个用户名字符串。如果传了特定的用户名字符串，会解析成一个相关的数值ID， 解析的时候，这个方法方法是阻塞的。
PS: 这个方法只在POSIX平台可用(换句话说，Windows或Android不行)。
## process.setgid(id)
id <string> | <number> 进程组名字或ID
process.setgid() 为进程方法设置组ID. (查看setgid(2).) 可给id参数传一个数值ID或字符串名。
如果已经有一个进程组ID名，那么在解析为相关的ID之前，此方法是阻塞。
PS: 这个方法只在POSIX平台可用(换句话说，Windows或Android不行)。
## process.setuid(id)
process.setuid(id) 设置进程的用户ID (参见 setuid(2).) id 可以是一个数值ID也可以是一个用户名字符串. 如果已经有一个用户名，在解析为相关的数值ID时，此方法阻塞。
## process.stderr
process.stderr 属性返回连接到stderr(fd 2)的流。 它是一个net.Socket(它是一个Duplex流)，除非 fd 2指向一个文件，在这种情况下它是一个**可写**流。
## process.stdin
process.stdin 属性返回连接到 stdin (fd 0)的流。 它是一个net.Socket(它是一个Duplex流)，除非 fd 0指向一个文件，在这种情况下它是一个Readable流。
~~~ js
process.stdin.setEncoding('utf8');
process.stdin.on('readable', () => {
  const chunk = process.stdin.read();
  if (chunk !== null) {
    process.stdout.write(`data: ${chunk}`);
  }
});

process.stdin.on('end', () => {
  process.stdout.write('end');
});
~~~
process.stdin 返回的 Duplex 流, 可以在旧模式下使用,兼容node v0.10。 更多信息查看流的兼容性。
## process.stdout
process.stdout 属性返回连接到 stdout (fd 1)的流。 它是一个net.Socket (它是一个Duplex流)， 除非 fd 1 指向一个文件，在这种情况下它是一个**可写**流。
## process.title
process.title 属性用于获取或设置当前进程在 ps 命令中显示的进程名字。
## process.umask([mask])
process.umask()方法用于返回或设置Node.js进程的默认创建文件的权限掩码。
## process.uptime()
process.uptime() 方法返回当前 Node.js 进程运行时间秒长。
## process.version
process.version 属性返回Node.js的版本信息。
## process.versions
process.versions属性返回一个对象，此对象列出了Node.js和其依赖的版本信息。 
## **Exit Codes**
- 1 未捕获异常 - 有一个未被捕获的异常, 并且没被一个 domain 或 an 'uncaughtException' 事件处理器处理。
- 2 - 未被使用 (Bash为防内部滥用而保留)
- 3 内部JavaScript 分析错误 - Node.js的内部的JavaScript源代码 在引导进程中导致了一个语法分析错误。 这是非常少见的, 一般只会在开发Node.js本身的时候出现。
- 4 内部JavaScript执行失败 - 引导进程执行Node.js的内部的JavaScript源代码时，返回函数值失败。 这是非常少见的, 一般只会在开发Node.js本身的时候出现。
- 5 致命错误 - 在V8中有一个致命的错误. 比较典型的是以FATALERROR为前缀从stderr打印出来的消息。
- 6 非函数的内部异常处理 - 发生了一个内部异常，但是内部异常处理函数 被设置成了一个非函数，或者不能被调用。
- 7 内部异常处理运行时失败 - 有一个不能被捕获的异常。 在试图处理这个异常时，处理函数本身抛出了一个错误。 这是可能发生的, 比如, 如果一个 'uncaughtException' 或者 domain.on('error') 处理函数抛出了一个错误。
- 8 - 未被使用. 在之前版本的Node.js, 退出码8有时候表示一个未被捕获的异常。
- 9 - 不可用参数 - 也许是某个未知选项没有确定，或者没给必需要的选项填值。
- 10 内部JavaScript运行时失败 - 调用引导函数时， 引导进程执行Node.js的内部的JavaScript源代码抛出错误。 这是非常少见的, 一般只会在开发Node.js本身的时候出现。
- 12 不可用的调试参数 - --inspect 和/或 --inspect-brk 选项已设置，但选择的端口号无效或不可用。
- 128 退出信号 - 如果Node.js的接收信号致命诸如 SIGKILL 或 SIGHUP，那么它的退出代码将是 128 加上信号的码值。 这是POSIX的标准做法，因为退出码被定义为7位整数，并且信号退出设置高位，然后包含信号码值。

# console
console 用于提供控制台标准输出。
Node.js 提供了与chrome浏览器行为一致的 console 对象，用于向标准输出流（stdout）或标准错误流（stderr）输出字符。
## Console.assert()
判断第一个参数是否为真，false的话抛出异常并且在控制台输出相应信息。
## Console.clear()
清空控制台。
## console.count([label])
label <string> 计数器的显示标签。 默认为 'default'。
维护一个指定 label 的内部计数器并且输出到 stdout 指定 label 调用 console.count() 的次数。
~~~js
var user = "";
function greet() {
  console.count();
  return "hi " + user;
}
user = "bob";
greet();
user = "alice";
greet();
greet();
console.count();
~~~
Console 的输出如下:
~~~sh
"<no label>: 1"
"<no label>: 2"
"<no label>: 3"
"<no label>: 1"
~~~
## console.countReset()
label <string> 计数器的显示标签。 默认为 'default'。
重置指定 label 的内部计数器。
## console.error()
打印一条错误信息。
## console.group([...label])
打印树状结构，配合groupCollapsed以及groupEnd方法;
## console.groupCollapsed()
创建一个新的内联 group。使用方法和group相同，不同的是groupCollapsed打印出来的内容默认是折叠的。
## console.groupEnd()
结束当前Tree
## console.info()
打印以感叹号字符开始的信息，使用方法和log相同
## console.log()
打印字符串，使用方法比较类似C的printf格式输出。
## console.table()
将列表型的数据打印成表格。
## console.time(label)
启动一个定时器，用以计算一个操作的持续时间。 定时器由一个唯一的 label 标识。 
当调用 console.timeEnd() 时，可以使用相同的 label 来停止定时器，并以毫秒为单位将持续时间输出到 stdout。 定时器持续时间精确到亚毫秒。
## console.timeEnd(label)
接受一个参数作为标识，结束特定的计时器。
~~~js
console.time('100-elements');
for (let i = 0; i < 100; i++) {}
console.timeEnd('100-elements');
// 打印 100-elements: 225.438ms
~~~
## console.trace()
打印字符串 'Trace :' 到 stderr ，并通过 util.format() 格式化消息与堆栈跟踪在代码中的当前位置。
## console.warn()
打印一个警告信息，可以使用 string substitution 和额外的参数。