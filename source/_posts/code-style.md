---
title: code style
date: 2016-09-12 11:44:51
 - study
tags:
 - code
---
**ZT**
每个程序员都有自己的编码风格，这基本上都是由他们的喜好决定的，此外，程序员还乐于争论各种编码风格的优劣，比如关于Tab和空格（见[《Tab v.s. 空格：一个永恒的神圣战争》](http://www.jwz.org/doc/tabs-vs-spaces.html)、[《空格“异教徒”去死》](http://www.codinghorror.com/blog/2009/04/death-to-the-space-infidels.html)）、80列规则（见[《保卫80列规则》](http://zuzu-curl.blogspot.com/2010/02/in-defense-of-80-column-rule.html)），还有大括号的缩进风格等。 

一致的编码风格，更便于阅读。因此程序员都想极力说服别人认同并使用与自己一致的编码风格。下面来了解一下我的编码风格变化历程吧，哪种编码风格是你的“菜”呢？ 

## 1980/1990年代的紧凑风格 

当我开始编程时，我使用tab字符，因为使用它保存的文件更小，还可以将代码限制到80列。我还遵循Kernighan和Ritchie的[《C Programming Language》](http://www.amazon.com/gp/product/0131103628/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&tag=hiltmon-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0131103628)一书中约定的“大括号放在同一行”规则，代码看起来是这样的： 
```c
int main(int argc, char *argv[]) {  
       int a = rand() % 100;  
       if (a > 25) {  
               call_a_function();  
               call_another_function();  
       } else {  
               call_b_function();  
       } // end if  
} // end main  
```
当时Tab缩进默认为8个字符宽度，这意味着你不能缩进过多，以免打破“80列”宽度的限制，这也迫使你写更少、更精简的函数名或代码（这是好事）。大括号在同一行上，意味着可以在一页中显示更多的代码。 

但也有一些不好的事情。我们开始使用更短的变量名，这使代码更难以阅读和维护。我们在代码行之间没有添加任何空白行，结果很难找到代码块开始和结束标记（因此我们必须在代码块结束的地方加上注释）。由于缩进太宽，留给我们的编码空间就少了。 

## 2000年的宽松风格

在2000年，我以及周围的程序员基本上都已经切换到了微软的C/C#约定上，即使用4个空格，而不是Tab，不限制代码行的长度，大括号单独一行。如下：
```c
int main(int argc, char *argv[])  
{  
   int a = rand() % 100;  
   
   if (a > 25)  
   {  
       call_a_function();  
       call_another_function();  
   }  
   else  
   {  
       call_b_function();  
   }  
}  
```
空格缩进意味着我们不必处理不同编辑器中Tab大小（Tab大小可以根据用户喜好进行设置）。我们使用空行，以使代码更易于阅读。单独一行的大括号使代码块更加明显（可以很快找出开始和结束点）。 

但是，这也导致了一些问题。由于行长度（大屏幕）无限制，我们不再感觉必须要限制缩进的数量，并开始创建更大更长的代码。由于当时的编辑器无法很好地格式化代码，程序员经常会在水平滚动时无法看到左侧的代码。代码变得难以阅读，并排比较更困难。 

因此，在2000年初，我选择使用4个空格来缩进，依然遵循80列规则，将大括号单独放在一行中，以改善可读性。 

## 2010年的典型风格 

现在，我开始使用Xcode 4。Xcode中默认为4个空格缩进，保持不变，还可以智能格式化代码，不会出现滚动到右边看不到左边代码的情况。但是，令我郁闷的是，默认代码片段中大括号竟然不一致，有时单独一行，有时和代码共用一行。 

看看默认的initWithNibName代码片段：
```c
- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil  
{  
   self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];  
   if (self) {  
       self.title = NSLocalizedString(@"Detail", @"Detail");  
   }  
   return self;  
}  
```
事实证明，苹果没有弄错。它使用了正确的Kernighan & Ritchie风格，在早期C语言书籍中是为了节省空间才将所有大括号放在代码后面的。根据维基百科，K&R风格是： 
> 当秉承K＆R时，每一个函数左括号都应在下一行中，并与开头行有相同的缩进。括号中的语句应该缩进，右括号应该与函数开头有相同的缩进，并单独一行。对于代码块内的控制语句，左括号应与控制语句在同一行中，右括号单独一行（除非有else或while关键字）。

事实上，这是典型的K&R风格，针对所有的C衍生语言，包括C++、C#和Objective-C。这是语言本身的设计者所遵循的风格。

因此，我也改变了我的括号使用风格，我仍然使用空格缩进，仍然遵循80列宽度原则，但现在更多依赖于编辑器来进行代码格式化。不过，我现在坚持真正的K＆R括号风格。

```c
int main(int argc, char *argv[])  
{  
   int a = rand() % 100;  
       
   if (a > 25) {  
       call_a_function();  
       call_another_function();  
   } else {  
       call_b_function();  
   }  
}  
```

英文原文：[Reprogramming My Brace Style Mind](http://hiltmon.com/blog/2013/01/07/reprogramming-my-brace-style-mind/)
