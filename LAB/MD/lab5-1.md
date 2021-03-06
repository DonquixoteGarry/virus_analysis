## lab 5-1

> 1811464 郑佶 信息安全单学位

#### 问题1:定位指定函数地址

> 指定函数:`DllMain`

使用`IDA Pro`的文本视图打开`Lab05-01.dll`,可找到`DllMain`的位置,如下

![](../IMG/LAB5-1-1.png)

由此可知,`DllMain`的地址在`.text`节的`0x1000D02E`处

#### 问题2:定位指定导入函数位置

> 指定导入函数:`gethostbyname`

打开`IDA Pro`的`import`子视图浏览导入函数,找到导入函数`gethostbyname`如下![](../IMG/LAB5-1-2.png)

由此可知,导入函数`gethostbyname`的地址某节的`0x100163CC`处

双击该函数.进入导入函数`gethostbyname`的文本视图,得到以下信息

![](../IMG/LAB5-1-3.png)

由此可知,该函数位于`idata`节,地址为`0x100163CC`

#### 问题3:调用指定导入函数函数的函数个数

> 指定导入函数:`gethostbyname`

在导入函数`gethostbyname`的文本视图下,点击函数位置使用`ctrl+x`查看引用情况,得到信息如下

![](../IMG/LAB5-1-4.png)

可发现,在`Address`栏中,有`5`组相同的函数基址,`9`组相同的函数地址

可知,导入函数`gethostbyname`被`5`个函数调用了`9`次

#### 问题4:指定函数调用的分析

> 指定地址:`0x10001757`

位于`0x10001757`的对导入函数`gethostbyname`的调用,即如下位置的调用

![](../IMG/LAB5-1-5.png)

点击该函数调用位置,进入该位置的文本视图,得到如下信息

![](../IMG/LAB5-1-6.png)

可知,在调用函数前压栈的参数`off_10019040+OxD`即是给出的参数的位置.

由于导入函数`gethostbyname`的参数应该是一个代表网址的字符串,所以该位置应该是一个字符串

双击偏移量`off_10019040`,跳转到字符串所在位置,得到如下信息

![](../IMG/LAB5-1-7.png)

由此可知,该字符串的值为`[This is RDO]pics.practicalmalwareanalysis.com`,而此次函数调用发起的`DNS`请求也正是对网址`pics.practicalmalwareanalysis.com`发起的请求

#### 问题5:指定地址子过程的局部变量个数

> 指定地址:`0x10001656`

在`IDA Pro`中点击`G`键,进入地址跳转页面,输入指定地址`0x10001656`,得到以下信息

![](../IMG/LAB5-1-8.png)

可以看出,`IDA Pro`一共识别出了`23`个局部变量.

#### 问题6:指定地址子过程的参数个数

> 指定地址:`0x10001656`

图同问题5,`IDA Pro`一共识别出了`1`个参数`lpThreadParameter`.

#### 问题7:定位指定字符串

> 指定字符串:`\cmd.exe /c`

通过菜单栏中的`View/Open Subviews/Strings`打开`String`窗口,得到以下信息

![](../IMG/LAB5-1-9.png)

可知,该字符串位于`0x10095B34`处

#### 问题8:分析引用指定字符串位置的行为

> 指定字符串:`\cmd.exe /c`

字符串在内存中的位置如下,在`0x10095B34`位置

![](../IMG/LAB5-1-10.png)

点击字符串跳转到该字符串的调用位置,如下

![](../IMG/LAB5-1-11.png)

可知,该字符串是作为函数`strcat`的参数`Source`被压栈的,也就是源字符串值,而写入的地址是`ebp+CommandLine`,与控制台相关

初步判断,`cmd.exe`是`windows xp`系统的控制台程序,所以该调用与控制台程序有关.

但是可以在调用`strcat`函数前的`0x100100A3`位置,发现对函数`sprintf`的调用,而作为参数的源字符串值如下

![](../IMG/LAB5-1-12.png)

可以发现字符串`Encrypt Magic Number For This Remote Shell Session`,指的是远端`shell`,这与控制台程序相关联

据此推测,该位置的功能应该是调用远端`shell`

#### 问题9:指定地址使用的指定变量的设置方式

> 指定变量:`dword_1008E5C4`/指定地址:`0x100101C8`

使用地址跳转界面跳转到变量指定地址,得到如下信息

![](../IMG/LAB5-1-13.png)

点击变量名,跳转到内存中所在位置,键入`ctrl+x`,查看对指定变量的交叉引用情况,得到如下信息

![](../IMG/LAB5-1-14.png)

显然,其中的`mov`指令是为该变量赋值,点击跳转到这个引用的位置,得到如下信息

![](../IMG/LAB5-1-15.png)

