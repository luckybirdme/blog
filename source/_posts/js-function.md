---
title: 函数式编程
date: 2020-06-27 15:56:09
tags: JavaScript
---

> 柯里化编程

<!-- more -->


## 一. 命令式编程
以具体案例来理解：二叉树镜像反转一下

```py

def invertTree(root):
    if root is None:
        return None
    # 翻转左边
    right = invertTree(root.left)
    # 翻转右边
    left = invertTree(root.right);
    # 左右互换
    root.left = left
    root.right = right
    # 返回反转的二叉树
    return root

```

这里直接在原二叉树上翻转，把达到目的的步骤详细的描述出来，就好像编写指令序列一样，一步步执行指令从而达到目的。


## 二. 函数式编程
以具体案例来理解：二叉树镜像反转一下

```py
def invert(node):
    if node is None:
        return None
    else
    	# 左节点用右节点表示，右节点有左节点表示
        return Tree(node.value, invert(node.right), invert(node.left))
```

这里直接返回一颗新的二叉树，而且是通过描述一个旧树到新树的映射来实现，因此函数式编程重点在于描述输入和输出的映射的关系。其主要特点如下：

1. 函数即不依赖外部的状态也不修改外部的状态，函数调用的结果不依赖调用的时间和位置，这样写的代码容易进行推理，不容易出错。这使得单元测试和调试都更容易。

2. 由于（多个线程之间）不共享状态，不会造成资源争用(Race condition)，也就不需要用锁来保护可变状态，也就不会出现死锁，这样可以更好地并发起来，尤其是在对称多处理器（SMP）架构下能够更好地利用多个处理器（核）提供的并行处理能力。

3. 由于函数式语言是面向数学的抽象，描述输入和输出的映射关系，更接近人的语言，而不是机器语言，代码会比较简洁，也更容易被理解。


## 三. 柯里化编程

柯里化（Currying）是以美国数理逻辑学家哈斯凯尔·科里(Haskell Curry)的名字命名的函数应用方式。与偏函数很像的地方是：都可以缓存参数，都会返回一个新的函数，以提高程序中函数的适用性。而不同点在于，柯里化（Currying）通常用于分解原函数式，将参数数量为 n 的一个函数，分解为参数数量为 1 的 n 个函数，并且支持连续调用。例如：

```js

// 一个用于计算三个数字累加的函数
const addExample = function(a, b, c){
    return a + b + c;
}
// 调用
addExample(10, 5, 3); // 结果: 18

// 通过柯里化，对上述函数进行演变
const addCurry = function(a){
    return function(b){
        return function(c){
            return a + b + c;
        }
    }
}
// 缔造新的 单一元 函数
const add10 = addCurry(10);
const add15 = add10(5);
const add18 = add15(3);
// 调用
add18();    // 结果: 18

```

更有实际意义的例子，利用闭包特性，缓存参数，并返回闭包函数

```js

// 声明一个用于创建 二元计算器 的函数
// 函数接收一个 oper 作为运算操作符( 字符串，可用值：+，-，*，/，% )
function createCalculator(oper){
    
    // 新的函数接收用于计算的两个数字
    return function(a, b){
        switch(oper){
            case '+': return a+b;
            case '-': return a-b;
            case '*': return a*b;
            case '/': return a/b;
            case '%': return a%b;
            default: throw 'Operator Formart Error: ' + oper;
        }
    }
}
// 生成加法函数
const add = createCalculator('+');
const sub = createCalculator('-');

// 运行
add(10, 8);     // 结果: 18
sub(10, 8);     // 结果: 2
```


根据不同场景，缓存不同参数


```js

// 定义一个流速转换函数：

function flowVelocityFormater(base, power) {
    return function(v) {
        return (v / Math.pow(base, power)).toFixed(2);
    }
}

// 在此基础上，得到几个基本的转换函数：
var bps2Gbps = flowVelocityFormater(1000, 9);
var Kbps2Gbps =  flowVelocityFormater(1000, 6);

// 实际转换时就可以使用
// 语义简洁清晰多了
newValue = bps2Gbps(value);

// 未来有一天，单位转换需要按1024而不是1000转换时，也只需要修改参数即可，代码的维护性大大增强。
bps2Gbps = flowVelocityFormater(1024, 9);

```