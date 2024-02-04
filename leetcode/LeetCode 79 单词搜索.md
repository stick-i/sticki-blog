# LeetCode 79. 单词搜索

原题链接：[79. 单词搜索](https://leetcode.cn/problems/word-search/)

题目难度：中等

## 题目描述

给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/04/word2.jpg)

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/11/04/word-1.jpg)

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "SEE"
输出：true
```

**示例 3：**

![img](https://assets.leetcode.com/uploads/2020/10/15/word3.jpg)

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCB"
输出：false
```

 

**提示：**

- `m == board.length`
- `n = board[i].length`
- `1 <= m, n <= 6`
- `1 <= word.length <= 15`
- `board` 和 `word` 仅由大小写英文字母组成

 

**进阶：**你可以使用搜索剪枝的技术来优化解决方案，使其在 `board` 更大的情况下可以更快解决问题？



## 解题思路

这道题可以用深度优先搜索来解，题目的意思是行走过程中拼成的单词要等于题目给出的单词，然后让我们找一条这样的行走路径。

可以这样去转化：

- 起点仍然是固定的，就是在网格中与单词第一个字母相同的格子，可能有多个起点，可以通过遍历网格找到。
- 从起点往四周去寻找，哪个方向能走取决于网格的内容是否与单词的字母相同，但同一个格子不能走两次。

这其实就很容易看出来了，就是一道迷宫题，深度优先搜索就可以应对。

假设我们现在从起点（记为x , y）出发，当前要找的内容为单词的第一个字母（第几个记为index），那么具体思路：

1. 设置递归的终止条件（按序号判断）

   1. 如果当前格子的值不等于单词的第一个字母，那肯定没戏了，直接返回false吧！
   2. 如果当前要找的字母已经是单词的最后一个字母了，那说明我们已经找完了呀，那直接返回true了。

2. 没满足终止条件，那说明我们已经走在当前这个格子上了，而且还得往后继续走，那我们得标记一下，防止后面重复走了这个格子。就创建一个`flag`二维数组好了，大小刚好为网格的大小，可以标记到每一个网格！

   现在我们把`flag[x][y]`设置为true。

3. 好了，那我们得继续往后面探索了，先看看四周的格子能走吗？

   - 首先得考虑下我们是不是在地图边界了，如果在地图边界的话，那有些方向就不能走了，不然我就越界了，这很不好。
   - 其次我们还得考虑下哪些方向的格子是走过的，哪些是没走过的。记得我们刚刚用 `flag` 二维数组来记录了所有走过的格子，所以我只要判断下四周的位置在`flag`里哪些的值是`false`，只有这些地方是可以进一步探索的。

4. 我已经确定好要走哪些格子了，那我得往后走了。递归的时候，我得把下一个位置的坐标传过去，还有标记着已经走过路径的`flag`数组。也千万别忘了告诉下一层递归它要找的字母，就是我刚找到的那个字母的下一个（index + 1）。

5. 递归的过程中会重复我们上面的步骤，当递归结束回来的时候，也会带回来我们在第一步设置的返回条件，找到了就是true，没找到就是false。

   那虽然我们同时往几个方向都走过了，但只要其中一个方向返回true，就说明有一条路径是符合条件的，那我们就返回true，要是都没找到，那我们就返回false。

==这段解题思路比较冗长且无聊，建议结合下面的核心代码一起理解：==

**核心代码：**

```java
char[][] board; // 网格，由题目提供数据
String word; // 要找的单词，题目提供

int[] sitx = {-1,0,1,0}; // 用来走格子的方向，可以减少重复代码
int[] sity = {0,-1,0,1};

// x和y表示当前坐标
// index表示当前已经走到第几个字母了
// flag表示已经走过的路径，其中走过的值为true
boolean find(int x, int y, int index ,boolean[][] flag) {
    // 如果当前格子的字母不是要找的，直接返回false
    if (word.charAt(index) != board[x][y]) {
        return false;
    }
    // 已经找到最后一个字母了，直接返回true
    if (index == word.length() - 1) {
        return true;
    }
    // 当前这个格子已经走了，给flag赋值
    flag[x][y] = true;
    for (int i = 0 ; i < 4; i++) {
        // 下一步要走的位置nextx、nexty
        int nx = x + sitx[i];
        int ny = y + sity[i];

        // 判断是否超过边界，以及是否已经走过
        boolean isEdge = nx == -1 || nx == board.length || ny == -1 || ny == board[0].length;
        if (isEdge || flag[nx][ny]) continue;

        // 如果没有走过，就往这个方向走
        boolean isFand = find(nx, ny, index + 1, flag);
        // 如果找到了直接返回
        if (isFand) return true;

    }
    // 没有找到格式的路径，返回上一级之前记得清理flag
    flag[x][y] = false;
    return false;
}
```

好，交一发！当然通过了，但速度不快，应该还能优化，但我看官方题解，却没有看到优化方法。

![image-20230115212007372](image/image-20230115212007372.png)

## 完整解题代码 Java

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        int n = board.length;
        int m = board[0].length;
        this.word = word;
        this.board = board;
        // 寻找起点，并开启第一步
        for (int i = 0 ; i < n; i++ ) {
            for (int j = 0 ; j < m ; j++) {
                if (board[i][j] == word.charAt(0)) {
                    boolean isFand = find(i, j, 0, new boolean[n][m]);
                    if (isFand) return true;
                }
            }
        }

        return false;
    }

    // 把上面的核心代码放在这即可

}
```

由于上面已经贴了核心代码了，所以这里我就只贴了其他的部分，如果要提交到LeetCode进行测试的话，记得要把前面的核心代码贴过来噢。

