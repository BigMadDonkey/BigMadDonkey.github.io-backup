---
title: MathJax公式解析器的Tips
author: MadDonkey
layout: post
categories: [Tips]
toc: true
---

Jekyll自带的公式解析器MathJax（好像是我自己调的？whatever）有的时候反应很慢，这倒还都能接受；可有的时候渲染出来的数学公式和本地Typora渲染的效果完全不同。本文章记录使用Jekyll过程中碰到的公式细节。

1. 块级公式（前后各一组`$$`的）要想成功渲染，最好在前面的`$$`之前空一行。

2. 无论是行级公式还是块级公式，"\|"前必须加转义符，否则会被解释成一大块意义不明的空白。

3. 行级公式在Typora上可以正确地分行。在JMathJax上，还是用块级公式吧。

4. 有时候即使等了很久，公式还是没有渲染出来，还少了些东西。这时注意是否是相邻的\*或\~距离太近影响了MathJax的判断。星号的使用可以使用`\star`，浪线的使用可以可以使用`\sim`。

<hr >

下面是一些Typora上数学公式可以识别而Jekyll的公式解析器无法识别的符号：

1. $A^\infin$`\infin` 正常应该显示无穷符号。没办法，使用`\infty`$A^\infty$代替吧。
2. 

