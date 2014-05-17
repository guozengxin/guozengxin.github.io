---
layout: post
title: "xpath路径表达式及实例"
description: "xpath的路径表达式有着丰富的技巧，通过巧妙的组合可以完成强大的功能。"
category: xml
tags: [xml, xpath, 路径表达式]
comments: true
---

### OverView

XPath 是一门在 XML 文档中查找信息的语言。XPath 可用来在 XML 文档中对元素和属性进行遍历。<br/>
XPath 是 W3C XSLT 标准的主要元素，并且 XQuery 和 XPointer 都构建于 XPath 表达之上。<br/>
XPath 的主要应用技巧就是在一篇XML文档中选择特定的节点，这就用到了 XPath 中的路径表达式，本文也就主要介绍路径表达式的用法。

<!-- more -->

### 基本规则

`/` 表示直接子节点，如果在开头表示从根节点开始选择。<br/>
`//` 表示任意层次的子孙节点。<br/>
`.` 指示当前上下文。<br/>
`..` 当前上下文节点的父级。<br/>
`*` 通配符；选择所有元素，与元素名无关。<br/>
`@` 属性；属性名的前缀。<br/>
`@*` 属性通配符；选择所有属性，与名称无关。<br/>
`:` 命名空间分隔符；将命名空间前缀与元素名或属性名分隔。<br/>
`( )` 为运算分组，明确设置优先级。<br/>
`[ ]` 应用筛选模式。<br/>
`[ ]` 下标运算符；用于在集合中编制索引。<br/>
`+` 执行加法。<br/>
`-` 执行减法。<br/>
`div` 根据 IEEE 754 执行浮点除法。<br/>
`*` 执行乘法。</br>
`mod` 从截断除法返回余数。<br/>

### XPath轴语法

1. `ancestor::` 选取当前节点的所有先辈（父、祖父等）。
2. `ancestor-or-self::` 选取当前节点的所有先辈（父、祖父等）以及当前节点本身。
3. `attribute::` 选取当前节点的所有属性。
4. `child::` 选取当前节点的所有子元素。
5. `descendant::` 选取当前节点的所有后代元素（子、孙等）。
6. `descendant-or-self::` 选取当前节点的所有后代元素（子、孙等）以及当前节点本身。
7. `following::` 选取文档中当前节点的结束标签之后的所有节点。
8. `namespace::` 选取当前节点的所有命名空间节点。
9. `parent::` 选取当前节点的父节点。
10. `preceding::` 选取文档中当前节点的开始标签之前的所有节点。
11. `preceding-sibling::` 选取当前节点之前的所有同级节点。
12. `self::` 选取当前节点。

### 常用示例

