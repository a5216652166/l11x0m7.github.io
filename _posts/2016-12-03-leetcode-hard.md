--- 
layout: post 
title: Leetcode的Hard难度题目汇总
date: 2016-12-03 
categories: blog 
tags: [算法, leetcode] 
description: 
--- 

# 126. Word Ladder II

#### 题目

Given two words (beginWord and endWord), and a dictionary's word list, find all shortest transformation sequence(s) from beginWord to endWord, such that:

1. Only one letter can be changed at a time
2. Each intermediate word must exist in the word list
For example,

Given:
beginWord = `"hit"`
endWord = `"cog"`
wordList = `["hot","dot","dog","lot","log"]`
Return

```
  [
    ["hit","hot","dot","dog","cog"],
    ["hit","hot","lot","log","cog"]
  ]
```
  
Note:

* All words have the same length.
* All words contain only lowercase alphabetic characters.

#### 思路

使用广度优先遍历，查询每个从begin开始的所有邻近单词。判定在该层上是否有到达end的单词，如果有，则保存该层上所有的能够到达终点的路径；如果没有，则保存当前所有路径，并将这些词从原始字典中删除，开始下一层的遍历，直到在某一层找到能够到达end的路径。

#### 代码

```cpp
class Solution {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord, unordered_set<string> &wordList) {
        vector<vector<string>> res;
        queue<vector<string>> path;
        if(wordList.empty())
            return res;
        if(beginWord==endWord){
            string tmp[] = {beginWord, endWord};
            res.push_back(vector<string>(tmp, tmp+1));
            res.push_back(vector<string>(tmp, tmp+2));
            return res;
        }
        unordered_set<string> tag;
        wordList.insert(endWord);
        int min_lev = INT_MAX;
        int level = 1;
        path.push({beginWord});
        tag.insert(beginWord);
        while(!path.empty()){
            vector<string> curway = path.front();
            path.pop();
            if(curway.size()>level){
                for(string tmp:tag)wordList.erase(tmp);
                tag.clear();
                if(curway.size()>min_lev)
                    break;
                else
                    level = curway.size();
            }
            string last = curway.back();
            for(int i=0;i<last.length();i++){
                string tmp = last;
                char curc = last[i];
                for(char c='a';c<='z';c++){
                    if(curc==c)
                        continue;
                    tmp[i] = c;
                    if(wordList.find(tmp)!=wordList.end()){
                        tag.insert(tmp);
                        curway.push_back(tmp);
                        if(tmp==endWord){
                            res.push_back(curway);
                            min_lev = level;
                        }
                        else
                            path.push(curway);
                        curway.pop_back();
                    }
                }
            }
        }
        return res;
    }
};
```


# 149. Max Points on a Line

#### 题目

`Given n points on a 2D plane, find the maximum number of points that lie on the same straight line.`

#### 思路  
算法的基本思路是遍历每个点，找到每个点对应的线段斜率，并统计斜率相同的线段数，从而得到结果。这里注意两个特殊情况：

* 两点在同一垂直线上
* 两点坐标重叠

复杂度：O(n^2)

注意如果使用的是map而不是unordered_map，则是O(n^2logn)。

#### 代码

```cpp
/**
 * Definition for a point.
 * struct Point {
 *     int x;
 *     int y;
 *     Point() : x(0), y(0) {}
 *     Point(int a, int b) : x(a), y(b) {}
 * };
 */
class Solution {
public:
    int maxPoints(vector<Point>& points) {
        int result = 0;
        for(int i=0;i<points.size();i++){
            unordered_map<double, int> m;
            int dup = 0, vertical = 0;
            double gradient = 0;
            int curmax = 0;
            for(int j=i+1;j<points.size();j++){
                if(points[i].x==points[j].x){
                    if(points[i].y==points[j].y)
                        dup++;
                    else
                        vertical++;
                    curmax = max(curmax, vertical);
                }
                else{
                    gradient = (points[i].y-points[j].y)*1.0/(points[i].x-points[j].x);
                    m[gradient]++;
                    curmax = max(curmax, m[gradient]);
                }
            }
            result = max(result, curmax+dup+1);
        }
        return result;
    }
};
```


