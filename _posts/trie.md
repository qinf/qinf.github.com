title: 数据结构之键树  
date: 2015-06-20 15:30:16  
categories: Technology  
tags: [算法与数据结构]
---
## 概念
> 如果一个关键字可以表示成字符的序号，即字符串，那么可以用键树（`keyword tree`），又称数字搜索树（`digital search tree`）或字符树，来表示这样的字符串的集合。键树又称为数字查找树（`Digital Search Tree`)或`Trie`树(`trie`为`retrieve`中间4个字符)，其结构受启发于一部大型字典的“书边标目”。

<!-- more -->
键树的度可以大于2，树中的结点不是包含一个或几个关键字，而是只含有组成关键字的符号。如果关键字是数值则结点中只包含一个数位；如果是单词，则结点只包含一个字母字符。    
假设有如下集合：{`CAI`, `CAO`, `CBA`, `BDA`}。对此集合先按首字母分成不同的子集，然后再按子集中第二个字符进行分割...，直到每个小子集只包含一个关键字为止。   
![](http://7xj0l3.com1.z0.glb.clouddn.com/key_tree.png)

## 表示方式
### 双链树(`Trie`树)
用树的孩子－兄弟链来表示键树(此时油称为双链树)  
每个`Node`有三个域：  
`symbol`域：存储关键字的一个字符；  
`son`域：存储指向第一棵子树大根的指针；  
`brother`域：存储指向右兄弟的指针。  
查找：从跟结点出发，顺着son结点查找，如果相等，继续查找下一个送结点。否则沿着`brother`查找。直到指针为空为止。此时若仍未完成`key`的查找，则查找不成功。  
双链表中插入删除一个结点相当于树中结点上插入删除一颗子树。  
### 用多重链表表示(又称Trie树，字典树)  
如果以树的多重链表表示建树，则树的每个结点中应该包含d个指针域，比如26个（代表26个英文字母），此时键树又称为`Trie`树。下面给出`Trie`的一个实现：  

```c++  
#include <iostream>
#include <string>
#include <cstring>
using namespace std;
const int branchNum = 256;
struct TrieNode {
    bool isStr; // 是否是一个字符串
    TrieNode* next[branchNum];  // 下一个字符
    TrieNode(): isStr(false) { memset(next, 0, sizeof(TrieNode) * branchNum); }
};
class Trie {
public:
    Trie() { root = new TrieNode(); } // 构造函数生产一个结点
    ~Trie() { if (root) _deleteTrie(root); }
    void insert(string word) {  // 插入操作
        _insert(word.cbegin(), word.cend());
    }
    TrieNode* getTrie();
    bool search(string word) {  // 查找操作
        return _search(word.cbegin(), word.cend());
    }
private:
    TrieNode* root;
    void _insert(string::const_iterator beg, string::const_iterator end);
    void _deleteTrie(TrieNode *tree);
    bool _search(string::const_iterator beg, string::const_iterator end);
};
// 查找操作
void Trie::_insert(string::const_iterator beg, string::const_iterator end) {
    TrieNode* location = root;
    while (beg != end) {
          // 待查找到下一字符不存在，开辟一个新结点并插入
        if (location->next[*beg] == NULL) {
            TrieNode* node = new TrieNode();
            location->next[*beg] = node;
        }
        // 继续向下
        location = location->next[*beg];
        ++beg;
    }
    location->isStr = true;
}

// 销毁Trie树时，调用此函数
void Trie::_deleteTrie(TrieNode* tree) {
    for (int i = 0; i < branchNum; ++i) {
        if (tree->next[i]) {
            _deleteTrie(tree->next[i]);
        }
    }
    delete tree;
}
// 搜索字符串是否存在
bool Trie::_search(string::const_iterator beg, string::const_iterator end) {
    TrieNode* location = root;
    while (beg != end && location) {
        location = location->next[*beg];
        ++beg;
    }
    return (location != NULL && location->isStr);
}
int main(int argc, const char * argv[]) {
    // insert code here...
    Trie tree;
    tree.insert("中华人民共和国");
    tree.insert("人民");
    tree.insert("aBcD");
    tree.insert("\0123");
    cout << tree.search("\0123") << endl;
    cout << tree.search("aBcD") << endl;
    cout << tree.search("人民") << endl;
    cout << tree.search("中华人民共和国") << endl;
    return 0;
}
```
以上是`Trie`树的一个实现，实际使用时可以做适当修改。
`Trie`树是利用字符串的公共前缀来降低开销的，`Trie`树常用于统计和排序大量字符串，经常用于统计文本词频。因此`Trie`树对**缺点**也十分明显，当字符串没有公共前缀时，将非常耗内存。

