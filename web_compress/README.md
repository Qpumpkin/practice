### web代码压缩　　

**整体思路**    
　　使用网上的压缩工具压缩js和css文件，自己编写简单的html和json压缩工具压缩html、json文件，然后集成到Makefile  

---  
#### js、css压缩  

　　直接用现成的工具压缩，没什么好说的。 
　　注意点：在用yuicompressor压缩一个a9的js手机页面时java执行报错，yuicompressor开源在github上，issue有人提这个问题，但没有得到修复。因此在yuicompressor压缩不了的情况换用jsmin来压缩js代码。   

#### html压缩  

　　html要实现完美压缩，实现很复杂，以至于网上都找不到相应工具了。自己写的压缩很简单，去除空格、tab、换行，这里面也存在**隐藏bug**，这是风险点。  
* 实现的功能：  
　　1、单个文件压缩  
　　2、只压缩空格、tab、换行符　　
　
* 匹配规则：  
　　">string<",去掉右尖括号和左尖括号之间只包含空格、tab、换行符的内容  
  
* 思路：  
　　把文件映射到内存，从映射的内存一个字节一个字节读取数据，如果不是'>',则写入压缩后的文件.
    如果是'>'，记录当前指针，往后读取查找'<'，记录指针。匹配这段字符是否只有空格\tab\换行。
    如果是则在文件写入'<'，否则全部内容写入。  

#### json翻译包压缩  

>其实json的压缩空间很小，一行可以少3个字符，对于1000行的翻译包，可以减少3k的文件大小。体现到flash上，大概只有零点几K了。而A9的翻译包才100多行。   

json语言包压缩注意事项：  
* 从json格式看，只要不处理双引号里面的内容就不会破坏json数据  
* 注意双引号里面可能有转义字符的双引号  



#### 集成到Makefile  

　　压缩工具都整理好了，就是如何集成到Makefile的问题了。上面的压缩工具基本都是基于单个文件的压缩，因此对于整个web目录的压缩用一个脚本来实现。  
　　脚本遍历目录 >> 判断文件格式 >> 压缩文件 >> 保存到新目录 >> 编译时指定新目录  

*makefile修改点*  
![makefile修改点](pic/20161017_Makefile修改.png)   
  
#### 压缩效果  

　　使用BT A9的代码测试了一下，压缩页面代码后的**升级文件小了12k**，页面显示暂没发现问题。  
　　对比web组用的压缩工具，发现压缩后大小差异在于js文件压缩后的大小  
　　![与web组的压缩效果对比](pic/20161017_压缩效果对比.png)  

* 运行时间  
　　具体运行时可以观察到，压缩整个web代码会用10来秒。  

```shell
root@ubuntu:http# /usr/bin/time ./compress_web.sh wifi compress_tools
#####################################
####### compress web...##############
####### compress done ###############
#####################################
8.68user 3.45system 0:12.52elapsed 96%CPU (0avgtext+0avgdata 37948maxresident)k
0inputs+3712outputs (0major+303997minor)pagefaults 0swaps
```
　　用时这么长，主要是压缩js和css使用java程序，每次java启动很缓慢导致时间很长，压缩html的C代码和shell脚本是执行很快的。  

---



　
