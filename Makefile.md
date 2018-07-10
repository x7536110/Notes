# Makefile学习笔记

## 什么是Makefile

- Makefile是一种配置文件，文件中定义了一些列的规定，指定项目文件的编译顺序、编译参数、以及更复杂的功能操作。makefile也可以执行操作系统的命令。 

- make是一个命令工具，它解释并执行makefil中的指令（规则）。


## Makefile的编写规则
```bash
target...:prerequisites ...
command
...
```
或者
```bash
target...:prerequisites; command ...
command
...
`target`是一个目标文件，可以是`Object File`,也可以是执行文件，还可以是一个标签(`Label`)

`prerequisites`就是，要生成那个`target`所需要的文件或是目标。

`command`也就是make需要执行的命令。（任意的Shell命令）

这是一个文件的依赖关系，也就是说，`target`这一个或多个的目标文件依赖于`prerequisites`中的文件，其生成规则定义在`command`中。

说白一点就是说，`prerequisites`中如果有*一个以上*的文件比`target`文件*要新*的话，`command`所定义的命令就会被执行。这就是Makefile的规则。也就是Makefile中最核心的内容。

### 一个makefile文件的例子
```bash
edit :  main.o kbd.o command.o display.o \        
        insert.o search.o files.o utils.o
        # \的作用是连接上下两行
    cc -o edit main.o kbd.o command.o \
        display.o insert.o search.o files.o utils.o
# 这句话的意思是生成可执行文件edit,中间目标文件为main.o/kbd.o/... 
# 如果中间文件(*.o)比输出文件(edit)更新，那么执行命令cc -o ...(编译并连接...)
main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
# 每个.o文件也有自己独立的一组依赖文件
clean :
    rm edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o
# clean命令，执行make clean后执行的命令
```
在该makefile文件中，make程序会逐层寻找依赖关系，存在以下几种情况：
- 假设工程未编译过，首先找不到edit文件，使用*.o文件编译链接生产edit文件；此时找不到.o文件，make继续深入根据依赖描述编译每一个.o文件，最后再次编译edit文件
- 假设工程已编译过，但是某个文件更新了，make会编译更新该文件对应的.o文件，再链接更新依赖该.o文件的文件

### makefile中变量的使用
和.sh一样，makefile也可以使用变量(宏)来简化编写流程，比如刚才的这段语句：
```bash
edit :  main.o kbd.o command.o display.o \        
        insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o \
        display.o insert.o search.o files.o utils.o
```
我们可以发现，从main.o开始到utils.o为止这段子串出现了两次，因此我们可以定义一个宏
```bash
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o
```
然后这样使用
```bash
edit : $(objects)
    cc -o edit $(objects)
......
clean : 
    rm edit $(objects)
```
是不是方便了很多？
### make的自动推导
GNU make在发现一个.o文件后，会自动的把同名的.c文件添加在依赖关系中，并且自动推导对应的cc命令，因此可以实现如下的简化：
```bash
utils.o : utils.c defs.h
    cc -c utils.c
#########################
utils.o : defs.h #utils.c
#   cc -c utils.c
# #号标注的为自动推导后的内容
```
因此，前面的这个复杂的makefile可以简化为以下形式：
```
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o
edit : $(objects)
    cc -o edit $(objects)
    main.o : defs.h
    kbd.o : defs.h command.h
    command.o : defs.h command.h
    display.o : defs.h buffer.h
    insert.o : defs.h buffer.h
    search.o : defs.h buffer.h
    files.o : defs.h buffer.h command.h
    utils.o : defs.h
clean :
    rm edit $(objects)