# 146. LRU Cache

#### 题目

Design and implement a data structure for Least Recently Used (LRU) cache. It should support the following operations: `get` and `set`.

`get(key)` - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.
`set(key, value)` - Set or insert the value if the key is not already present. When the cache reached its capacity, it should invalidate the least recently used item before inserting a new item.

#### 思路

第一种，使用带空head和空rear的链表，每次get的时候将被get的值移动到前面（移动的时间复杂度为O(1)，但是查询的时间复杂度为O(n)）；每次set的时候，
* 如果该值在原列表中存在，则将该值移动到开头；
* 如果不在原链表中，且没有达到容量，则插入到链表开头；
* 如果不在原链表中，且达到了容量，则删除链表尾元素，并插入到链表开头。

每次查找链表中对应key的时候都要遍历链表，因此复杂度较低。  
第二种，可以考虑用map做这样的映射：key->pair<value, 在链表中的位置指针>。

#### 代码

```cpp
# 第一种
struct Node{
    int key;
    int value;
    Node* next;
    Node* pre;
    Node(int key=-1, int value=-1){
        this->key = key;
        this->value = value;
        next=NULL,pre=NULL;
    }
};
class LRUCache{
public:
    int cap;
    int len;
    unordered_map<int, int> m;
    Node* head;
    Node* rear;
    // @param capacity, an integer
    LRUCache(int capacity) {
        // write your code here
        cap = capacity;
        head = new Node();
        rear = new Node();
        head->next = rear;
        rear->pre = head;
        len = 0;
    }
    
    // @return an integer
    int get(int key) {
        // write your code here
        if(m.find(key)!=m.end()){
            Node* h = head;
            while(h&&h->key!=key)
                h = h->next;
            moveFront(h);
            return m[key];
        }
        else
            return -1;
    }

    // @param key, an integer
    // @param value, an integer
    // @return nothing
    void set(int key, int value) {
        // write your code here
        if(cap<1)
            return;
        if(m.find(key)==m.end()){
            m[key] = value;
            if(len>=cap)
                deleteLast();
            else
                len++;
        
            Node* h = new Node(key, value);
            h->next = head->next;
            head->next->pre = h;
            head->next = h;
            h->pre = head;
        }
        else{
            Node* h = head;
            while(h&&h->key!=key)
                h = h->next;
            h->value = value;
            m[key] = value;
            moveFront(h);
        }
        
    }
    void moveFront(Node* &h){
        h->pre->next = h->next;
        h->next->pre = h->pre;
        h->next = head->next;
        head->next->pre = h;
        h->pre = head;
        head->next = h;
    }
    void deleteLast(){
        Node* h = head;
        while(h->next->next){
            h = h->next;
        }
        h->pre->next = h->next;
        h->next->pre = h->pre;
        m.erase(h->key);
        // delete h;
    }
};
```

```cpp
# 第二种
class LRUCache{
public:
    int cap;
    list<int> l;
    unordered_map<int, pair<int, list<int>::iterator>> m;
    // @param capacity, an integer
    LRUCache(int capacity) {
        // write your code here
        cap = capacity;
    }
    
    // @return an integer
    int get(int key) {
        // write your code here
        auto it = m.find(key);
        if(it!=m.end()){
            update(it);
            return it->second.first;
        }
        else
            return -1;
    }

    // @param key, an integer
    // @param value, an integer
    // @return nothing
    void set(int key, int value) {
        // write your code here
        if(cap<1)
            return;
        auto it = m.find(key);
        if(it==m.end()){
            if(m.size()>=cap){
                m.erase(l.back());
                l.pop_back();
            }
            l.push_front(key);
        }
        else
            update(it);
        m[key] = make_pair(value, l.begin());
    }
    void update(unordered_map<int, pair<int, list<int>::iterator>>::iterator it){
        int key = it->first;
        l.erase(it->second.second);
        l.push_front(key);
        it->second.second = l.begin();
    }
};
```

# 460. LFU Cache

#### 题目

Design and implement a data structure for Least Frequently Used (LFU) cache. It should support the following operations: `get` and `set`.

