---
title: "Lab"
date: 2021-02-10T11:20:22+08:00
lastmod: 2021-02-10T11:20:22+08:00
draft: false
keywords: []
description: ""
tags: []
categories: []
author: "lttzz"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---



<!--more-->

# 二进制炸弹实验

[bomblab.tar](http://cs.nju.edu.cn/sufeng/course/mooc/0809NJU064_bomblab.tar)

执行程序，随意给定输入字符串（偷懒直接拿main.c了

``` bash
$ ./bomb bomb.c
Welcome to my fiendish little bomb. You have 7 phases with
which to blow yourself up. Have a nice day!

BOOM!!!
The bomb has blown up.

BOOM!!!
The bomb has blown up.

BOOM!!!
The bomb has blown up.

BOOM!!!
The bomb has blown up.

BOOM!!!
The bomb has blown up.

BOOM!!!
The bomb has blown up.

BOOM!!!
The bomb has blown up.
```

查看 `main.c`，关键函数 `phase_i` 的细节需要反汇编得出。

``` c
/***************************************************************************
 * Dr. Evil's Insidious Bomb, Version 1.1
 * Copyright 2011, Dr. Evil Incorporated. All rights reserved.
 *
 * LICENSE:
 *
 * Dr. Evil Incorporated (the PERPETRATOR) hereby grants you (the
 * VICTIM) explicit permission to use this bomb (the BOMB).  This is a
 * time limited license, which expires on the death of the VICTIM.
 * The PERPETRATOR takes no responsibility for damage, frustration,
 * insanity, bug-eyes, carpal-tunnel syndrome, loss of sleep, or other
 * harm to the VICTIM.  Unless the PERPETRATOR wants to take credit,
 * that is.  The VICTIM may not distribute this bomb source code to
 * any enemies of the PERPETRATOR.  No VICTIM may debug,
 * reverse-engineer, run "strings" on, decompile, decrypt, or use any
 * other technique to gain knowledge of and defuse the BOMB.  BOMB
 * proof clothing may not be worn when handling this program.  The
 * PERPETRATOR will not apologize for the PERPETRATOR's poor sense of
 * humor.  This license is null and void where the BOMB is prohibited
 * by law.
 ***************************************************************************/

#include <stdio.h>
#include <stdlib.h>
#include "support.h"
#include "phases.h"

/* 
 * Note to self: Remember to erase this file so my victims will have no
 * idea what is going on, and so they will all blow up in a
 * spectaculary fiendish explosion. -- Dr. Evil 
 */

FILE *infile;

int main(int argc, char *argv[])
{
    char *input;

    /* Note to self: remember to port this bomb to Windows and put a 
     * fantastic GUI on it. */

    /* When run with no arguments, the bomb reads its input lines 
     * from standard input. */
    if (argc == 1) {  
	infile = stdin;
    } 

    /* When run with one argument <file>, the bomb reads from <file> 
     * until EOF, and then switches to standard input. Thus, as you 
     * defuse each phase, you can add its defusing string to <file> and
     * avoid having to retype it. */
    else if (argc == 2) {
	if (!(infile = fopen(argv[1], "r"))) {
	    printf("%s: Error: Couldn't open %s\n", argv[0], argv[1]);
	    exit(8);
	}
    }

    /* You can't call the bomb with more than 1 command line argument. */
    else {
	printf("Usage: %s [<input_file>]\n", argv[0]);
	exit(8);
    }

    /* Do all sorts of secret stuff that makes the bomb harder to defuse. */
    initialize_bomb();

    printf("Welcome to my fiendish little bomb. You have 7 phases with\n");
    printf("which to blow yourself up. Have a nice day!\n");

    /* Warm up phase! */
    input = read_line();             /* Get input                   */
    if( phase_0(input) ) {           /* Run the phase               */
        phase_defused();             /* Drat!  They figured it out! */
        printf("Well done! You seem to have warmed up!\n");
	}

    /* Hmm...  Six phases must be more secure than one phase! */
    input = read_line();             /* Get input                   */
    if( phase_1(input) ) {           /* Run the phase               */
        phase_defused();             /* Drat!  They figured it out! Let me know how they did it. */
        printf("Phase 1 defused. How about the next one?\n");
	}

    /* The second phase is harder.  No one will ever figure out
     * how to defuse this... */
    input = read_line();
    if( phase_2(input) ) {
        phase_defused();
        printf("That's number 2.  Keep going!\n");
	}

    /* I guess this is too easy so far.  Some more complex code will
     * confuse people. */
    input = read_line();
    if( phase_3(input) ) {
        phase_defused();
        printf("Halfway there!\n");
	}

    /* Oh yeah?  Well, how good is your math?  Try on this saucy problem! */
    input = read_line();
    if( phase_4(input) ) {
        phase_defused();
        printf("So you got that one.  Try this one.\n");
	}
    
    /* Round and 'round in memory we go, where we stop, the bomb blows! */
    input = read_line();
    if( phase_5(input) ) {
        phase_defused();
        printf("Good work!  On to the next...\n");
	}

    /* This phase will never be used, since no one will get past the
     * earlier ones.  But just in case, make this one extra hard. */
    input = read_line();
    if( phase_6(input) ) {
        phase_defused();
	}

    /* Wow, they got it!  But isn't something... missing?  Perhaps
     * something they overlooked?  Mua ha ha ha ha! */
    
    return 0;
}
```



## 0x00 phase_0

``` c
72	/* Warm up phase! */
73	input = read_line();             /* Get input                   */
74	if( phase_0(input) ) {           /* Run the phase               */
75	    phase_defused();             /* Drat!  They figured it out! */
76	    printf("Well done! You seem to have warmed up!\n");
77	}
```

warm up phase，贴心。



``` assembly
360	08049454 <phase_0>:
361	 8049454:	55                   	push   %ebp  ; 保存旧的ebp
362	 8049455:	89 e5                	mov    %esp,%ebp
363	 8049457:	83 ec 08             	sub    $0x8,%esp
364	 804945a:	83 ec 08             	sub    $0x8,%esp  ; 调整栈帧
365	 804945d:	68 e0 a1 04 08       	push   $0x804a1e0  ; 参数2入栈
366	 8049462:	ff 75 08             	pushl  0x8(%ebp)   ; 参数1入栈
367	 8049465:	e8 f7 07 00 00       	call   8049c61 <strings_not_equal>
368	 804946a:	83 c4 10             	add    $0x10,%esp  ; 调整栈帧
369	 804946d:	85 c0                	test   %eax,%eax   ; 检测eax的值是否为0
370	 804946f:	74 0c                	je     804947d <phase_0+0x29>  ; 是0就跳转到47d处继续执行
371	 8049471:	e8 53 0a 00 00       	call   8049ec9 <explode_bomb>
372	 8049476:	b8 00 00 00 00       	mov    $0x0,%eax
373	 804947b:	eb 05                	jmp    8049482 <phase_0+0x2e>  ; 否则explode_bomb
374	 804947d:	b8 01 00 00 00       	mov    $0x1,%eax  ; 设置函数返回值为
375	 8049482:	c9                   	leave  
376	 8049483:	c3                   	ret 

; phase_0调用了strings_note_equal函数
; 易知ebp+8处存放的是main调用phase_0时传入的参数，也即输入值input

1018  08049c61 <strings_not_equal>:
1019 8049c61:	55                   	push   %ebp
1020 8049c62:	89 e5                	mov    %esp,%ebp  ; 调整栈帧
1021 8049c64:	53                   	push   %ebx  ; 压栈的方式保存被调用者保存寄存器的值
1022 8049c65:	83 ec 10             	sub    $0x10,%esp  ; 调整栈帧
1023 8049c68:	ff 75 08             	pushl  0x8(%ebp)   ; string_not_equal的参数1压栈,也即input
1024 8049c6b:	e8 c5 ff ff ff       	call   8049c35 <string_length>  ; 计算字符串长度
1025 8049c70:	83 c4 04             	add    $0x4,%esp
1026 8049c73:	89 c3                	mov    %eax,%ebx  ; 将参数1的程度保存到ebx
1027 8049c75:	ff 75 0c             	pushl  0xc(%ebp)  ; string_not_equal的参数2压栈
1028 8049c78:	e8 b8 ff ff ff       	call   8049c35 <string_length>
1029 8049c7d:	83 c4 04             	add    $0x4,%esp
1030 8049c80:	39 c3                	cmp    %eax,%ebx  ; 比较参数1和参数2的长度是否相同
1031 8049c82:	74 07                	je     8049c8b <strings_not_equal+0x2a>  ; 长度相同则到c8b处比较各字符是否相同
1032 8049c84:	b8 01 00 00 00       	mov    $0x1,%eax
1033 8049c89:	eb 3c                	jmp    8049cc7 <strings_not_equal+0x66>  ; 长度不同则直接返回1
1034 8049c8b:	8b 45 08             	mov    0x8(%ebp),%eax
1035 8049c8e:	89 45 f8             	mov    %eax,-0x8(%ebp)
1036 8049c91:	8b 45 0c             	mov    0xc(%ebp),%eax
1037 8049c94:	89 45 f4             	mov    %eax,-0xc(%ebp)  ; 将传入的字符串地址赋值给局部变量
1038 8049c97:	eb 1f                	jmp    8049cb8 <strings_not_equal+0x57>
1039 8049c99:	8b 45 f8             	mov    -0x8(%ebp),%eax
1040 8049c9c:	0f b6 10             	movzbl (%eax),%edx
1041 8049c9f:	8b 45 f4             	mov    -0xc(%ebp),%eax
1042 8049ca2:	0f b6 00             	movzbl (%eax),%eax
1043 8049ca5:	38 c2                	cmp    %al,%dl      ; 比较当前位字符
1044 8049ca7:	74 07                	je     8049cb0 <strings_not_equal+0x4f>  ; 相同跳转cb0比较下一位
1045 8049ca9:	b8 01 00 00 00       	mov    $0x1,%eax    ; 不同则中断比较返回1
1046 8049cae:	eb 17                	jmp    8049cc7 <strings_not_equal+0x66>
1047 8049cb0:	83 45 f8 01          	addl   $0x1,-0x8(%ebp)  ; 地址+1
1048 8049cb4:	83 45 f4 01          	addl   $0x1,-0xc(%ebp)  ; 地址+1
1049 8049cb8:	8b 45 f8             	mov    -0x8(%ebp),%eax
1050 8049cbb:	0f b6 00             	movzbl (%eax),%eax
1051 8049cbe:	84 c0                	test   %al,%al    ; 如果遇到EOF则比较结束，因为长度相同，只需要检测其中一个
1052 8049cc0:	75 d7                	jne    8049c99 <strings_not_equal+0x38>  ; 没遇到EOF继续比较
1053 8049cc2:	b8 00 00 00 00       	mov    $0x0,%eax
1054 8049cc7:	8b 5d fc             	mov    -0x4(%ebp),%ebx
1055 8049cca:	c9                   	leave  
1056 8049ccb:	c3                   	ret 
```

`phase_0` 将输入值和设定值传入`strings_not_equal`比较，相同返回1，不相同调用`explode_bomb`，具体的比较过程交给`strings_not_equal`处理。

用gdb动态调试找出设定好的字符串，`phase_0` 处下断，使用`run input.txt` 运行程序的同时给定命令行参数指定读入的文件。

![phase_0](/Picbed/2021_02/0210_00.png)

## 0x01 phase_1

``` c
67  /* Hmm...  Six phases must be more secure than one phase! */
68  input = read_line();             /* Get input                   */
69  if( phase_1(input) ) {           /* Run the phase               */
70      phase_defused();             /* Drat!  They figured it out! Let me know how they did it. */
71      printf("Phase 1 defused. How about the next one?\n");
72      }
```

查看反汇编代码，似乎仅仅是读入两个数，然后与设定的局部双精度浮点变量进行比较，比较的方式奇怪了些，不是一个双精度浮点数和一个整数在真值上比较，而是一个浮点数和两个整数在机器数上比较。

``` assembly
377  08049484 <phase_1>:                                                   
378   8049484:       55                      push   %ebp                
379   8049485:       89 e5                   mov    %esp,%ebp         
380   8049487:       83 ec 28                sub    $0x28,%esp  ; 调整栈帧              
381   804948a:       c7 45 f4 a1 84 76 09    movl   $0x97684a1,-0xc(%ebp)  
382   8049491:       db 45 f4                fildl  -0xc(%ebp)  ; int->double后压栈
383   8049494:       dd 5d e8                fstpl  -0x18(%ebp) ; double->double后出栈
384   8049497:       8d 45 e0                lea    -0x20(%ebp),%eax
385   804949a:       50                      push   %eax        ; sscanf的参数4入栈
386   804949b:       8d 45 e4                lea    -0x1c(%ebp),%eax   
387   804949e:       50                      push   %eax        ; sscanf的参数3入栈           
388   804949f:       68 0f a2 04 08          push   $0x804a20f  ; sscanf的参数2入栈
389   80494a4:       ff 75 08                pushl  0x8(%ebp)   ; sscanf的参数1入栈
390   80494a7:       e8 24 fc ff ff          call   80490d0 <__isoc99_sscanf@plt>  ; 调用sscanf
391   80494ac:       83 c4 10                add    $0x10,%esp	; 调整栈帧
392   80494af:       83 f8 02                cmp    $0x2,%eax   ; 测试sscanf是否成功读取2次
393   80494b2:       74 0c                   je     80494c0 <phase_1+0x3c> ; 是就跳到4c0处继续执行
394   80494b4:       e8 10 0a 00 00          call   8049ec9 <explode_bomb> ; 否则调用explode_bomb
395   80494b9:       b8 00 00 00 00          mov    $0x0,%eax
396   80494be:       eb 2c                   jmp    80494ec <phase_1+0x68> ; 返回0
397   80494c0:       8d 45 e8                lea    -0x18(%ebp),%eax
398   80494c3:       8b 10                   mov    (%eax),%edx
399   80494c5:       8b 45 e4                mov    -0x1c(%ebp),%eax
400   80494c8:       39 c2                   cmp    %eax,%edx    ; 比较ebp+0x18和ebp+0x1c处的值
401   80494ca:       75 0f                   jne    80494db <phase_1+0x57>
402   80494cc:       8d 45 e8                lea    -0x18(%ebp),%eax
403   80494cf:       83 c0 04                add    $0x4,%eax
404   80494d2:       8b 10                   mov    (%eax),%edx
405   80494d4:       8b 45 e0                mov    -0x20(%ebp),%eax
406   80494d7:       39 c2                   cmp    %eax,%edx    ; 比较ebp+0x14和ebp+0x20处的值
407   80494d9:       74 0c                   je     80494e7 <phase_1+0x63>
408   80494db:       e8 e9 09 00 00          call   8049ec9 <explode_bomb>
409   80494e0:       b8 00 00 00 00          mov    $0x0,%eax
410   80494e5:       eb 05                   jmp    80494ec <phase_1+0x68>
411   80494e7:       b8 01 00 00 00          mov    $0x1,%eax
412   80494ec:       c9                      leave   
413   80494ed:       c3                      ret
```

![phase_1](/Picbed/2021_02/0210_01.png)



# 计算机硬件系统设计

![封面](/Picbed/2022_01/0108_00.png)

## 课程导学与实验环境

## 数字逻辑基础实验

### 组合逻辑电路设计

真值表、卡诺图、逻辑表达式

#### 2路选择器（16位）

#### 16位无符号比较器

先设计4位的无符号比较器，然后级联出16位无符号比较器。

比较器要实现的是判断大小，可能的结果包括大于、等于和小于，等于比较每一位即可，而由于三种可能的结果是互斥的，故而大于或者小于实现其一即可，我实现的是大于。

相等的判断用异或很容易实现，大于就没有那么直观。真值表？不了，这么多一个个点头大。卡诺图？不了，我不熟。那就只剩下逻辑表达式了啊。

如何判断两个无符号数字的大小关系呢？让人来做的话，就是从高位一位一位的比较下去，同样可以用这个思路写出逻辑表达式，按规律写，别遗漏。

``` text
X3&!Y3 + !X3&!Y3&X2&!Y2 + !X3&!Y3&!X2&!Y2&X1&!Y1 + !X3&!Y3&!X2&!Y2&!X1&!Y1&X0&!Y0 + X3&Y3&X2&!Y2 + X3&Y3&X2&Y2&X1&!Y1 + X3&Y3&X2&Y2&X1&Y1&X0&!Y0
```

![](/Picbed/2022_01/0108_01.png)

16位的无符号比较器分成四组，如何将低一级的四位的比较输出连接到高一级的四位比较器上呢？又回到上面说的，四位比较器的三种可能结果是互斥的，因而此处低级比较器的输出情况只有三种，列一下可能的情况：

``` text
低一级输出 => 高一级输入
100 => 1 0
010 => 0 0 或者 1 1
001 => 0 1
```

从结果逆推关系，容易看出来，低一级输出的前两位和后两位分别做异或，分别对应高一级比较器的输入。

![](/Picbed/2022_01/0108_02.png)

![](/Picbed/2022_01/0108_03.png)

#### 码表数码管驱动设计

### 同步时序电路设计

### 小型数字系统设计

## 数据表示实验

## 运算器设计

## 存储器设计

## MIPS CPU设计

## MIPS指令流水线设计

## 单总线MIPS CPU设计

