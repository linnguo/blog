##Graphviz简介
graphviz是一款图形软件，与一般的"所见即所得"的普通画图工具主要使用鼠标拖拽不同，graphviz使用一门名为dot的语言用来描述图表，用户使用dot写脚本，graphviz根据脚本自动布局生成图表。graphviz将这种方式称为"所思即所得"。

使用这种方式有几个好处，一个是将用户从排版中解放出来，由工具自动处理这个过程，用户不必再关心如何布局，修改添加删除节点的时候也不用再对整个图的排版布局重新进行人工的整理；另一个好处是某些复杂的情况，比如代码的类图和调用图，dot脚本可以使用其他工具自动生成。

graphviz提供若干的布局器，dot(这个指的是布局命令,上一个dot指的是脚本名字)是默认的布局方式，主要用于画有向图；还有其他的布局方式可以参考man dot；
##HelloWorld
dot的语法非常简单，一个最简单的有向图可以像下面这样

	digraph g{
   		a;
   		b;
   		a -> b;
	}
	
digraph是关键字，代表有向图，g是图的名字，a和b是仅有的两个节点，a指向b;将这段代码保存为文件g.gv，使用如下命令生成图片；
	
	dot g.gv -Tpng -o g.png
![](graphviz_intro/g.png)
	
##﻿Dot语法
	[strict] (graph|digraph) name { statement-list }
以上是一个顶级图的定义方式。如果使用了strict关键字，指明一个图是"严格的"，那么将禁止在同一对节点之间定义多条边。如果使用digraph关键字，表明这是一个有向图，连线符必须为->；如果使用graph关键字，表明这是一个无向图，连线符必须为--；

语句有以下几种形式:

	name0=val0;		//设置样式属性
	node[name=val];	//设置节点的样式属性
	edge[name=val];	//设置边的样式属性

	//创建一个节点n0，并设置它的各种属性
	n0[name0=val0, name1=val1, ...]; 
	
	//创建一条边连接节点n0和n1，并设置它的各种属性
	n0 edgeop n1[name0=val0, name1=val1, ...]; 
	
	//创建一个子图，子图的名称必须以cluster开头
	[subgraph name] { statement-list } 

节点的常用属性及值如下一般有

- 形状(shape):record, box, oval, ellipse, rect ...
- 边框颜色(color) 填充颜色(fillcolor):
- 样式(style):solid, dashed, dotted, bold, filled, rounded ...
- 标签(label)

一个更复杂的例子，一个简单的站桩对战角色的动画状态图
	
	digraph{
		rankdir=LR;
		node[shape="record"]
		
		入场[style = "filled", fillcolor = "green"];
		预备;
		受伤;
		攻击;
		死亡;
		
	
		入场 -> 预备;
	
		预备 -> 受伤;
		受伤 -> 预备;
	
		预备 -> 前摇;
		后摇 -> 预备;
	
		预备 -> 死亡;
		受伤 -> 死亡;
	
		subgraph cluster_attack{
			bgcolor = "grey";
			前摇 -> 攻击 -> 后摇;
		}
	}
![](graphviz_intro/anim.png)

当节点的shape属性为record时，可以在label中使用分隔符‘|’来分割这个节点；使用{name0|name1}的形式来纵向分割节点;
还可以在label中定义表格，使用的语法和html类似。更多的例子可以访问
[http://www.graphviz.org/content/node-shapes#html](http://www.graphviz.org/content/node-shapes#html)

##局限
graphviz的好处是把用户从排版中解放出来，但是相应的，也带来了“不能确定每个结点的位置”的局限性，比如时序图，使用graphviz就很难表达(非常可能是我不会)。