* `get(key)` - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.
* `set(key, value)` - Set or insert the value if the key is not already present. When the cache reaches its capacity, it should invalidate the least frequently used item before inserting a new item. For the purpose of this problem, when there is a tie (i.e., two or more keys that have the same frequency), the least recently used key would be evicted.

Follow up:  
Could you do both operations in **O(1)** time complexity?

Example:

```
LFUCache cache = new LFUCache( 2 /* capacity */ );

cache.set(1, 1);
cache.set(2, 2);
cache.get(1);       // returns 1
cache.set(3, 3);    // evicts key 2
cache.get(2);       // returns -1 (not found)
cache.get(3);       // returns 3.
cache.set(4, 4);    // evicts key 1.
cache.get(1);       // returns -1 (not found)
cache.get(3);       // returns 3
cache.get(4);       // returns 4
```

#### 思路

用三个hash表，分别为：

* key -> (freq, list iterator)
* key -> value
* freq -> list

相当于将每个键按照出现的不同频率进行分桶。每个频率对应一个存储key的列表。而每个key对应所在频率的列表位置，是一个pair。那么在get的时候，我们只需要找到对应key的freq，之后找到该freq下该key的位置即可。在set的时候，如果超出容量，则删除least元素（频率最低、被访问的时间间隔最久），之后再插入新的元素到freq=1的list里。

注意，只要产生可能使列表为空的操作（比如插入或者更新，插入会让最小频率变为1），都要更新最小频率参数least，这个参数用来在删除元素的时候使用。

时间复杂度：O(1)

#### 代码

```cpp
class LFUCache {
public:
    LFUCache(int capacity) {
        this->capacity = capacity;
        least = 1;
    }
    
    int get(int key) {
        auto it = key2pair.find(key);
        int ret;
        if(it!=key2pair.end()){
            update(it);
            ret = key2value[key];
            if(freq2list[least].size()==0){
                least = key2pair[key].first;
                // cout<<"least:"<<least<<endl;
            }
        }
        else
            ret = -1;
        return ret;
    }
    
    void set(int key, int value) {
        if(capacity<=0)
            return;
        int val = get(key);
        if(val!=-1){
            key2value[key] = value;
        }
        else{
            if(key2pair.size()>=capacity){
                key2pair.erase(freq2list[least].back());
                key2value.erase(freq2list[least].back());
                freq2list[least].pop_back();
            }
            key2value[key] = value;
            freq2list[1].push_front(key);
            key2pair[key].first = 1;
            key2pair[key].second = freq2list[1].begin();
            least = 1;
        }
        // for(auto i:freq2list){
        //     cout<<(i.first)<<" ";
        //     for(auto j:(i.second))
        //         cout<<j<<" ";
        //     cout<<endl;
        // }
        
    }
    void update(unordered_map<int, pair<int, list<int>::iterator>>::iterator it){
            int freq = it->second.first;
            key2pair[it->first].first = freq + 1;
            freq2list[freq].erase(it->second.second);
            // cout<<"freq:"<<freq<<" "<<freq2list[freq].size()<<endl;
            freq2list[freq+1].push_front(it->first);
            it->second.second = freq2list[freq+1].begin();
    }
private:
    int capacity;
    int least;
    unordered_map<int, pair<int, list<int>::iterator>> key2pair;
    unordered_map<int, int> key2value;
    unordered_map<int, list<int>> freq2list;
};

/**
 * Your LFUCache object will be instantiated and called as such:
 * LFUCache obj = new LFUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.set(key,value);
 */
```

# 466. Count The Repetitions

#### 题目

Define `S = [s,n]` as the string S which consists of n connected strings s. For example, `["abc", 3]` ="abcabcabc".

On the other hand, we define that string s1 can be obtained from string s2 if we can remove some characters from s2 such that it becomes s1. For example, “abc” can be obtained from “abdbec” based on our definition, but it can not be obtained from “acbbe”.

You are given two non-empty strings s1 and s2 (each at most 100 characters long) and two integers 0 ≤ n1 ≤ 106 and 1 ≤ n2 ≤ 106. Now consider the strings S1 and S2, where `S1=[s1,n1]` and `S2=[s2,n2]`. Find the maximum integer M such that `[S2,M]` can be obtained from `S1`.

