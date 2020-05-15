---
layout:     post
title:      golang interface总结
subtitle:   golang总结
date:       2020-05-13
author:     jianshao
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - golang
    - interface
---

>使用golang写东西过程中的总结，持续更新。。。

前端时间想写个前缀树golang实现，刚开始使用函数式编程（c语言那种）很快就写完了。不过既然拿golang练手，肯定要搞复杂点，顺便练练golang里的其他东西，比如interface。一直想着把功能往interface上套，终究还是不熟练，前前后后花了挺长时间才有个大概满意的结果。在这个过程中也确实加深了对interface的理解，总结还是要总结一下的。

interface总体来说有2大功能：

1.作为映射使用。

2.作为多态的实现基础，实际上跟C++中的多态还不完全一样。



想要实现的算法是前缀树，实现算法本身的逻辑并没有什么难度，主要是提高程序的扩展性。第一个想法就是在每个节点上使用interface存储数据，这里使用interface作为映射部分的特性。可以设计节点结构如下：

```go
type trieNode struct {
		fileds map[string]*trieNode   //直接使用map，省事
		isLeaf bool                   //叶子节点标识，只有叶子才能存储数据
		value  interface{}            //具体数据，数据的类型要使用者自己控制
}

type Trie struct {
		root *trieNode
}
```
针对Trie实现增删改查接口，使用者唯一需要注意的地方就是对数据操作时，如果期望类型与实际类型不一致，操作时会panic。当然了，在操作时可以增加defer/recover规避。



上面这种实现方式用C也完全一样，下面尽量用golang的方式重新设计一个。

```go
type iTrieNode interface {
  GetValue() interface{}
  SetValue(v interface{})
  IsLeafNode() bool
  SetLeafNode()
  GetChildNode(field string) iTrieNode
  SetChileNode(field string, node iTrieNode)
}

type trieNode struct{
		fileds map[string]*trieNode   //直接使用map，省事
		isLeaf bool                   //叶子节点标识，只有叶子才能存储数据
		value  interface{}            //具体数据，数据的类型要使用者自己控制
}

type iTrie interface{
  NewTrieNode() iTrieNode
  GetRootNode() iTrieNode
}

type Trie struct {
  root iTrieNode                  //前缀树的根节点，初始化时申请
  this iTrie                      //使用this访问派生类的成员
}

type TestTrie struct{
  *Trie
}
```

在实现的过程中碰到了几个问题：

1.以基类不能访问派生类的成员，如即使TestTrie结构实现了iTrie接口，也不能通过Trie访问TestTrie的GetRootNode方法。

这个地方跟C++的多态有点不一样，C++实现多态后直接就能使用最后一个实现方法，而golang确不行。

从网上参考别人的处理方法就是将派生类以参数形式传入方法里，然后使用该参数调用需要的方法。

```go
func (t *TestTrie)Add(trie iTrie) {
  node := trie.GetRootNode()
  ......
}
```

这样处理可以解决上述的问题，但是在调用方法时需要将自己作为参数传入，整体方法的调用风格不够统一（看着不爽）。所以为了使调用风格看起来统一，在基类里增加了一个this成员，所有对成员方法的调用都通过this，这样看起来就比较舒服了。就像下面这样：

```go
func (t *TestTrie)getThis() iTrie {
  return t.this
}

func (t *TestTrie)Add() {
  this := t.getThis()
  node := this.GetRootNode()
}
```


