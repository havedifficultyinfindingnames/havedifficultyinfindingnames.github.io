---
categories: ["C/C++", "持续更新"]
tags: ["gcc", "ld", "linker script", "链接器", "编译原理"]
---

GNU Linker Script描述了链接器时的行为。顾名思义，这个脚本便控制是一堆`.o`文件合成一个`.o`的过程。记住这一点就可以理解其中出现很多莫名其妙的，仿佛未定义的变量。

原始参考：

- <https://github.com/iDalink/ld-linker-script>
- <https://allthingsembedded.com/post/2020-04-11-mastering-the-gnu-linker-script>
- <https://www.cnblogs.com/HiDark/p/18166817>

# object文件组成

这一章与脚本本身无关，主要是提醒要手写这个脚本，就必须首先知道程序由什么段组成。文件组成和架构相关，并不是统一的。一般而言，就是老生常谈的`text`、`rodata`、`data`、`bss`段。但是比如STM32，就额外需要`isr_vector`段（放在地址`0`的，所以不和`text`段一起），就需要提前明确。

# 声明与指令

脚本由一个个块组成。所有块列举如下：

- `SECTIONS`
- `MEMORY`
- `PHDRS`
- `VERSION`

脚本中可以用在全局的指令如下：

- `ENTRY(symbol)` 指定入口点
- `INCLUDE filename` 包含别的脚本
- `INPUT(filename, ...)` 指定输入文件
- `OUTPUT(filename)` 指定输出文件
- `SEARCH_DIR(path)` 指定搜索路径
- `STARTUP(filename)` 指定文件为第一个文件
- `OUTPUT_FORMAT(format)` 指定输出格式
- `TARGET(format)` 指定输入格式
- `REGION_ALIAS(alias, region)` 指定别名
- `ASSERT(boolean, message)` 断言
- `EXTERN(symbol, ...)` 声明符号
- `PROVIDE` 定义弱符号
- `HIDDEN` 定义不导出符号
- `PROVIDE_HIDDEN` 定义弱且不导出符号

# 符号

脚本中可以定义符号（即`goto`使用的那个），默认为全局符号。同label一样，符号也不需要声明即可使用；而同label不同的是，符号更像一个`void*`类型的变量，可以运算与赋值。这个符号同样可以被外部引用，例如在C语言中可以通过`extern void* identifier;`声明并引用它。与之相关的有两个指令可以定义变量：

- `PROVIDE` 若符号已经存在，则使用已定义的；否则才生成新符号。
- `HIDDEN` 不导出符号。注意不是隐藏符号，符号依然在链接脚本相关的输入文件中全局可见，只是不生成在输出文件中。
- `PROVIDE_HIDDEN` PROVIDE且HIDDEN

# SECTION指令

``` 
SECTION [address] [(type)] :
  [AT(lma)]
  [ALIGN(section_align) | ALIGN_WITH_INPUT]
  [SUBALIGN(subsection_align)]
  [constraint]
  {
    output-section-command
    output-section-command
    …
  } [>region] [AT>lma_region] [:phdr :phdr …] [=fillexp] [,]
```

未完待续

# MEMORY指令

```
MEMORY
  {
    name [(attr)] : ORIGIN = origin, LENGTH = len
    …
  }
```

`attr`有如下可选：

- `r` 读
- `w` 写
- `x` 可执行
- `a` 可分配
- `i` 初始
- `!` 取反（对后一个生效还是对后面都生效？待验证）

