> bash编程与'正规'编程语言相比，尽管同样存在着变量，语句，各种控制结构等概念，但在使用时，某些代码并不能按照一般编程语言的直觉来撰写，而且这种'不适感'并不是简单的'与逻辑'用`&&`还是`AND`来表示这种东西所造成的。可无论怎样，这门语言的设计必然不是胡乱设计的，而是有一套准则，下面就按照常规编程语言所具有的内容来梳理一下bash是如何表达这些的。

## 变量
#### 数值变量的声明
&emsp;&emsp;当我们想声明一个变量同时给其赋值数值时我们会很自然的使用如下声明语句：
```shell
a=1
echo $a #1
b=a+1 
echo $b #a+1
c=$a+1
echo $c #1+1
```
但从结果中我们可以看到bash并没有将a看作一个数值，进而使用a的加减法也并没有被当作一个算数表达式，而是把`=`右边的内容当成是字符串来看待，那bash中如何使用变量来存储数值呢？方法有如下：
```shell
# 直接将b声明为整数，此时=右边的式子会被当成算数表达式
declare -i b
a=1
b=$a+1
echo $b #2

# 使用let命令，执行变量上的算数操作
a=1
let b=$a+1
echo $b #2

# 使用expr命令,对变量进行算数计算(注意操作符两边的空格)
a=1
b=`expr $a + 1`
echo $b #2

# 使用双括号(())
a=1
b=$((a+1))
echo $b #2
```
上述便是常见的几种将变量解释为数值的方法
## 控制结构
控制结构的语句莫过于`if/then`,`for`,`while`,在一般的编程语言中是判断一个布尔值为`true`或是`false`来决定是否执行操作。但在bash中并没有布尔值这个东西，控制结构语句所判断的是命令的返回值，即`$?`，不同的表达式或命令的返回值有不同的定义，具体情况要具体分析，另外要注意当返回值为0时其相当于`true`，这和常规编程语言是不同的。
### if/then
`if/then`结构用于测试后接命令的返回值，若 **返回0则相当于条件为`true`** 就执行后接的命令，再次注意，**if后面接的是命令，而不是布尔值那种东西，判断的是命令的返回值**，比如下面的例子：
```shell
if a=1
then
echo $a
fi #1
```
可以看到`if`后面是一个命令，并且其返回值为0,所以又执行了`then`后面的命令输出了1。完整命令结构如下：
```shell
if [ conditionl ]
then
    command1
    command2
    command3
elif [ condition2 ]
then
    command4
    command5
else
    default-command
fi
```
&emsp;&emsp;在大多数情况下我们看到的是`if []`这样的结构，在初次见到的时候一直把它当成c语言中`if()`来理解，以为`[]`是用来包含某个条件，但实际上`[]`并不是一个符号，而是一个命令，其相当于命令`test`,既然其是个命令也就可以理解为什么`[]`和其中的语句要有空格。下面就来看下`[]`的一些常见用法：
```shell
[ 0 ]
echo $? #0
[ 1 ]
echo $? #0
[ -1 ]
echo $? #0
[ ]
echo $? #1
[ $undefined ]
echo $? #1 判断一个未定义变量会返回1
```
在大多数情况下并不是在`[]`中直接判断一个变量，而是使用参数判断文件是否存在，字符串是否相等等情况，具体例子如下：
```shell
# -e判断文件是否存在
file1=/home/waitrap
[ -e $a ]
echo $? # 0 说明文件存在
# 判断数字是否相等
a=1
b=2
[ $a -eq $b ]
echo $? # 1 说明两者不等
```
其余更详细的参数在ABS书中都有列举。
### while
`while`语句的完整结构如下：
```shell
while [ condition ]
do
    command(s)...
done
```
这里的`[]`和`if...else`中的`[]`是相同的，同样`while`后面也可以接命令和`[[ ]]`。
与`while`相对应的是`until`语句，其结构如下：
```shell
until [ condition ]
do
    command(s)...
done
```
但与`while`不同的是，当条件为假时即命令不返回0时才执行循环中的命令。
### for loop
`for`循环结构如下：
```shell
for arg in list
do  
    command(s)...
done
```
这种结构和python中的循环类似，list可类比可迭代对象，使用示例如下：
```shell
for planet in "Mercury Venus Earth Mars Jupiter Saturn Uranus Neptune Pluto"
# All planets on same line.
# Entire 'list' enclosed in quotes creates a single variable.
# Why? Whitespace incorporated into the variable.
do
echo $planet
done
```
## 函数
一个函数就是一个子程序，一个函数调用就好比调用一个命令，其基本的形式如下：
```shell
function function_name{
    command...
}
or
function_name(){
    command...
}
```
不同于C语言函数，我们不会在`()`中传入参数，对于参数是使用`$1,$2..`例子如下：
```shell
func(){
    echo $1
    echo $2
}
# 调用函数
func a b # 输出a b
```
因为**函数其实就是一个命令，所以函数的返回值，即相当于命令的退出状态**，我们可以使用`return`来返回函数退出状态，例子如下：
```shell
func(){
    return 2
}
func
echo $? #2
```
不同于其它编程语言，在函数中声明的变量并不只在函数中可见，当我们调用函数后，变量在外部也可见(注意是调用后)，若想变量一直只在函数中可见，则用`local`修饰，例子如下：
```shell
func(){
    a=1
}
echo $a # 没有输出
func
echo $a # 1
func1(){
    local b=1
}
func1
echo $b # 没有输出
```
## 数组
在较新bash中支持使用一维数组，数组变量的声明可以使用`declare -a`,或者直接定义数组中的值比如:
```shell
a[1]=1
echo ${a[1]} #1 引用数组值使用${}
#直接定义数组所有值
array=(1 2 3)
echo ${array[0]} #1
echo ${array[@]} #1 2 3
echo ${array[*]} #1 2 3 @ *会返回所有值
```
一个单独的变量其实也可以看作数组，使用索引来访问，比如：
```shell
a=1
echo ${a[0]} #1

b="1 2 3"
echo ${b[0]} # 1 2 3注意是一个元素 
```
值得注意上述我们都是利用索引来寻找值，若使用键值来关联值也是可以的，比如：
```shell
declare -A a #声明为关联数组
a[q]=1
a[w]=2
echo ${a[q]} #1
echo ${a[w]} #2
```