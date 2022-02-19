## 测试操作符
- `-L`:测试该文件是否是一符号链接

## 参数替换
- `${parameter}`:与`$parameter`相同
- `${var//Pattern/Replacement}`:在变量中，如果匹配到Pattern,就用Replacement来替代，如果Replacement被忽略，那么会直接删除匹配到的Pattern,例子如下：
    ```shell
    echo $PATH #/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/local/go/bin
    sub=${PATH//:/ } #使用空格替换:
    echo $sub #/usr/local/bin /usr/bin /bin /usr/local/games /usr/games /usr/local/go/bin
    ```
## 特殊字符
- `!`:这是`bash`中的一个关键字，用于反转命令的退出状态，也可以用于反转测试操作符含义，比如：
    ```shell
    [ -d /usr/local/bin ] #测试目录
    echo $? #0
    [ ! -d /usr/local/bin ] 
    echo $? #1 退出状态被反转
    ! [ -d /usr/local/bin ]
    echo $? #1 直接放在前面，来反转[ ]这个命令的退出状态
    ```
- `` ` ``,`$()`:这两个表达用于命令替换，其可以将命令的输出传递到另一个上下文中，注意命令替换会调用一个`subshell`。尽可能的使用`$()`，其允许嵌套，书写格式上也更加清楚。

## 常用工具
- `printf`:格式化打印，加强版`echo`,与c语言`printf`语句用法类似，但没有`()`常见字符和转义序列如下：
    ```shell
    %s  字符串                  
    %c  单个字符
    %d  整数
    %o  八进制整数
    %x  十六进制整数
    %f  浮点数
    %b  带反斜杠转义符的字符串
    %%  百分号
    ```