可以发现,赋给指定变量的值来源于`eax`寄存器,而这个寄存器存储的是函数`sub_10003695`的返回值,点击该函数,调转到函数主体位置,得到以下信息

![](../IMG/LAB5-1-16.png)

显然,这个函数的功能是判断版本以及平台信息,并将版本相关信息返回.

为判断该位置分支跳转具体情况,用空格键跳转到图形视图,得到如下信息.

![](../IMG/LAB5-1-17.png)

可知,该位置通过判断指定变量的值来决定为函数`strcat`压入的参数字符串是`\\cmd.exe /c`还是`\\command.exe /c`.

经过上网查资料可知,`command.exe`是`Windows 9x`等系统的控制台程序,而因为`cmd.exe`是`windows xp`等系统的控制台程序,所以此变量是根据系统版本调用不同的控制台程序,说明函数`sub_10003695`的功能是获取系统版本信息.

综上,该指定变量的值是根据系统版本信息设置的.

#### 问题10:指定函数中与指定字符串比较结果为真时的行为

> 指定函数:`memcmp`/指定字符串:`robotwork`

可以在指定的地址`0x1000FF58`附近找到`loc_0x10010444`代码块,得到以下信息,正是调用的指定字符串的位置

![](../IMG/LAB5-1-18.png)

从函数`sub_100052A2`的形式可以看出,该函数唯一调用的参数是`SOCKET`类型参数`s`.根据压栈的参数`s`未被赋值的情况,可以判断其为引用参数,也就是赋值对象.可以判断函数`sub_100052A2`最后将查询到的值返回给了`SOCKET`型参数`s`.

其中,当`memcmp`函数比较结果为真时,会调用函数`sub_100052A2`.点击该函数,跳转到该函数主体部分,调整到图形视图,得到如下信息

![](../IMG/LAB5-1-19.png)

可以看到,在地址`0x100052F1`处,函数`sub_100052A2`调用了函数`RegOpenKeyEx`.

经过查询资料,发现函数`RegOpenKeyEx`的功能是打开指定的注册表键,而在此语句前的`5`个`push`语句压栈的值就是该函数的`5`个参数.作为参数的主键名`hKey`值为`80000002`,经查询资料,代表的值是`HKEY_LOCAL_MACHINE`.其中子键名,即字符串`offset aSoftwareMicros`的值`SOFTWARE\\Microsoft\\Windows\\CurrentVersion`.总之,函数`RegOpenKeyEx`查询了注册表中的`HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion`位置打开了键,并返回值.经查询资料,返回值表示的是打开键的结果,打开成功为`0`,打开不成功为非零值.

在图形界面中,接着可以发现,在注册表子键打开成功后,函数会跳转到`loc_10005309`,具体信息如下

![](../IMG/LAB5-1-20.png)

可见,接下来调用了函数`RegQueryValueEx`,功能是查询项目值,其中第二个参数项目名的具体值是字符串`WorkTime`,这是查询的具体项目的名字.

总而言之,此处的行为是查询注册表项目`HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\WorkTime`后将查询的信息返回参数`SOCKET`类型参数`s`.

#### 问题11:指定导出函数的行为

> 指定导出函数:`PSLIST`

打开`IDA Pro`的`Export`视图,查询导出函数表,可得到以下信息

![](../IMG/LAB5-1-21.png)

点击函数地址`0x10007025`,并切换到图形视图查看,得到以下信息

![](../IMG/LAB5-1-22.png)

可知,该函数的第一次分支判断条件依赖于函数`sub_100036C3`的返回值,点击函数`sub_100036C3`了解该函数主体,得到以下信息

![](../IMG/LAB5-1-23.png)

该函数调用了`GetVersionEx`函数取得系统版本信息,并根据得到的版本信息决定返回值.由此可知函数`PSLIST`的第一次分支依赖于系统版本信息.

另外,在函数`PSLIST`第二次分支中,函数`sub_10006518`与函数`sub_1000664C`中都调用了函数`CreateToolhelp32Snapshot`,该函数用于建立进程及其相关信息的快照.

函数`PSLIST`的分支之一调用了函数`sub_1000664C`,且函数`sub_1000664C`调用了变量`s`.根据该函数调用的函数`sub_100038BB`的参数表可知,`s`是`SOCKET`类型变量.

由此推测,该函数通过某些判断将进程信息快照赋给`SOCKET`类型变量`s`,

#### 问题12:对指定函数的交叉引用分析

> 指定函数:`sub_10004E79`

选择菜单栏的`View/Graphs/User Xrefs Chart`进入函数`sub_10004E79`的引用情况,得到如下信息

![](../IMG/LAB5-1-24.png)

