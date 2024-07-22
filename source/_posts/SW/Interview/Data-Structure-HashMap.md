---
title: 哈希表
date: 2020-06-06 16:22:33
tags: 
    - C/C++
    - 算法
    - 数据结构
categories: 
    - 算法/数据结构
---
散列技术是在记录的存储位置和它的关键字之间建立一个确定的对应关系f，使得每个关键字key对应一个存储位置f(key)。f称之为散列函数，又称之为哈希函数。采用散列技术将记录存储在一块连续的存储空间中，这块连续的存储空间称为散列表或者哈希表。

<!-- more -->
散列技术最适合的求解问题是查找与给定值相等的记录。
* 注：如果key1 != key2但是f(key1) = f(key2)，此时称key1和key2冲突，称其为`同义词`，实际设计需避免。


- [散列函数构造方法](#散列函数构造方法)
- [处理散列冲突](#处理散列冲突)
- [构建哈希表和查找](#构建哈希表和查找)
- [Leetcode哈希表题目汇总](#leetcode哈希表题目汇总)
<!-- more -->
## 散列函数构造方法
* 线性均匀分布
取关键字的某个线性函数值作为散列地址。`f(key) =  a * key + b`
* 数字分析法
避开可能重复的特征，选取一定不相同的特征作散列地址
* 平方取中法
* 折叠法
* 除数留余法
利用同余的特点，但容易导致冲突。`f(key) =  key mod p`
表长为m，p选择小于等于表长的最小质数（最好接近表长）或者不包括小于20质因子的合数。
* 随机数
采用random函数。
  
## 处理散列冲突
* 开放定址法
* 再散列函数法
* 链地址法
* 公共溢出区法

  
## 构建哈希表和查找
```cpp
#define SUCCESS 1
#define UNSUCCESS 0
#define HASHSIZE 12
#define NULLKEY -32768
typedef struct {
    int *elem;  //数组元素存储基址，动态分配
    int count;  //当前数据元素个数
}HashTable;
int m = 0;  //哈希表长度
```
初始化哈希表
```cpp
Status InitHashTable(HashTable *H){
    int i = 0;
    m = HASHSIZE;
    H->count = m;
    H->elem = (int *)malloc(sizeof(int) * m);
    for (i = 0; i < m ; i++){
        H->elem[i] = NULLKEY;   //地址设置为NULLKEY（默认）
    }
    return OK;
}
```
定义散列函数：
```cpp
int Hash(int key){
    return key & m;  //除数留余法
}
```
插入关键字进入哈希表：
```cpp
void InserHash(HashTable *, int key){
    int addr = Hash(key);  //获取哈希地址
    while(H->elem[addr] != NULLKEY){
        //如果地址不为空，说明已经有值插入
        addr = (addr + 1) % m; //使用开放定址法的线性探测重新分配地址
    }
    H->elem[addr] = key;  //将该地址的值设为key
}
```
散列表查找关键字，首先计算散列地址，若已经被其他的占了则使用开放定址法重新寻址，直到找不到为止：
```cpp
status SerchHash(HashTable H, int key, int *addr){
    *addr = Hash(key);
    while(H.elem[*addr] != key){  //不匹配则说明冲突，继续寻找
        *addr = (*addr + 1) % m;
        if (H.elem[*addr] == NULLKEY || *addr == Hash(key)){
            //循环一圈或者开放定址的为空，则说明不存在
            return UNSUCCESS;
        }
    }
    return SUCCESS;
}
```
## Leetcode哈希表题目汇总
set和map使用参考[C++容器及使用](http://175.24.64.165/2020/06/06/C-容器/)
[1. 两数之和](https://leetcode-cn.com/problems/two-sum/)
给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那两个整数，并返回他们的数组下标。
你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。
```
给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
  
一遍哈希：事实证明，我们可以一次完成。在进行迭代并将元素插入到表中的同时，我们还会回过头来检查表中是否已经存在当前元素所对应的目标元素。如果它存在，那我们已经找到了对应解，并立即将其返回
```
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int,int> m;
        
        for(int i = 0; i < nums.size(); i++)
        {
            if(m.find(target-nums[i]) != m.end())      //m中存在对应的键值
                return {m[target-nums[i]], i};        
            //  m[target-nums[i]]为已经加入map的元素的索引，所以小于本轮循环中的i，放在前面

            m[nums[i]] = i;       //向map中添加元素
        }
        return {};
    }
};
```
[3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)
给定一个字符串，请你找出其中不含有重复字符的最长子串的长度。

```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
示例 2:
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
示例 3:
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```
  
使用双指针和哈希表解决该问题：左右指针范围的内容构建表，返回最长无重复的双指针差值：
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        // 哈希集合，记录每个字符是否出现过
        unordered_set<char> occ;
        int n = s.size();
        // 右指针，初始值为 -1，相当于我们在字符串的左边界的左侧，还没有开始移动
        int rk = -1, ans = 0;
        // 枚举左指针的位置，初始值隐性地表示为 -1
        for (int i = 0; i < n; ++i) {
            if (i != 0) {
                // 左指针向右移动一格，移除一个字符
                occ.erase(s[i - 1]);
            }
            while (rk + 1 < n && !occ.count(s[rk + 1])) {
                // 不断地移动右指针
                occ.insert(s[rk + 1]);
                ++rk;
            }
            // 第 i 到 rk 个字符是一个极长的无重复字符子串
            ans = max(ans, rk - i + 1);
        }
        return ans;
    }
};
```