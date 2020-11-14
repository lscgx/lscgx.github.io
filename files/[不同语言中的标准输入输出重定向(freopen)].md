

# [不同语言中的标准输入输出重定向(freopen)]

通常在Online Judge上的题目要求程序从标准输入流(standard input, stdin)读入输入数据，将答案输出到标准输出流(standard output, stdout)，并通常会无视标准错误输出流(standard error, stderr)的输出。当然，过多的stderr输出必然会占用许多额外的时间，导致TLE，所以即使把debug信息输出到stderr，有时也要注意通过注释和条件编译去掉for循环中的debug语句。不过更多的时候，我们主要需要关心的还是stdin和stdout。

不过有时候，某些Online Judge的某些题目就会要求从给定文件读入测试数据，或将结果输出到给定文件中，或二者皆有。大多数编程语言在提供文件IO操作API的同时，都提供有许多直接操作标准输入输出的API。而很多初学者对这些API都要比对文件操作的API熟悉得多。所以如果在此时，能用标准输入输出的API来替代文件输入输出的API无疑是很方便的。在本地，这可以通过shell的重定向轻松办到

```
./p.exe < input.txt > output.txt 2> log.txt
```

不过在Online Judge上，这一点就行不通了。好在很多语言都提供有一些API，能实现类似的功能。

除了希望使用标准输入输出的API来实现文件输入输出外。在本地调试程序的时候，有时候通过终端（屏幕）进行交互会方便得多，即使要进行文件输入输出也可以借由shell轻松自如的选择文件。所以，程序在本地使用stdio，而在Online Judge上才使用指定的文件会方便不少。

