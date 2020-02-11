---
title: 前缀树（字典树）
vssue-title: 前缀树（字典树）
date: 2019-09-02
tags:
  - python
  - 算法
categories: [算法]
---
[[toc]]

Trie (发音为 "try") 或前缀树是一种树数据结构，用于检索字符串数据集中的键。这一高效的数据结构有多种应用：

1. 自动补全
2. 拼写检查
3. IP 路由 (最长前缀匹配)
4.  T9 (九宫格) 打字预测
5. 单词游戏


## 字典树优势

还有其他的数据结构，如平衡树和哈希表，使我们能够在字符串数据集中搜索单词。为什么我们还需要 Trie 树呢？
尽管哈希表可以在 O(1)O(1) 时间内寻找键值，却无法高效的完成以下操作：

1. 找到具有同一前缀的全部键值。
2. 按词典序枚举字符串的数据集。
Trie 树优于哈希表的另一个理由是，随着哈希表大小增加，会出现大量的冲突，时间复杂度可能增加到 O(n)O(n)，其中 nn 是插入的键的数量。与哈希表相比，Trie 树在存储多个具有相同前缀的键时可以使用较少的空间。此时 Trie 树只需要 O(m)O(m) 的时间复杂度，其中 mm 为键长。而在平衡树中查找键值需要 O(m \log n)O(mlogn) 时间复杂度。




## Trie 树的结点结构

Trie 树是一个有根的树，其结点具有以下字段：

- 最多 RR 个指向子结点的链接，其中每个链接对应字母表数据集中的一个字母。
- 布尔字段，以指定节点是对应键的结尾还是只是键前缀。

### 插入

我们通过搜索 Trie 树来插入一个键。我们从根开始搜索它对应于第一个键字符的链接。有两种情况：

- 链接存在。沿着链接移动到树的下一个子层。算法继续搜索下一个键字符。

- 链接不存在。创建一个新的节点，并将它与父节点的链接相连，该链接与当前的键字符相匹配。
  重复以上步骤，直到到达键的最后一个字符，然后将当前节点标记为结束节点，算法完成。

![](https://pic.leetcode-cn.com/0cddad836ee9a200b150a3d89f96035f44f3643c4fba0cb1f329e2307c714895-file_1562596867185)

### 查找键

每个键在 trie 中表示为从根到内部节点或叶的路径。我们用第一个键字符从根开始，。检查当前节点中与键字符对应的链接。有两种情况：

- 存在链接。我们移动到该链接后面路径中的下一个节点，并继续搜索下一个键字符。
- 不存在链接。若已无键字符，且当前结点标记为 isEnd，则返回 true。否则有两种可能，均返回 false :
  还有键字符剩余，但无法跟随 Trie 树的键路径，找不到键。
  没有键字符剩余，但当前结点没有标记为 isEnd。也就是说，待查找键只是Trie树中另一个键的前缀。

![](https://pic.leetcode-cn.com/7cc64e93088feeedece697a7cae0c7240245e4c5e05de22634b610d7dddb31c8-image.png)

``class Trie:
    word = ''
    def __init__(self):
        """
        Initialize your data structure here.
        """ 
        self.root = {}
        self.end = -1 
        

```python
def insert(self, word: str) -> None:
    """
    Inserts a word into the trie.
    """
    curNode = self.root   
    for w in word:
        if not w in curNode:
            curNode[w] = {}
        curNode = curNode[w]
    curNode[self.end] = True

def search(self, word: str) -> bool:
    """
    Returns if the word is in the trie.
    """
    curNode = self.root
    for w in word:
        if w in curNode:
            curNode = curNode[w]
        else: 
            return False
    if not self.end in curNode:
        return False
    return True

def startsWith(self, prefix: str) -> bool:
    """
    Returns if there is any word in the trie that starts with the given prefix.
    """
    curNode = self.root
    for w in prefix:
        if w in curNode:
            curNode = curNode[w]
        else: 
            return False
    return True
```