**Example**  

```
Input:
s1="acb", n1=4
s2="ab", n2=2

Return:
2
```

#### 思路
这道题考虑这样两种情况：

* s2在s1中的出现有一定的循环规律，比如s1=“eacfgeae”，n1=5，s2=“ea”，这样每2个s1可以出现4个s2，且有一个s2被分割了。这里有种特殊情况就是刚好没被分割，但是做法和被分割的一样。
* s2在s1中没有出现规律。

这样，我们只要遍历n1次s1，找到有没有循环规律，如果有，则跳出，没有则执行到结束。

时间复杂度：O(n1*len(s1))

#### 代码

```cpp
class Solution {
public:
    int getMaxRepetitions(string s1, int n1, string s2, int n2) {
        int len1 = s1.size();
        int len2 = s2.size();
        int a[n1]{};
        int b[len2+1]{};
        int j = 0;
        int m = 0;
        int k = 1;
        for(;k<=n1;k++){
            for(auto c:s1){
                if(c==s2[j])j++;
                if(j==len2){
                    m++;
                    j = 0;
                }
            }
            a[k] = m;
            if(!j||b[j])break;
            b[j] = k;
        }
        if(n1==k)return a[n1]/n2;
        n1 -= b[j];
        return (n1 / (k - b[j]) * (a[k] - a[b[j]]) + a[n1 % (k - b[j]) + b[j]]) / n2;
    }
};
```

# 472. Concatenated Words

#### 题目

Given a list of words (without duplicates), please write a program that returns all concatenated words in the given list of words.

A concatenated word is defined as a string that is comprised entirely of at least two shorter words in the given array.

**Example:**

```
Input: ["cat","cats","catsdogcats","dog","dogcatsdog","hippopotamuses","rat","ratcatdogcat"]

Output: ["catsdogcats","dogcatsdog","ratcatdogcat"]

Explanation: "catsdogcats" can be concatenated by "cats", "dog" and "cats"; 
 "dogcatsdog" can be concatenated by "dog", "cats" and "dog"; 
"ratcatdogcat" can be concatenated by "rat", "cat", "dog" and "cat".
```

**Note:**  

1. The number of elements of the given array will not exceed **10,000**.
2. The length sum of elements in the given array will not exceed **600,000**.
3. All the input string will only include lower case letters.
4. The returned elements order does not matter.

#### 思路1

Trie思路：  

* 先对所有词建立一棵Trie树，把所有单词路径都存进去。
* 用dfs遍历每个词，看这个词能否被拆成两个或两个以上的词。这里要注意，最后一个Test会TLE，所以可以用静态空间代替堆空间来建立Trie树，以减少内存使用。

#### 代码1

```cpp
struct Node{
    bool isword;
    Node* child[26];
    Node(){
        isword = false;
        memset(child, NULL, sizeof(Node*)*26);
    }
};
class Solution {
public:
    vector<string> findAllConcatenatedWordsInADict(vector<string>& words) {
        vector<string> res;
        static Node node_pool[60000], *root = node_pool;
        memset(node_pool, 0, sizeof(node_pool));
        int count = 1;
        Node* r;
        for(auto i:words){
            r = root;
            for(int j=0;j<i.size();j++){
                if(r->child[i[j]-'a']==nullptr)
                    r->child[i[j]-'a'] = &node_pool[count++];
                r = r->child[i[j]-'a'];
            }
            r->isword = true;
        }
        for(auto word:words){
            if(dfs(word, root, 0, 0))
                res.push_back(word);
        }
        return res;
    }
    bool dfs(string& word, Node* r, int ind, int count){
        if(ind>=word.size()){
            if(count>=2)
                return true;
            return false;
        }
        Node* root = r;
        int i = ind;
        while(i<word.size()&&r->child[word[i]-'a']!=nullptr){
            r = r->child[word[i]-'a'];
            i++;
            if(r->isword&&dfs(word, root, i, count+1)){
                return true;
            }
        }
        return false;
    }
};
```

#### 思路2

DP思路：

* 对每个词先进行排序，短的放前面，长的放后面。
* 对每个词进行遍历，看这个词能否被拆成多个词，如果可以，则存入res，如果不能，则作为单词存入set