```
因为make的用处主要是描述目标文件与依赖，所以当依赖比较复杂当时候，不光可以采用 *一个目标文件:一堆依赖* 的形式进行描述，还可以采用 *多个目标文件：共同依赖* 的形式进行描述  
依然是拿该make文件举例
```bash
command.o : defs.h command.h
display.o : defs.h buffer.h
search.o : defs.h buffer.h
###########################
command.o display.o : defs.h
command.o : command.h
display.o search.o : buffer.h
```
两种方式，见仁见智


### 关于clean指令的一些注意事项
推荐使用.PHONY修饰clean指令 
.PHONY的意思是“伪目标”，这样做的目的主要是防止工程目录下存在一个同名的clean文件，导致makefile中的clean命令无法执行。 
```bash
clean : rm edit $(objects)
###############
.PHONY : clean
clean : -rm edit $(objects)
```

### makefile的相互引用

makefile文件中，可以使用include关键字引用其他的makefile文件，语法是`include<MakeFileName>`，include关键字之前不允许有tab字符。  
如果在当前目录下没有找到include引用的文件，那么make程序会去系统的全局include目录下寻找(/usr/include/)，如果make执行时附加了-I/--include-dir参数，也会去该参数对应的目录下寻找。

### makefile的shell支持

makefile作为一个批处理，自然也是支持UNIX的标准shell的，这对简化我们的Makefile文件编写有重要意义。  
首先我们可以使用通配符来定义一系列比较相似的文件，make支持三个通配符：`'*','?','[]'`，如果我们的文件名中有通配符符号，可以使用转义字符来描述，规则同大部分语言，都是用`\`来转义  

### VPATH-文件自动搜索
在一些大工程中，有大量的源文件，我们通常会使用分目录的方式对源文件进行分类。所以，当make需要寻找文件的依赖关系时，可以在文件前加上路径，也可以把一个路径告诉make，让make自动去寻找。  
Makefile中的特殊变量“VPATH”就是完成这个功能的。如果没有指定这个变量，make只会在当前目录中去寻找依赖文件和目标文件。如果定义了这个变量，那么，make就会在当前目录找不到的情况下，去指定目录寻找文件。
```bash
VPATH = src:../headers:../include
```
上面语句定义了三个目录，分别是"src","../headers","../include"，当前目录如果没有搜索到目标文件，就会去以上三个目录继续搜索，目录之间使用冒号":"分割。  

另一种方式是使用vpath（全小写）关键字，指定搜索策略。其使用方法如下：
```bash
vpath < pattern> < directories> 
# 符合模式<pattern>的文件指定搜索目录<directories>。
vpath < pattern>
# 清除符合模式<pattern>的文件的搜索目录。
vpath
# 清除所有已被设置好了的文件搜索目录。
```
例如`vpath %.h ../headers`的意思就是，所有[.h]文件都去../headers目录下进行搜索。

### 伪目标的详解
先前的例子中，我们提到了.PHONY关键字，我们使用该关键字修饰了clean关键字，其作用是防止存在同名文件clean导致make clean无法执行。
这是什么原理呢？其实clean也是一个目标，但是这个目标文件没有依赖`prerequisites`，只有`command`，所以执行make clean的时候就会直接执行`command`里的命令。但是如果该目录下存在一个clean的文件，因为我们没有指定`prerequisites`，所以该文件一定是最新的，根据make的原则，clean就不会被生成，也就不会执行后续的`command`指令。而.PHONY关键字的意思是，不管当前目录是否存在一个叫做clean的文件（目标），都会执行后续的`command`指令。    
这个特性有很多用法，一个例子就是，假设我们一个Makefile文件想生存多个可执行文件，并且所有的目标文件都写在一个Makefile中，那么我们就可以使用伪目标的特性  
例子：
```bash
all : prog1 prog2 prog3
.PHONY : all
prog1 : prog1.o utils.o
    cc -o prog1 prog1.o utils.o
prog2 : prog2.o
    cc -o prog2 prog2.o
prog3 : prog3.o sort.o utils.o
    cc -o prog3 prog3.o sort.o utils.o
```
我们想同时生成prog1 prog2 prog3这三个可执行文件，就可以先把这三个文件作为依赖，生成一个叫做all的文件，之后再把all用.PHONY修饰，所以并没有真的生成all这个可执行文件，只是生成prog1 prog2 prog3这三个可执行文件。  
从这个角度来看，.PHONY也有点批处理程序的意思呢…
### 多目标
待续……