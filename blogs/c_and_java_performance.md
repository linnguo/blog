##不同语言的执行效率
首先得承认这不是一个好例子，逻辑过于简单，受环境的干扰也特别大。不能作为评价一门语言综合效率的用例，仅仅是基于个人兴趣的小实验的记录。  

C语言版本1
	
	#include<stdio.h>
	int main(){
   		long a = 0;
   		for(long i=0; i<100000000; i++){
   			a += i;
   		}
   		printf("%ld\n", a);
   		return0;
	}


Java版本1

	public class T{
	   publicstaticvoid main(String args[]){
	       long a =0;
	       for(long i=0; i<100000000;++i){
	           a += i;
	       }
	       System.out.println(a);
	   }
	}
如以上代码所示，计算0到100000000的累加值，测试过程及结果如下

	gcc t.c
	time ./a.out
	4999999950000000
	real	0m0.237s
	user	0m0.234s
	sys	0m0.002s
	
	javac T.java
	time java T
	4999999950000000
	real	0m0.123s
	user	0m0.108s
	sys	0m0.020s
神奇的结果，以效率著称的C输给了Java，Java版本的用时大概是C版本的1/2；
不过以上的结果是在gcc未开启编译优化的情况下得出的，让我们看看开启优化后的情况；开启O2优化后的测试结果
	
	gcc -O2 t.c
	time ./a.out
	4999999950000000
	real	0m0.003s
	user	0m0.001s
	sys	0m0.002s
效果不可思议的好，都不能用提速多少倍来评价了，像作弊了一样；我们有必要看一看优化都做了什么，首先查看一下不开启优化时的汇编代码；
	
	LBB0_1:							## =>This Inner Loop Header: Depth=1
	cmpq	$100000000, -24(%rbp)  	## imm = 0x5F5E100
	jge	LBB0_4
	## BB#2:                     	##   in Loop: Header=BB0_1 Depth=1
	movq	-24(%rbp), %rax
	movq	-16(%rbp), %rcx
	addq	%rax, %rcx
	movq	%rcx, -16(%rbp)
	## BB#3:						##   in Loop: Header=BB0_1 Depth=1
	movq	-24(%rbp), %rax
	addq	$1, %rax
	movq	%rax, -24(%rbp)
	jmp	LBB0_1
代码很简单，循环的汇编版本，栈变量 -24(%rbp)与常量$100000000进行比较，大于等于时则跳出循环，否则进入循环进行累加，累加值放在-16(%rbp)另一个栈变量里;
然后是开启优化之后的汇编代码，只看关键部分
movabsq	$4999999950000000, %rsi ## imm = 0x11C37934E58F80
编译器直接把循环优化掉，算出了最终结果；这实在是很神奇，编译器优化是我的知识盲区；不过我怀疑，我们平时的业务代码中，可能能进行这么彻底的优化的用例不是那么多；常量循环累加可能太简单了，我们稍微把逻辑增加一些，让循环次数作为一个变量从外部获得；
C语言版本2

	#include<stdio.h>
	int main(int args,char* argv[]){
	   long a =0;
	   long x =0;
	   sscanf(argv[1],"%ld",&x);
	   for(long i=0; i<x; i++){
	       a += i;
	   }
	   printf("%ld\n", a);
	   return0;
	}
查看不优化时的汇编代码如下，

	LBB0_1:                                 ## =>This Inner Loop Header: Depth=1
	movq	-40(%rbp), %rax
	cmpq	-32(%rbp), %rax
	jge	LBB0_4
	## BB#2:                                ##   in Loop: Header=BB0_1 Depth=1
	movq	-40(%rbp), %rax
	movq	-24(%rbp), %rcx
	addq	%rax, %rcx
	movq	%rcx, -24(%rbp)
	## BB#3:                                ##   in Loop: Header=BB0_1 Depth=1
	movq	-40(%rbp), %rax
	addq	$1, %rax
	movq	%rax, -40(%rbp)
	jmp	LBB0_1
和常量循环的代码差不多，比较变量，进入循环累加或者跳出，设定同样的循环次数，和常量循环版本运行结果几乎一样；然后查看优化之后的汇编代码如下；

	## BB#1:                                ## %.lr.ph
	movl	$1, %ecx
	cmovgq	%rax, %rcx
	leaq	-1(%rcx), %rax
	leaq	-2(%rcx), %rdx
	mulq	%rdx
	shldq	$63, %rax, %rdx
	leaq	-1(%rcx,%rdx), %rbx
这段代码以我微薄的汇编水平理解起来非常的吃力，cmov代表“如果src大于dst则进行移动”，lea是使用内存寻址计算指令进行计算，shld是逻辑左移；用高级语言来表示上面这段代码的意义为 (x-1)*(x-2)/2 + x - 1，简化一下的话是x*(x-1)/2，计算之后的结果确实是从0累加到x-1的值；也是算法级别的优化，直接把循环变为同等意义的多项式；  
不过我还是觉得，一般的业务逻辑代码可能得不到这种级别的优化，有机会要在复杂度上做更多的实验，来验证这个问题；  
需要提出的问题1，是Java有没有类似的优化参数和过程？2，假设Java没有优化，为什么会比同样没有优化的C代码快？  
问题1需要等Java高手来解答，对于问题2，我听说过一个听起来很有道理的解释是这样的，C的编译器要保证编译出的指令在大部分CPU上正常运行，所以只能使用老的通用指令，而CPU不断发展，产生了很多新的速度更快的指令，jvm是有能力根据不同的平台选择最优指令的，所以同样的指令逻辑，Java非常有可能会更快一些；