#### 代码2

```cpp
class Solution {
public:
    vector<string> findAllConcatenatedWordsInADict(vector<string>& words) {
        vector<string> res;
        unordered_set<string> s;
        auto comp = [](string a, string b){return a.size()<b.size();};
        sort(words.begin(), words.end(), comp);
        for(auto word:words){
            if(dfs(word, s))
                res.push_back(word);
            else
                s.insert(word);
        }
        return res;
    }
    bool dfs(string& word, unordered_set<string>& s){
        if(word=="")
            return false;
        vector<bool> dp(word.size()+1, false);
        dp[0] = true;
        for(int i=1;i<=word.size();i++){
            for(int j=i-1;j>=0;j--){
                if(dp[j]&&s.find(word.substr(j, i-j))!=s.end()){
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[word.size()];
    }
};
```

# 517. Super Washing Machines

#### 题目

You have n super washing machines on a line. Initially, each washing machine has some dresses or is empty.

For each move, you could choose any m (1 ≤ m ≤ n) washing machines, and pass one dress of each washing machine to one of its adjacent washing machines at the same time .

Given an integer array representing the number of dresses in each washing machine from left to right on the line, you should find the minimum number of moves to make all the washing machines have the same number of dresses. If it is not possible to do it, return -1.

#### 思路

把所有机器的move最小和转化成每台机器的必需move的最大值（保证每台机器在调整平衡的时候被尽可能多的使用，表示不断重复利用），那么这些机器的最大值中的最大值就是所有机器的move最小值。

对于一台机器来说，计算它左边的机器缺多少（或多多少）件衣服l，右边同样记为r。之后要让左右两边的数平衡，需要考虑l和r的取值：

1. `l >= 0 && r >= 0` 则表示左右都缺衣服，那么需要当前机器输送衣服给左右两边的机器。当前机器（这种情况只有当前机器需要操作）需要操作l+r次；
2. `l < 0 && r >= 0` 则表示左边多，右边少，需要操作`max(l, r)`次；
3. `l >= 0 && r < 0` 则表示左边少，右边多，需要操作`max(l, r)`次；
4. `l < 0 && r < 0`表示两边都多衣服，则需要操作`max(l, r)`次，即把多余的衣服给当前机器。

#### 代码

```cpp
class Solution {
public:
    int findMinMoves(vector<int>& machines) {
        int len = machines.size();
        int sum[len+1];
        sum[0] = 0;
        for(int i=0;i<len;i++)
            sum[i+1] = machines[i] + sum[i];
        if(sum[len] % len != 0) return -1;
        int avg = sum[len] / len;
        int l, r;
        int res = 0;
        for(int i=0;i<len;i++) {
            l = i * avg - sum[i];
            r = (len-i-1) * avg - (sum[len] - sum[i+1]);
            if(l >= 0 && r >= 0)
                res = max(res, l + r);
            else
                res = max(res, max(abs(l), abs(r)));
            // cout<<res<<endl;
        }
        return res;
    }
};
```



# 502. IPO

#### 题目

Suppose LeetCode will start its IPO soon. In order to sell a good price of its shares to Venture Capital, LeetCode would like to work on some projects to increase its capital before the IPO. Since it has limited resources, it can only finish at most k distinct projects before the IPO. Help LeetCode design the best way to maximize its total capital after finishing at most k distinct projects.

You are given several projects. For each project i, it has a pure profit Pi and a minimum capital of Ci is needed to start the corresponding project. Initially, you have W capital. When you finish a project, you will obtain its pure profit and the profit will be added to your total capital.

To sum up, pick a list of at most k distinct projects from given projects to maximize your final capital, and output your final maximized capital.


**Example 1:**

```
Input: k=2, W=0, Profits=[1,2,3], Capital=[0,1,1].

Output: 4

Explanation: Since your initial capital is 0, you can only start the project indexed 0.
After finishing it you will obtain profit 1 and your capital becomes 1.
With capital 1, you can either start the project indexed 1 or the project indexed 2.
Since you can choose at most 2 projects, you need to finish the project indexed 2 to get the maximum capital.
Therefore, output the final maximized capital, which is 0 + 1 + 3 = 4.
```

