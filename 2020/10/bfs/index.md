# BFS


# BFS

## 例题

```java
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node start, Node target) {
    Queue<Node> q; // 核心数据结构
    Set<Node> visited; // 避免走回头路

    q.offer(start); // 将起点加入队列
    visited.add(start);
    int step = 0; // 记录扩散的步数

    while (q not empty) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            Node cur = q.poll();
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列 */
            for (Node x : cur.adj())
                if (x not in visited) {
                    q.offer(x);
                    visited.add(x);
                }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```

例题：二叉树的最小深度（类似层序遍历）

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int minDepth(TreeNode root) {
        if(root == null) {
            return 0;
        }
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        int depth = 1;
        while (!queue.isEmpty()) {
            int len = queue.size();
            for(int i=0; i<len; i++) {
                TreeNode node = queue.poll();
                if(node.left == null && node.right == null) {
                    return depth;
                }
                if(node.left != null) {
                    queue.offer(node.left);
                }
                if(node.right != null) {
                    queue.offer(node.right);
                }
            }
            depth++;
        }
        return depth;
        
    }
}
```

再来看看DFS的解法

```java
class Solution {
    public int minDepth(TreeNode root) {
        if (root == null)
            return 0;
        if (root.left == null)
            return 1 + minDepth(root.right);
        if (root.right == null) {
            return 1 + minDepth(root.left);
        }
        return 1 + Math.min(minDepth(root.left), minDepth(root.right));
    }
} 
```

使用BFS容易寻找最短距离，但DFS是考递归的堆栈记录走过的路径。

> DFS是线，BFS是面

例如：单词接龙

```java
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
      // base case
        if(!wordList.contains(endWord)) {
            return 0;
        }
      // 核心数据结构
        Queue<String> queue = new LinkedList<>();
      // 避免回头路
        Set<String> visited = new HashSet<>();
        queue.offer(beginWord);
        visited.add(beginWord);
      // 记录答案
        int count = 0;
        while(!queue.isEmpty()) {
            int len = queue.size();
            count ++;
            for(int i=0; i<len; i++) {
                String start = queue.poll();
              // 从单词集中寻找是否可以转换的
                for(String s : wordList) {
                  // 若已经遍历过
                    if(visited.contains(s)) {
                        continue;
                    }
                  // 若不可以转换
                    if(!isConvert(start, s)) {
                        continue;
                    }
                  // 判断是否到达终点
                    if(s.equals(endWord)) {
                        return count + 1;
                    }
                    visited.add(s);
                    queue.offer(s);
                }
            }
        }
        return 0;
    }

    public boolean isConvert(String start, String s) {
        if(s.length() != start.length()) {
            return false;
        }
        int count = 0;
        for(int i=0; i<s.length(); i++) {
            if(start.charAt(i) != s.charAt(i)) {
                count ++;
                if(count > 1) {
                    return false;
                }
            }
        }       
        return true;
    }
}
```

例：leetcode752

> 你有一个带有四个圆形拨轮的转盘锁。每个拨轮都有10个数字： '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' 。每个拨轮可以自由旋转：例如把 '9' 变为  '0'，'0' 变为 '9' 。每次旋转都只能旋转一个拨轮的一位数字。
>
> 锁的初始数字为 '0000' ，一个代表四个拨轮的数字的字符串。
>
> 列表 deadends 包含了一组死亡数字，一旦拨轮的数字和列表里的任何一个元素相同，这个锁将会被永久锁定，无法再被旋转。
>
> 字符串 target 代表可以解锁的数字，你需要给出最小的旋转次数，如果无论如何不能解锁，返回 -1。

遍历 “0000” 的相邻节点，拨动数字只能向上或向下因此“1000”和“9000”；“0100”和“0900“…及还是一个求最短距离的问题。

```java
class Solution {
    public String rollUp(String s, int j) {
        char[] ch = s.toCharArray();
        if (ch[j] == '9')
            ch[j] = '0';
        else
            ch[j] += 1;
        return new String(ch);
    }

    public String rollDown(String s, int j) {
        char[] ch = s.toCharArray();
        if(ch[j] == '0') {
            ch[j] = '9';
        } else {
            ch[j] -= 1;
        }
        return new String(ch);
    }

    public int openLock(String[] deadends, String target) {
        Queue<String> queue = new LinkedList<>();
        Set<String> dead = new HashSet<>();
        Set<String> visited = new HashSet<>();
        for(String s : deadends) {
            dead.add(s);
        }
        queue.add("0000");
        visited.add("0000");
        int count = 0;
        while(!queue.isEmpty()) {
            int len = queue.size();
            for(int i=0; i<len; i++) {
                String str = queue.poll();
                if(dead.contains(str)) {
                    continue;
                }
              // 终点
                if(target.equals(str)) {
                    return count;
                }
								// 向该节点的四周扩散
                for(int j=0; j<4; j++) {
                    String up = rollUp(str, j);
                    if(!visited.contains(up)) {
                        queue.offer(up);
                        visited.add(up);
                    }
                    String down = rollDown(str, j);
                    if(!visited.contains(down)) {
                        queue.offer(down);
                        visited.add(down);
                    }
                }

            }
            count ++;
        }
        return -1;
    }
}
```

## 双向BFS优化

> **传统的 BFS 框架就是从起点开始向四周扩散，遇到终点时停止；而双向 BFS 则是从起点和终点同时开始扩散，当两边有交集的时候停止**。

**不过，双向 BFS 也有局限，因为你必须知道终点在哪里**。比如二叉树最小高度的问题，你一开始根本就不知道终点在哪里，也就无法使用双向 BFS；但是第二个密码锁的问题，是可以使用双向 BFS 算法来提高效率的。

```java
class Solution {
    public String rollUp(String s, int j) {
        char[] ch = s.toCharArray();
        if (ch[j] == '9')
            ch[j] = '0';
        else
            ch[j] += 1;
        return new String(ch);
    }

    public String rollDown(String s, int j) {
        char[] ch = s.toCharArray();
        if(ch[j] == '0') {
            ch[j] = '9';
        } else {
            ch[j] -= 1;
        }
        return new String(ch);
    }

    public int openLock(String[] deadends, String target) {
      // 用集合不用队列，可快速进行判断
        Set<String> q1 = new HashSet<>();
        Set<String> q2 = new HashSet<>();
        Set<String> dead = new HashSet<>();
        Set<String> visited = new HashSet<>();
        for(String s : deadends) {
            dead.add(s);
        }
        q1.add("0000");
      // q2 从终点开始
        q2.add(target);
        visited.add("0000");
        int count = 0;
        while(!q1.isEmpty() && !q2.isEmpty()) {
          // 用temp存储扩散结果，因为哈希集合在遍历时不可修改
            Set<String> temp = new HashSet<>();

          // 扩散q1集合
            for(String cur : q1) {
                if(dead.contains(cur)) {
                    continue;
                }
              // 相遇
                if(q2.contains(cur)) {
                    return count;
                }
                visited.add(cur);

              // 将一个节点未遍历的相邻节点加入集合
                for(int j=0; j<4; j++) {
                    String up = rollUp(cur, j);
                    if(!visited.contains(up)) {
                        temp.add(up);
                    }                
                    String down = rollDown(cur, j);
                    if(!visited.contains(down)) {
                        temp.add(down);
                    }
                }
            }
          // 交换遍历
            q1 = q2;
            q2 = temp;
            count ++;
        }
        return -1;
    }
}
```


