# bomblab实验报告

先观察给的.c文件，发现是要输入六组语句并且判断正误的，并且很容易发现判断函数phase_i(i=1,2…6)要有一个错误，炸弹即爆炸，那我们就要用到gdb调试了,在终端输入``gdb bomb``进入调试

## phase_1

首先在phase_1处断点（命令：`b phase_1`）找到phase_1的拆弹语句。然后`r`运行，先随便输入点东西这里我输入了很多a,断在该处之后单步调试，因为要进入函数内部，我们用命令`step`或者`s`进行单步调试。调试发现一个`strings_not_equal`函数，跟进去看发现寄存器`rdi`为我们输入的很多个a，而寄存器`rsi`已经有了一句话。

![bomblab_phase_1_1.png](https://i.loli.net/2021/04/25/dVN392JXrMQjDwu.png)

那么能直接断定这个就是我们的拆弹语句，重新调试进去输入那个语句`Border relations with Canada have never been better.`发现成功拆掉了这个炸弹，那么phase_1就拆掉了。

## phase_2

拆完了之后就看向phase_2，我们先`delete`	清除所有断点然后`b phase_2`新增断点，`r`之后先输入之前的拆弹语句来到phase_2，我们照例输入很多的a，`s`单步调试进去发现有一个函数叫`read_six_numbers`，跟进去发现

![bomblab_phase_2_1.png](https://i.loli.net/2021/04/25/J7O8Yjl9kKnC1SN.png)

调用了`scanf`函数并且能看到参数`%d %d %d %d %d %d`，那无疑我们这次应该输入六个整数。那就先猜，就先输入6个0也罢，但此时我们不能’s’单步调试了，因为`scanf`内部构造很复杂，`s`单步调试会把你键盘按烂的。因此我们用`next`或`n`单步调试，跳过了`scanf`函数的内部，成功给了6个整数，然后继续调试，发现

![bomblab_phase_2_2.png](https://i.loli.net/2021/04/25/Kil6fnbgrRJH4hm.png)

`ptr[rsp]=1`才能跳转避免执行到`explode`函数，我们稍微调试一下也可以发现`ptr[rsp]`存了第一个输入的数值，那么就可以很容易得到第一个整数是`1`，我们把第一个值改成1，其它的照常不变，发现第一个数躲过了爆炸，那么说明我们的方案可行，接下来只需要把剩下五个数判断完了就可以了。继续`n`单步调试，发现第二个测试的数是

![bomblab_phase_2_3.png](https://i.loli.net/2021/04/25/HRzX9CYaBQLN1Sc.png)

一样的是比较`eax`上面我们可以看到有一个

```assembly
add eax,eax
```

然后再比较，那么第二个数不就应该是2了吗？，虽然我也不知道为什么`ptr[rbx]`它就是第二个数，但是稍微想想也知道肯定是依次对你的输入去判断的，所以第二个数是2了。同理，他每次都加上自己，那么每次输入的数就是前面数的两倍，那清晰了，答案应该就是`1 2 4 8 16 32`。清除断点输入后发现没有爆炸，那么phase_2也拆弹成功了。

## phase_3

然后断点下在phase_3，接着运行到那边。运行到scanf发现格式化字符串为”%d %d”，那就是两个整数，并且看到后面有一个`cmp eax,1`，并且要求`eax>1`，否则就执行爆炸函数了，`eax`在`scanf`之后获得了`scanf`函数的返回值，scanf的返回值就是输入数的个数。那我们就随便输入俩数看看。

![bomblab_phase_3_1.png](https://i.loli.net/2021/04/25/JPg3Rp4u7Tjr6If.png)

继续调试发现，如果`ptr[rsp+8]`大于`7`会发生跳转不妨先输入大于`7`的值看看会跳转到哪儿，输入之后，好的，成功爆炸，躲不掉的那种哦。

![bomblab_phase_3_2.png](https://i.loli.net/2021/04/25/etqlSa591MrFCVp.png)

那看来我们第一个数只能输入`0~7`之内的值，我们先输入`0 0`来看看，继续单步调试发现

![bomblab_phase_3_3.png](https://i.loli.net/2021/04/25/mz7utwg8CNbLYPU.png)

那说明我们应该输入`0 207`，因为`eax`被赋值了`0xcf`，然后又比较中也含有`eax`重新来一遍发现竟然过了，直接能进入到phase_4的那种，那么你就会思考，`1~7`会发生什么，据测试，每个数对应了一个整数，你可以理解为有一个函数`f(x)x∈[0,7]∩Z`然后你必须正确输入`x f(x)`的其中一个对应。那你可能还会想，负数有没有对应，其实我也试过，负数直接就不行了，因为`jg`指令是判断无符号数的，负数就会被看成一个很大的正整数，那么你还是不可避免的爆炸了。

## phase_4

在`phase_4`处断点，依次输入前三条拆弹语句，第四句老规矩输入很多`a`，`s`单步调试进入`scanf`，发现语句依然是`%d %d`，好嘛，又是两个整型，那重来，先`0 0`，`n`单步调试进去发现有一个语句`cmp eax,2 jne`，`jne`为`jump not equal`这个判断也很简单，就是看`scanf`有没有输入2个数，它都是`%d %d`了，肯定你只能输入两个数啊，不是两个就爆炸了(~~Ps:别问我为什么这么肯定的~~)。

![bomblab_phase_4_1.png](https://i.loli.net/2021/04/25/fTMdxrpsWg1ocXE.png)

输入两个`0`发现莫名其妙过了，其实我想就这么水过去的，但是还得去分析的。

`disassemble func4`查看一下它那个函数的汇编代码。

```assembly
Dump of assembler code for function func4:
   0x08048d0b <+0>:	sub    $0x1c,%esp
   0x08048d0e <+3>:	mov    %ebx,0x14(%esp)
   0x08048d12 <+7>:	mov    %esi,0x18(%esp)
   0x08048d16 <+11>:	mov    0x20(%esp),%eax
   0x08048d1a <+15>:	mov    0x24(%esp),%edx
   0x08048d1e <+19>:	mov    0x28(%esp),%esi 
   0x08048d22 <+23>:	mov    %esi,%ecx
   0x08048d24 <+25>:	sub    %edx,%ecx       
   0x08048d26 <+27>:	mov    %ecx,%ebx
   0x08048d28 <+29>:	shr    $0x1f,%ebx    
   0x08048d2b <+32>:	add    %ebx,%ecx     
   0x08048d2d <+34>:	sar    %ecx          
   0x08048d2f <+36>:	lea    (%ecx,%edx,1),%ebx    
   0x08048d32 <+39>:	cmp    %eax,%ebx
   0x08048d34 <+41>:	jle    0x8048d4d <func4+66>
   0x08048d36 <+43>:	lea    -0x1(%ebx),%ecx
   0x08048d39 <+46>:	mov    %ecx,0x8(%esp)
   0x08048d3d <+50>:	mov    %edx,0x4(%esp)
   0x08048d41 <+54>:	mov    %eax,(%esp)
   0x08048d44 <+57>:	call   0x8048d0b <func4>
   0x08048d49 <+62>:	add    %eax,%ebx   
   0x08048d4b <+64>:	jmp    0x8048d66 <func4+91>
   0x08048d4d <+66>:	cmp    %eax,%ebx  
   0x08048d4f <+68>:	jge    0x8048d66 <func4+91>
   0x08048d51 <+70>:	mov    %esi,0x8(%esp)
   0x08048d55 <+74>:	lea    0x1(%ebx),%edx 
   0x08048d58 <+77>:	mov    %edx,0x4(%esp)
   0x08048d5c <+81>:	mov    %eax,(%esp)
   0x08048d5f <+84>:	call   0x8048d0b <func4>
   0x08048d64 <+89>:	add    %eax,%ebx  
   0x08048d66 <+91>:	mov    %ebx,%eax   
   0x08048d68 <+93>:	mov    0x14(%esp),%ebx
   0x08048d6c <+97>:	mov    0x18(%esp),%esi
   0x08048d70 <+101>:	add    $0x1c,%esp
   0x08048d73 <+104>:	ret    
End of assembler dump.
```

这个函数先放在这我们先看后面有哪些条件

```assembly
test   eax, eax
jne    phase_4+76 <phase_4+76> 
cmp    dword ptr [rsp + 0xc], 0
je     phase_4+81 <phase_4+81>

```

第一次是跳转会爆炸，第二次是不跳转会爆，那么换言之，两次都必须等于，那么第一次的`test eax,eax`它干什么了呢？先想一想它怎么判断两个数相等，两个数相等当且仅当它们之差为0时成立，字符串也同理。那么换言之，它就判断`eax`的值是不是`0`而已相等为`0`，不相等则不为`0`。那么很清楚了，防止它跳转，我们要保证`eax`寄存器值为0。下面就是判断第二个输入的值是否为0了，为0跳转。

![bomblab_phase_4_2.png](https://i.loli.net/2021/04/25/usg2zr6lbFYjwhE.png)

实际上，这里的`ptr[rsp+0xc]`就是我们输入第二个数的低八位。因此第二个数只能输入`0`因为第一次比较用到了寄存器比较，我们也不知道运行这个函数之后函数的返回值是多少(`rax`保存函数返回值)，只能去调试看看。因此这个答案是`0 0`

## phase_5

依然先断点，输入之前四句拆弹语句。到这里之后随便输入点东西，发现了

![bomblab_phase_5_1.png](https://i.loli.net/2021/04/25/sQSBH3YFexy6dNJ.png)

这回是要输入一串字符串，而且还有长度检测，不等于6直接爆炸qwq。

那我们先随便输入一个`aaaaaa`看看情况

![bomblab_phase_5_2.png](https://i.loli.net/2021/04/25/JaWfPYRIysXc9qU.png)

很明显，有一个循环，以`eax`为循环变量，依次对输入的字符进行一系列的操作。具体操作是：先对字符`and 0xf`然后把结果保存在`rdx`里面，返回的字符是`0x4024b0+rdx`,最后这个保存到`rsp+rax+0x10`里面，在往栈底偏移`0x10`的地方起一个保存好的字符串。接下来又要怎么操作呢？接着单步调试看看：

![bomblab_phase_5_3.png](https://i.loli.net/2021/04/25/SiLjU51Z3BroAk9.png)

发现在`rsp+0x10`那个位置的字符串要被`"flyers"`字符串比较，相等跳转，不跳转就炸了，那么唯一没有看的就是刚刚那个`0x4024b0+rdx`到底是什么了。但是可以猜测这应该是一个字符串，然后`rdx`做偏移取字符串的下标对应的字符。`print (char *)0x4024b0`查看这个字符串发现输出了一个很长的东西`maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?`因为可以看到它的偏移被`and 0xf`可以很容易证明这个偏移肯定小于等于`0xf`，那么我们取出16位的字符串`maduiersnfotvbyl`,`flyers`的话，它的偏移可以是：`9 15 14 5 6 7`这里建议大家写脚本跑一下，自己硬算也可以的。为了防止字符无效，我们尽量取满可能，因为偏移是固定的，但是高四位不管是什么都是可以的，反正最后要被`and 0xf`

![bomblab_phase_5_4.png](https://i.loli.net/2021/04/25/EWxXJ2PuAQ3HoIc.png)

因为每个字符都是相互独立的，所以你可以在这六行任意取一个可读字符最后拼接成字符串。因为有些字符不可编辑，所以采取这种措施是最妙的。

## phase_6

待更新

因为作者目前比较菜，phase_6和secret_bomb都不会做，如果上面有哪里说的不对的恳请指正。