从上面调用的`API`中,可以发现值得注意的`API`,例如`GetSystemDefaultLangID`和`send`,分别用于得到系统的默认`LangID`和调用`SOCKET`的`send`函数.将这些功能结合起来考虑推测该函数是将`LangID`信息以`SOCKET`形式发送

综上,可将该函数重命名为`send_DefalutLangID`

#### 问题13:指定函数调用`Windows API`的情况

> 指定函数:`DllMain`

选择菜单栏的`View/Graphs/User Xrefs Chart`进入函数`DllMain`的引用情况,选择以下设置,限定引用的深度等信息

![](../IMG/LAB5-1-25.png)

得到具体交叉引用情况,由于图片太宽不便展示,以下只说明分析结果.

直接调用的`Windows API`有`_strnicmp`、`CreateThread`、`strncpy`,共`3`个

在深度`2`调用的`Windows API`有`strchr`、`strncmp`、`WinExec`、`gethostbyname`、`Sleep`、`inet_nota`、`CreateThread`、`ExitThread`、`FreeLibrary`、`closesocket`、`closeHandle`、`select`、`WSAStartup`,共`13`个

#### 问题14:指定地址的指定函数引发的程序休眠时间

> 指定地址:`0x10001358`/指定函数:`sleep`

使用指定地址跳转界面跳转到该函数的调用位置,得到如下信息

![](../IMG/LAB5-1-26.png)

可知,函数`sleep`的参数,即休眠时间(毫秒),是地址`10001357`的`push`指令压栈前`eax`的值.

综合上下文代码,可以得出这个值的得出过程:

- `off_10019020`与`0x0D`进行`add`运算,得到`结果1`
- `结果1`作为参数调用函数`atoi`,得到返回值`结果2`
- `结果2`与`0x3E8`进行`imul`运算,得到结果

查询可知,`off_10019020+0x0D`的值为`30`.经过运算,休眠时间为`30000`毫秒,即`30`秒

#### 问题15:指定地址的指定函数的参数值

> 指定地址:`0x10001701`/指定函数:`socket`

使用指定地址跳转界面跳转到该调用位置,得到如下信息

![](../IMG/LAB5-1-27.png)

显然,`3`个参数依次为`2`、`1`、`6`

#### 问题16:修改指定函数参数

> 指定函数:`socket`

通过查阅相关资料,得到如下关键的宏定义信息

![](../IMG/LAB5-1-28.png)

由此,可以把参数修改为`AF_INET`,`SOCK_STREAM`,`IPPROTO_TCP`

#### 问题17:指定指令的使用

> 指定指令:`in`指令(`opcode=0xED`)

选择菜单栏中的`Search/text`,选择以下的搜索设置

![](../IMG/LAB5-1-29.png)

得到以下与`in`指令相关的搜索结果

![](../IMG/LAB5-1-30.png)

点击跳转到调用位置,得到如下信息

![](../IMG/LAB5-1-31.png)

可知,其中`eax`中的字符串的十六进制值为`0x564D5868`,转换为字符串值即为`VMXh`,所以魔术字符串`VMXh`的确被恶意代码使用了.

另外,在`String`界面中,可发现以下字符串

![](../IMG/LAB5-1-32.png)

跳转到调用该字符串的位置,得到如下信息

![](../IMG/LAB5-1-33.png)

可以发现,在调用位置附近的`0x1000DEE1`,调用了函数`sub_10006196`,这正是`in`指令的所在位置,这更证明了该魔术字符串可以用于检测`VMWare`虚拟机

#### 问题18:分析指定地址代码

> 指定地址:`0x1001D988`

使用指定地址跳转界面跳转到该位置,得到如下信息

![](../IMG/LAB5-1-34.png)

看起来是没有实际意义的字符序列

#### 问题19:使用指定脚本分析指定地址代码

> 指定脚本:`Lab05-01.py`/指定地址:`0x1001D988`

选择菜单栏的`File/Script file`,选定给定的脚本,得到如下信息

![](../IMG/LAB5-1-35.png)

可以发现,无意义字符串被解密,转换为了有意义的字符串`xdoor is this backdoor,string decoded for Practical Malware Analysis Lab :)1234`

#### 问题20:无意义字符序列的转换方式

> 无意义字符串序列即`问题18`得到的字符序列

点击地址`0x1001D988`,点击`A`键,就可以得到如下结果

![](../IMG/LAB5-1-36.png)

#### 问题21:分析指定脚本

> 指定脚本:`Lab05-01.py`

使用记事本打开该脚本,如下

![](../IMG/LAB5-1-37.png)

可以发现该脚本的运行流程为

- 函数`ScreenEA`获取光标位置
- 进入循环体
  - 函数`Byte`获取指定位置字符
  - 进行异或运算
  - 函数`PatchByte`回显运算结果

- 循环结束