1. `/AAA` 从根节点选择AAA
2. `/AAA/DDD/BBB` 从根节点AAA的孩子DDD的孩子BBB节点
3. `//BBB` 选择所有BBB节点
4. `//DDD/BBB` 选择所有父节点为DDD的BBB节点
5. `/AAA/CCC/DDD/*` 选择所有/AAA/CCC/DDD/下的节点
6. `/*/*/*/BBB` 选择有3个祖先的BBB节点
7. `//*` 选择所有节点
8. `/AAA/BBB[1]` 选择/AAA下的第一个BBB节点
9. `/AAA/BBB[last()]` 选择父节点为AAA的最后一个BBB节点
10. `//@id` 选择所有属性id的节点
11. `//BBB[@id]` 所有含有属性id的BBB节点
12. `//BBB[@name]` 所有含有属性name的BBB节点
13. `//BBB[@*]` 所有拥有属性的节点
14. `//BBB[not(@*)]` 所有不拥有属性的节点
15. `//BBB[@id='b1']` 选择属性id值为b1的BBB节点
16. `//BBB[@name='bbb']` 选择属性name值为bbb的BBB节点
17. `//BBB[normalize-space(@name)='bbb']` 选择属性name值为bbb的BBB节点，比较时去掉开头结尾空白
18. `//*[count(BBB)=2]` 选择含有两个BBB孩子的节点
19. `//*[count(*)=2]` 选择拥有两个孩子的节点
20. `//*[count(*)=3]` 选择拥有三个孩子的节点
21. `//*[name()='BBB']` 选择所有BBB节点（name()函数返回节点名），和//BBB相同
22. `//*[starts-with(name(),'B')]` 选择所有name以B开头的节点
23. `//*[contains(name(),'C')]` 选择name包含C的节点
24. `//*[string-length(name()) = 3]` 选择name长度为3的节点
25. `//*[string-length(name()) < 3]` 选择name长度小于3的节点
26. `//*[string-length(name()) > 3]` 选择name长度大于3的节点
27. `//CCC | //BBB` 选择所有BBB或者CCC节点
28. `/AAA/EEE | //BBB` /AAA/EEE 或者 //BBB
29. `/AAA/EEE | //DDD/CCC | /AAA | //BBB` : /AAA/EEE 或 //DDD/CCC 或 /AAA 或 //BBB
30. `/child::AAA` 同/AAA （child::表示当前环境的孩子节点）
31. `/child::AAA/child::BBB` 同/AAA/BBB
32. `/descendant::*` 选择根元素的所有后代节点（descendant::表示当前环境的后代节点， 同//）
33. `/AAA/BBB/descendant::*` 选择/AAA/BBB下所有后代节点
34. `//CCC/descendant::*` 选择所有CCC节点的后代节点
35. `//CCC/descendant::DDD` 选择所有CCC节点的后代为DDD的节点
36. `//DDD/parent::*` 选择所有DDD节点的父节点（parent::表示当前环境的父节点）
37. `//FFF/ancestor::*` 选择所有FFF节点的祖先节点（ancestor::表示当前环境的祖先节点）
38. `/AAA/BBB/following-sibling::*` 选择/AAA/BBB之后的所有同级节点
39. `//CCC/following-sibling::*` 选择//CCC之后的所有同级节点
40. `/AAA/XXX/preceding-sibling::*` 选择所有/AAA/XXX之前的同级节点。
41. `//CCC/preceding-sibling::*` 选择//CCC之前的所有同级节点
42. `/AAA/XXX/following::*` 选择/AAA/XXX所有文档顺序的后续节点（following轴包含同一文档中按文档顺序位于上下文节点之后的所有节点, 除了祖先节点,属性节点和命名空间节点）
43. `//ZZZ/following::*` 选择//ZZZ的所有文档顺序的后续节点
44. `/AAA/XXX/preceding::*` 选择/AAA/XXX的所有文档顺序的之前的节点（preceding::选取文档中当前节点的开始标签之前的所有节点）
45. `//GGG/preceding::*` 选择//GGG的文档顺序之前的节点
46. `/AAA/XXX/descendant-or-self::*` 选择/AAA/XXX的所有后代节点，包括自身
47. `/AAA/XXX/DDD/EEE/ancestor-or-self::*` 选择/AAA/XXX/DDD/EEE的所有祖先节点，包括自身
48. `//BBB[position() mod 2 = 0 ]` 选择下标偶数的BBB节点
49. `//BBB[ position() = floor(last() div 2 + 0.5) or position() = ceiling(last() div 2 + 0.5) ]` 选择中间的BBB节点（floor向下取整，ceiling向上取整） 
50. `//CCC[ position() = floor(last() div 2 + 0.5) or position() = ceiling(last() div 2 + 0.5) ]` 选择中间的CCC节点

### 参考
1. XPath实例参考: <http://www.zvon.org/xxl/XPathTutorial/General_chi/examples.html>.
2. XPath轴说明: <http://www.w3school.com.cn/xpath/xpath_axes.asp>.
3. XPath运算符: <http://msdn.microsoft.com/zh-cn/library/ms256122.aspx>.
