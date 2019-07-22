- SHELL的类型
 Linux Shell的种类很多，目前流行的Shell包括ash、bash、ksh、csh、zsh等。

```
cat /etc/shells  #查看自己主机中当前有哪些种类的Shell
/bin/sh  
/bin/bash  
/sbin/nologin  
/bin/bash2  
/bin/ash  
/bin/bsh  
/bin/tcsh  
/bin/csh
 echo $SHELL       #查看当前shell类型
/bin/bash 
```
- COMMAND [options] [ARGUMENTS]
  1.选项:用于启用或者关闭命令的某些功能
	   1.1短选项 -c -h
	   1.2长选项 --all
  2.参数:命令的作用对象,比如文件名,用户名
  3.注意
      3.1多个选项以及参数之间用空白分隔符分开
      3.2取消和结束命令:ctrl+c,ctrl+d
      3.3多个命令用";"分开
      3.4一个命令可以用"\"分成多行
- which
which命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。
-a   查看命令包括的所有文件夹
-n 　指定文件名长度，指定的长度必须大于或等于所有文件中最长的文件名。
-p 　与-n参数相同，但此处的包括了文件的路径。
-w 　指定输出时栏位的宽度。
-V 　显示版本信息

```
which --skip-alias ls 不显示别名信息
echo $PATH #查看外部命令的存储路径
which pwd #查看pwd所在的位置
```
- whatis命令
 描述:用于查询一个命令执行什么功能，并将查询结果打印到终端上。
```
	[root@localhost ~]# whatis ls
	ls                   (1)  - list directory contents
	ls                   (1p)  - list directory contents
	
	[root@localhost ~]# whatis cp
	cp                   (1)  - copy files and directories
	cp                   (1p)  - copy files
	
	[root@localhost ~]# whatis chown
	chown                (1)  - change file owner and group
	chown                (1p)  - change the file ownership
	chown                (2)  - change ownership of a file
	chown                (3p)  - change owner and group of a file
	
	[root@localhost ~]# whatis man
	man                  (1)  - format and display the on-line manual pages
	man                  (1p)  - display system documentation
	man                  (7)  - macros to format man pages
	man                 (rpm) - A set of documentation tools: man, apropos and whatis.
	man-pages           (rpm) - Man (manual) pages from the Linux Documentation Project.
	man.config [man]     (5)  - configuration data for man


```

- type
描述:区别外部命令和内部命令
操作:type -a可以显示所有可能的类型，比如有些命令如pwd是shell内建命令，也可以是外部命令。
type -p只返回外部命令的信息，相当于which命令。
type -f只返回shell函数的信息。
type -t 只返回指定类型的信息。
- enable
描述:临时关闭或者激活指定的shell内部命令
操作:-n：关闭指定的内部命令；
-a：显示所有激活的内部命令；
-f：从指定文件中读取内部命令。
- alias
描述:来设置指令的别名
 1.设置别名 alias 别名='原命令 -选项/参数'
 ```
alias ll='ls -lt'
 ```
 2.查看已经设置的别名列表
 ```
alias -p
 ```
 3.删除别名 格式：unalias name
```
unalias cp
```
 4.生效
 <1>.若要每次登入就自动生效别名，则把别名加在/etc/profile或~/.bashrc中。然后# source ~/.bashrc
 <2>.若要让每一位用户都生效别名，则把别名加在/etc/bashrc最后面，然后# source /etc/bashrc
 5.重名问题
	如果别名同原命令同名，如果要执行原命令，可使用
    \ALIANNAME
	"ALIANNAME"
	COMMAND ALIANNAM		
	'COMMAND'
	/PATH/COMMAND
 6.命令的优先级
   命令优先级(别名>内部>外部)
- echo
功能说明：显示文字。
语 　 法：echo [-ne][字符串]或 echo [--help][--version]
补充说明：echo会将输入的字符串送往标准输出。输出的字符串间以空白字符隔开, 并在最后加上换行号。
参　　 数：-n 不要在最后自动换行
-e 若字符串中出现以下字符，则特别加以处理，而不会将它当成一般
文字输出：
   \a 发出警告声；
   \b 删除前一个字符；
   \c 最后不加上换行符号；
   \f 换行但光标仍旧停留在原来的位置；
   \n 换行且光标移至行首；
   \r 光标移至行首，但不换行；
   \t 插入tab；
   \v 与\f相同；
   \\ 插入\字符；
   \nnn 插入nnn（八进制）所代表的ASCII字符；
–help 显示帮助
–version 显示版本信息

```
$echo -e "a\bdddd"
dddd

$echo -e "a\adddd" //输出同时会发出报警声音
adddd


$echo -e "a\ndddd" //自动换行
a
dddd


```