Note:

1. You may assume all numbers in the input are non-negative integers.
2. The length of Profits array and Capital array will not exceed 50,000.
3. The answer is guaranteed to fit in a 32-bit signed integer.

#### 思路

此题的思路就是贪婪算法+优先级队列。

一开始我有W块钱，只能买W以及以下的东西，但是买了以后不花钱（因为是pure profit），且可以净赚`Profit[i]`块，但只能买k个，且不重样。那么一个简单的思路就是我每次买的时候，先扫描我能买的，然后选择可以获得的净利润`Profit[i]`，循环k遍，就得到最大的利润。

但是上述做法的复杂度为`O(n*k)`，是暴力解法。其实可以把一开始的数据分在两个容器里，用优先级队列存放我能够的买的东西的`Profit[i]`，另一个用按`Capital`排序的vector存放我暂时买不了的东西的`pair(Capital[i], Profit[i])`。这样我每次买的时候，从优先级队列中取出最大利益值，并更新W，之后再根据更新过的W从vector中选择我现在可以买的东西的`Profit[i]`到优先级队列中。

#### 代码

```cpp
class Solution {
public:
    int findMaximizedCapital(int k, int W, vector<int>& Profits, vector<int>& Capital) {
        int len = Profits.size();
        priority_queue<int> pq;
        vector<pair<int, int>> cp;
        for(int i=0;i<len;i++) {
            if(Profits[i]>0) {
                if(W>=Capital[i])
                    pq.push(Profits[i]);
                else
                    cp.push_back(pair<int, int>(Capital[i], Profits[i]));
            }
        }
        auto f = [](pair<int, int> a, pair<int, int> b) {return a.first<b.first;};
        sort(cp.begin(), cp.end(), f);
        while(k--) {
            if(pq.empty())
                return W;
            W += pq.top();
            pq.pop();
            int i = 0;
            while(!cp.empty()&&cp[i].first<=W) {
                pq.push(cp[i].second);
                cp.erase(cp.begin());
            }
        }
        return W;
    }
};
```


# 514. Freedom Trail

#### 题目

[Freedom Trail](https://leetcode.com/problems/freedom-trail/?tab=Description)


#### 思路

比较正式的思路就是DP，每次我们考虑两个多源问题：

1. 从上面的哪个位置跳到当前匹配的位置（对应多个来源）；
2. 从上面的位置跳到匹配key中当前字符的ring所在的位置（对应多个目的）。

其实就是从ring中的哪个上一目标位置跳到ring中的哪个当前目标位置。

这样，我们可以用`dp[i][j]`来存储目标`key[i]`对应的跳到`ring[j]`的步数，可见`key[i] == ring[j]`，而从多个源跳入当前位置`j`，则需要遍历多个源，然后选择`dp[i][j]`为跳入`j`的步数最小值。

当然也可以用DFS方法来遍历每一种可能。


#### DP代码
```cpp
class Solution {
public:
    int findRotateSteps(string ring, string key) {
        int m = ring.size();
        int n = key.size();
        if(m < 1 || n < 1)
            return 0;
        int dp[n+1][m];
        memset(dp, 0, sizeof(dp));
        set<int> pre;
        pre.insert(0);
        for(int i=0;i<n;i++) {
            set<int> tmp;
            for(int j=0;j<m;j++) {
                if(key[i] == ring[j]) {
                    // cout<<j<<ring[j]<<endl;
                    for(auto k : pre) {
                        int step = abs(j - k);
                        step = min(step, m - step);
                        int jump = dp[i][k] + step + 1;
                        if(dp[i+1][j] != 0)
                            dp[i+1][j] = min(dp[i+1][j], jump);
                        else
                            dp[i+1][j] = jump;
                        // cout<<dp[i+1][j]<<endl;
                    }
                    tmp.insert(j);
                }
            }
            pre = tmp;
        }
        int res = INT_MAX;
        for(int i=0;i<m;i++) {
            // cout<<dp[n][i];
            if(ring[i] == key[n-1])
                res = min(res, dp[n][i]);
        }
        return res;
    }
};
```