当然，前面还漏了一点，用标准输入输出的API除了可以节省力气外，还有一个好处是：如果同样一道题，在多数OJ上要用标准输入输出，但有些OJ要求文件输入输出时，代码无需经过太多修改就能AC。嗯，其实我就是在做Codeforces上新加的Andrew Stankevich Contests想到可以做这么一个总结的。下面的代码都用[Think Positive](http://codeforces.com/gym/100200)这道题测试过，可以AC的完整代码请通过[github](https://github.com/watashi/AlgoSolution/commit/3d3d96dd0810156c64c7b95cdc1738920ccee90b)访问。

#### C

首先是大家再熟悉不过的C语言的例子，直接通过`"stdio.h"`中的`freopen`函数实现标准输入输出的重定向，用`#ifndef`实现条件编译。

```c
int main(void) {
#ifndef __WATASHI__
  freopen("whatasha.in", "r", stdin);
  freopen("whatasha.out", "w", stdout);
#endif
  /* ... */
  return 0;
}
```

理想的情况，我们希望新加的代码尽量独立而不影响其它部分。不过C语言标准中并不允许我们通过如下方法：

```c
/* initializer element is not constant */
FILE``* __dummy_in = ``freopen``(``"whatasha.in"``, ``"r"``, stdin);
```

将`freopen`移到`main`之外。而C语言虽然可以根据平台来实现在`main`之前执行一段代码，不过方法都不可移植并且略微复杂，采用这样的方法来改写就本末倒置了。

#### C++

在C++中，我们就可以把这部分代码移到`main`之外了，而且还可以做得更C++一点：

```cpp
begin
  {$IFNDEF __WATASHI__}
  assign(input, 'whatasha.in');
  assign(output, 'whatasha.out');
  reset(input);
  rewrite(output);
  {$ENDIF}

  (* ... *)
end.
```

这段代码虽然不符合C++标准，不过GNU C++, GNU C++0x和MS VC++都允许’$'符号作为标识符的一部分。选用$的目的是为了避免重名。当然依个人喜好也可以将$替换成_，再加上namespace什么的。

#### Pascal

Pascal中直接对`input`和`output`进行`assign`即可。在Free Pascal中，如果打开了-So开关(Borland TP 7.0 compatible)，那么相应的`reset`和`rewrite`是必须的，否则会得到`Error 103 - File not open`。

```pascal
begin
  {$IFNDEF __WATASHI__}
  assign(input, 'whatasha.in');
  assign(output, 'whatasha.out');
  reset(input);
  rewrite(output);
  {$ENDIF}

  (* ... *)
end.
```

#### Python

许多语言没有条件编译，视其为不必要的，甚至是邪恶的。这时候，我们可以使用环境变量。

```python
if os.getenv('USER') != 'watashi':
  sys.stdin = open('whatasha.in', 'r')
  sys.stdout = open('whatasha.out', 'w')
```

#### Perl

Perl中提供了`BEGIN`, `UNITCHECK`, `CHECK`, `INIT`和`END`块，他们将按特定顺序在特定阶段被执行。将重定向代码放到`BEGIN`块中好处是，即使把它放到程序末尾，它也会最先执行。Codeforces曾经不能正确处理带`BEGIN`块的代码，不过`INIT`就没有问题，用`INIT`还有一个好处就是，它可以节省一个字节。

```perl
INIT {
  if ($ENV{'USER'} ne 'watashi') {
    open STDIN, '<', 'whatasha.in';
    open STDOUT, '>', 'whatasha.out';
  }
}
```

需要注意的是，Perl中`<>`和``其实是不同的，前者等价于``，不过由于通常`@ARGV`都为空，二者的效果是一样的。

#### Ruby

Ruby中也有类似的BEGIN块。

```ruby
BEGIN {
  if ENV['USER'] != 'watashi'
    $stdin = open('whatasha.in')
    $> = open('whatasha.out', 'w')
  end
}
```

需要说明的是，Ruby默认从`$<`读入，输出到`$>`。前者等价于ARGF，是个只读变量，试图给它赋值将会得到`$< is a read-only variable (NameError)`。后者就是`$stdout`的alias，所以我们也可以把代码改成。

```
$stdout` `= open(``'whatasha.out'``, ``'w'``)
```

当ARGV为空时，从`$>`读入就会变成从`$stdin`读入。在这一方面Ruby借鉴了Perl的许多行为。

#### D

```D
void main() {
  if (getenv("USER") != "watashi") {
    stdin = File("whatasha.in", "r");
    stdout = File("whatasha.out", "w");
  }

  // ...
}
```

#### Go

```go
func main() {
  if (os.Getenv("USER") != "watashi") {
    os.Stdin, _ = os.Open("whatasha.in")
    os.Stdout, _ = os.Create("whatasha.out")
  }

  // ...
}
```

#### Scheme

Codeforces并不支持Scheme，倒是ZOJ支持。

```scheme
(define (freopen input output)
  (set-current-input-port (open-file input "r"))
  (set-current-output-port (open-file output "w")))

(if (not (equal? (getenv "USER") "watashi"))
  (freopen "whatasha.in" "whatasha.out"))
```

#### PHP

从下面的语言开始，就没有那么freopen的解决方案了。PHP不允许我们重新定义常量，所以不能修改`STDIN`(`php://stdin`)和`STDOUT`(`php://stdout`)于是我们只能定义个变量$STDIN再做个全局替换了。对于输出到`php://output`的`print`和`echo`，还要用`ob_start`处理一下。

```php
if (getenv('USER') == 'watashi') {
  $STDIN = STDIN;
} else {
  $STDIN = fopen('whatasha.in', 'r');
  $STDOUT = fopen('whatasha.out', 'w');
  ob_start(function($buffer) {
    global $STDOUT;
    fwrite($STDOUT, $buffer);
  });
}

// replace STD{IN,OUT} with $STD{IN,OUT}
```

#### Scala

Scala写一个”main”的方法比较多，不过都可以通过下面的代码来设置`scala.Console.{in,out}`。`scala.Predef._`中的各种IO函数都是`scala.Console._`的alias。

```scala
if (util.Properties.userName != "watashi") {
  scala.Console.setIn(
    new BufferedInputStream(new FileInputStream("whatasha.in")))
  scala.Console.setOut(
    new PrintStream(new FileOutputStream("whatasha.out")))
}
```

不过`scala.Console.{in,out}`和`java.lang.System.{in,out}`并不像`std::c{in,out}`和`std{in,out}`那样会保持同步。所以如果用到了其它的IO函数，就可能会有一些问题，需要额外处理。

#### Java

理想的情况，我们只需要在静态代码块中调用`System.set{In,Out}`就好了。这也是一个非常freopen的解决方案。

```java
public class Main {
  static {
    try {
      System.setIn(new BufferedInputStream(new FileInputStream("whatasha.in")));
      System.setOut(new PrintStream(new FileOutputStream("whatasha.out")));
    } catch (IOException ex) {
    }
  }
}
```

可遗憾的是由于Codeforces的权限限制，上面的代码会抛出`java.security.AccessControlException: access denied (java.lang.RuntimePermission setIO)`。于是我们只好退而求其次，定义两个变量`in`和`out`，将原来的`System.in`和`System.out`全局替换掉。

```java
public class Main {
  static InputStream in;
  static PrintStream out;

  static {
    if (System.getProperty("ONLINE_JUDGE") == null) {
      in = new Scanner(System.in);
      out = System.out;
    } else {
      String file = "whatasha";
      try {
        in = new BufferedInputStream(new FileInputStream(file + ".in"));
        out = new PrintStream(new FileOutputStream(file + ".out"));
      } catch (IOException ex) {
      }
    }
  }

  // replace System.{in,out} with {in,out}
}
```

这里套一层BufferedInputStream有时候是必要的，否则可能会TLE。更好的办法也许是伪造一个System，毕竟全局替换是很容易有所遗漏的。

```java
final class System {
  public static InputStream in = java.lang.System.in;
  public static PrintStream out = java.lang.System.out;
  public static PrintStream err = java.lang.System.err;

  public static void exit(int status) {
    java.lang.System.exit(status);
  }

  static {
    if (java.lang.System.getProperty("ONLINE_JUDGE") != null) {
      try {
        String file = "whatasha";
        in = new BufferedInputStream(new FileInputStream(file + ".in"));
        out = new PrintStream(new FileOutputStream(file + ".out"));
      } catch (IOException ex) {
      }
    }
  }
}
```

当然，Codeforces上那些专用Java的选手早有模板来处理Java的各种IO问题了。

#### C#

类似Java，理想的情况我们可以这么做：

```c#
public class Main {
  static Main() {
    Console.SetIn(new StreamReader("whatasha.in"));
    Console.SetOut(new StreamWriter("whatasha.out"));
    (Console.Out as StreamWriter).AutoFlush = true;
  }
}
```

不过与Java可以同时有多个静态代码块不同，C#只能有一个静态构造函数。所以可以考虑写成下面这样，避免和已有的静态构造函数冲突。

```c#
#if __WATASHI__
  public static bool __freopn = ((Func<string, string, bool>)(
      (input, output) => {
        Console.SetIn(new StreamReader(input));
        Console.SetOut(new StreamWriter(output));
        (Console.Out as StreamWriter).AutoFlush = true;
        return true;
      }))("whatasha.in", "whatasha.out");
#endi
```

不过类似java的情况，`SetIn`和`SetOut`会得到Runtime error。于是我们还可以类似地伪造一个`Console`

```c#
public class Main {
#if !__WATASHI__
  class FakeConsole: StreamWriter {
    StreamReader reader;

    public FakeConsole(string input, string output): base(output) {
      reader = new StreamReader(input);
      AutoFlush = true;
    }

    public int Read() {
      return reader.Read();
    }

    public string ReadLine() {
      return reader.ReadLine();
    }
  }

  static FakeConsole Console = new FakeConsole("positive.in", "positive.out");
#endif

  // ...
}
```

顺带一提，之前在Codeforces一旦用C#进行文件操作就会抛出`System.Security.SecurityException: Request for the permission of type 'System.Security.Permissions.FileIOPermission`。看来MikeMirzayanov同学已经fix了这个bug。

#### OCaml

OCaml并没有类似`freopen`功能的函数，不过如果有`Unix`模块的支持的话，通过`openfile`和`dup2`可以实现同样的功能。事实上glibc的`freopen`就是通过`dup2`系统调用实现的，而msvcrt的`freopen`的源代码则较为复杂一些。

```ocaml
#load "unix.cma";;
open Unix;;

let freopen = fun input output ->
  let fin = openfile input [O_RDONLY] 0 in
  dup2 fin stdin;
  let fout = openfile output [O_WRONLY; O_CREAT; O_TRUNC] 0o644 in
  dup2 fout stdout;
;;

let _ =
  if getlogin () <> "watashi" then
    freopen "whatasha.in" "whatasha.out"
;
```

按照上面的规律，你应该已经猜到了这个方法在Codeforces上是不work的。如果所有的输入输出都是通过`Scanf.scanf`和`Printf.printf`完成的话，还可以做一个类似的全局替换。但如果还调用了`Pervasives`中的IO函数，那就要麻烦不少了，基本就是ad-hoc地修改了。

#### Haskell

Haskell也没有类似freopen功能的函数（别说FFI）。但在Haskell中为了避免超时，通常要把String换成Data.ByteString.Char8，甚至有时候还要把getLine换成getContents。同时，作为纯函数式语言，IO Monad天然地把IO操作给隔离了出来，所以许多情况改起来还不算太麻烦。

假如原来的程序用的是interact，那么事情就更简单了。

```c#
{-# LANGUAGE CPP #-}
import Control.Monad
import Data.Maybe
import qualified Data.ByteString.Char8 as C

interact' :: (C.ByteString -> String) -> IO ()
#ifndef __WATASHI__
interact' = hInteract "whatasha.in" "whatasha.out"

hInteract :: FilePath -> FilePath -> (C.ByteString -> String) -> IO ()
hInteract input output f =
  liftM f (C.readFile input) >>= writeFile output
#else
interact' = interact . (. C.pack)
#endif
```

