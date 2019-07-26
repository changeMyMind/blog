### Shell 脚本查漏补缺

---

[不懂得可以点这里进行查询]([http://linux.51yip.com/](http://linux.51yip.com/))  [学习的话可以点这里哦](https://github.com/fengyuhetao/shell) 实在感觉不行的话 使用 —help 或者 -h 可能会有些帮助

#### 简单加法入门

```shell
#!/bin/bash
#退出状态码，最大为255，超过则进行模运算
#testing the exit status
var1=101
var2=201
var3=$[$var1+$var2]
echo The answer is $var3
exit 5
```

#### 使用expr执行数学运算

```shell
#!/bin/bash
# An example of using the expr command

var1=10
var2=20
var3=`expr $var2 / $var1`
echo The result is $var3
```

### 小数计算处理方式

1. bc + scale定义精度

```shell
#!/bin/bash
# 注意 shell命令中 `` 和 $() 等价
var1=12
var2=7
var3=`echo "scale=2;$var1/$var2"|bc`
echo The result is $var3
```

2.  awk 定义精度

```shell
#!/bin/bash
# awk定义精度 跟c语言输出差不多 在awk中 记得对应的变量参数需要添加单引号  比如想引用var1变量使用 '$var1'
# 进行赋值的时候(计算所得) 可以使用 `` 或者 $() 
var1=12;var2=7
var3=`awk 'BEGIN{printf "%.2f\n",('$var1'/'$var2')}'`
echo The result is $var3
```

3.  乘法浮点计算

```shell
#!/bin/bash
# 乘法基本操作
var1=20.1;var2=0.2
var3=`echo "scale=2;$var1 * $var2"|bc`
echo The result is $var3
```



#### 时间格式化

```shell
date "+%Y-%m-%d %H:%M:%S" 
# 显示为2019-05-31 11:20:23 这种格式
```

```shell
3#!/bin/bash
# copy the /usr/bin directory listing to a log file
today=`date "+%Y-%m-%d %H:%M:%S"`
ls /usr/bin -al > log.$today
```



####快速清楚文件内容

```shell
cat /dev/null > fileName
```



####  分行读取文件内容

```shell
#!/bin/bash
# reading data from a file
count=1
cat test | while read line
do
	echo "Line $count: $line"
	count=`expr $count + 1`
done
echo "Finished processing the file"
```



#### N的迭代 参数携带

```shell
#! /bin/bash
# n! $1 -> n 第一个参数表示 命令后携带的第一个参数
factorial=1
for (( number=1;number<=$1;number++ ))
do
	factorial=$[ $number * $factorial ]
done
echo The result is $factorial
```

#### N的迭代 用户输入

```shell
#! /bin/bash
# 用户输入N 并输出N的迭代 如果没有指定值 赋值到REPLY 参数中 如果指定 
# 如：read -p "Enter a number:" number 那么就在 number参数中
read -p "Enter a number:"
factorial=1
for (( number=1;number<=$REPLY;number++ ))
do
	factorial=$[ $number * $factorial ]
done
echo The result is $factorial
```



### if then else

```shell
#!/bin/bash
# testing the else section
selection="-1"
while [ $selection = "-1" ]
do
	read -p "Enter Select：" str
	if [ $str = "-1" ];then
		selection="-2"
		echo "结束"
	else
		# 小测试 当然直接使用case 比较好 这里就是玩一玩 不要强迫症搞事情 自己想优化自己优化去
		case $str in
		-2)
			echo "字符串-2";;
		ts)
			echo "字符串ts";;
		1)
			echo "正确";;
		2)
			echo "错误";;
		*)
    	echo "输入有问题";;
    esac
	fi
done
```



#### 目录判断

```shell
#!/bin/bash
# 目录判断
selection="-1"
while [ $selection = "-1" ]
do
  read -p "enter your directory: " directory
  if [ $directory = "-1" ]
  then
    selection="-2"
  	echo "结束"
  	# 此处的break 加不加都一样 加了多余
  	# break;
  elif [ -d $directory ]
  then
    echo "the directory you entered exists"
    cd $directory
    # 随便 ls 一下 可以去掉哈
    ls -a
  else
    echo "输入的路径不存在"
  fi
done
```



#### C语言风格for循环N的阶层

```shell
#!/bin/bash
# testing the C-style for loop
while [ true ]
do
	read -p "enter your number: " number
	if [ $number -eq -1 ]
	then
		echo "结束"
		break
	fi
	fastNumber=1;
	# 此处的number不能有 $ 符号哦
	for (( i=1; i<=number; i++ ))
	do
		fastNumber=$[ $fastNumber * $i ]
	done
	echo The result is $fastNumber
done
```



#### 获取字符串长度

```shell
var1=dsafsafsdf232
echo $var1 | awk '{print length($0)}'
```



#### 全局变量

```shell
#!/bin/bash
# using a global variable to pass a value
function db1 {
	value=$[ $value ** 2 ]
}
read -p "Enter a value: " value
db1
echo "The new value is : $value"
```



####局部变量

```shell
#!/bin/bash
# demostrating the local keyword
function func1 {
	local temp=$[ $value + 5 ]
	# 此处输出的是局部变量的值
	echo "$temp"
	# result没有声明local 所以是全局变量的值
	result=$[ $temp * 2 ]
}
temp=4
value=6
func1
# 此处输出的是全局变量的值
echo "$temp"

echo "The result is $result"
if [ $temp -gt $value ]
then
	echo "temp`s value has changed"
else
	echo "temp`s value has not change"
fi
```

