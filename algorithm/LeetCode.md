



## 03. 数组中重复的数字

### 题目描述

找出数组中重复的数字。

> 在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

**示例 1：**

```
输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```

**限制：**

```
2 <= n <= 100000
```

### 解题思路

#### 一、 原地置换

> 思路分析

 		如果没有重复数字，那么正常排序后，数字i应该在下标为i的位置，所以思路是重头扫描数组，遇到下标为i的数字如果不是i的话，（假设为m),那么我们就拿与下标m的数字交换。在交换过程中，如果有重复的数字发生，那么终止返回ture

1、题目明确说明了数组长度为n，范围为 n-1，也就是若无重复元素排序后下标0123对应的数字就应该是0123；

2、对数组排序，其实也就是让萝卜归位，1号坑要放1号萝卜，2号坑要放2号萝卜......排序过程中查找有无重复元素。先考虑没有重复元素的情况：

```yaml
 nums[i]     3  1  0  2   萝卜   
     i       0  1  2  3   坑  
```

0号坑说我要的是0号萝卜，不要3号萝卜，所以会去和3号坑的萝卜交换，因为如果0号坑拿了3号坑的3号萝卜，那说明3号坑装的也肯定是别人家的萝卜，所以要跟3号坑换，换完是这样的：

```yaml
 nums[i]     2  1  0  3   萝卜  
     i       0  1  2  3   坑 
```

然而0号坑还没找到自己的萝卜，它不要2号萝卜，又去和2号坑的萝卜交换，换完是这样的：

```yaml
 nums[i]     0  1  2  3   萝卜 
     i       0  1  2  3   坑  
```

这时候刚好就是一一对应的，交换过程也不会出现不同坑有相同编号的萝卜。要注意交换用的是while，也就是0号坑只有拿到0号萝卜，1号坑才能开始找自己的萝卜。

3、如果有重复元素，例如：

```yaml
 nums[i]     1  2  3  2    萝卜
     i       0  1  2  3    坑
```

同样的，0号坑不要1号，先和1号坑交换，交换完这样的：

```yaml
 nums[i]     2  1  3  2    萝卜
     i       0  1  2  3    坑
```

0号坑不要2号萝卜，去和2号坑交换，交换完这样的：

```yaml
 nums[i]     3  1  2  2    萝卜
     i       0  1  2  3    坑
```

0号坑不要3号萝卜，去和3号坑交换，交换完这样的：

```yaml
 nums[i]     2  1  2  3    萝卜
     i       0  1  2  3    坑
```

0号坑不要2号萝卜，去和2号坑交换，结果发现你2号坑也是2号萝卜，那我还换个锤子，同时也说明有重复元素出现。

4、总结

其实这种原地交换就是为了降低空间复杂度，只需多要1个坑来周转交换的萝卜就好了，空间复杂度O（1）。

> 复杂度分析

- 时间复杂度 O(N) ： 遍历数组使用 O(N)，每轮遍历的判断和交换操作使用 O(1) 。
- 空间复杂度 O(1) ： 使用常数复杂度的额外空间。

> 代码实现

```java
class Solution {
    // 数组长度为n，范围为n-1，若无重复元素排序后下标0123对应的数字也应该是0123
    public int findRepeatNumber(int[] nums) {
        int temp;
        // 从头扫描数组
        for(int i=0;i<nums.length;i++){
            // 如果下标i的数字不是i，说明位置不对，需要交换
            while(nums[i]!=i){
                // 如果下标i的数字与要交换的位置的数字一样，说明有重复元素，直接返回
                if(nums[i]==nums[nums[i]]){
                    return nums[i];
                }
                // 交换，nums[i]号萝卜归位
                temp=nums[i];
                nums[i]=nums[temp];
                nums[temp]=temp;
            }
        }
        return -1;
    }
}
```

#### 二.、哈希表Set

> 思路分析

利用数据结构特点，容易想到使用哈希表（Set）记录数组的各个数字，当查找到重复数字则直接返回。

算法流程：
初始化： 新建 HashSet ，记为 dic ；
遍历数组 nums中的每个数字 num ：
当 num 在 dic 中，说明重复，直接返回 num ；
将 num 添加至 dic 中；
返回 -1 。本题中一定有重复数字，因此这里返回多少都可以。

> 复杂度分析

- 时间复杂度 O(N)： 遍历数组使用 O(N)，HashSet 添加与查找元素皆为 O(1) 。
- 空间复杂度 O(N)： HashSet 占用 O(N)大小的额外空间。

> 代码实现

Java版本

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        Set<Integer> dic=new HashSet<>();
        for(int num:nums){
            if(dic.contains(num)){
                return num;
            }
            dic.add(num);
        }
        return -1;
    }
}
```

Python版本

```python
class Solution:
    def findRepeatNumber(self, nums: [int]) -> int:
        dic = set()
        for num in nums:
            if num in dic: return num
            dic.add(num)
        return -1
```



## 04. 二维数组中的查找

### 题目描述

> 在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

**示例:**

现有矩阵 matrix 如下：
```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```
给定 target = 5，返回 true。

给定 target = 20，返回 false。

**限制：**

- 0 <= n <= 1000
- 0 <= m <= 1000

### 解题思路

#### 一、暴力

> 分析

如果不考虑二维数组排好序的特点，则直接遍历整个二维数组的每一个元素，判断目标值是否在二维数组中存在。

依次遍历二维数组的每一行和每一列。如果找到一个元素等于目标值，则返回 true。如果遍历完毕仍未找到等于目标值的元素，则返回 false。

> 代码

```Java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return false;
        }
        int rows = matrix.length, columns = matrix[0].length;
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < columns; j++) {
                if (matrix[i][j] == target) {
                    return true;
                }
            }
        }
        return false;
    }
}
```

> 复杂度分析

- 时间复杂度：O(nm)。二维数组中的每个元素都被遍历，因此时间复杂度为二维数组的大小。
- 空间复杂度：O(1)。

#### 二、线性查找

> 分析

由于给定的二维数组具备每行从左到右递增以及每列从上到下递增的特点，当访问到一个元素时，可以排除数组中的部分元素。

从二维数组的右上角开始查找。如果当前元素等于目标值，则返回` true`。如果当前元素大于目标值，则移到左边一列。如果当前元素小于目标值，则移到下边一行。

可以证明这种方法不会错过目标值。如果当前元素大于目标值，说明当前元素的下边的所有元素都一定大于目标值，因此往下查找不可能找到目标值，往左查找可能找到目标值。如果当前元素小于目标值，说明当前元素的左边的所有元素都一定小于目标值，因此往左查找不可能找到目标值，往下查找可能找到目标值。

- 若数组为空，返回` false`
- 初始化行下标为 0，列下标为二维数组的列数减 1
- 重复下列步骤，直到行下标或列下标超出边界
  - 获得当前下标位置的元素` num`
  - 如果 `num` 和 `target` 相等，返回 `true`
  - 如果 `num` 大于 `target`，列下标减 1
  - 如果 `num` 小于 `target`，行下标加 1
- 循环体执行完毕仍未找到元素等于 `target` ，说明不存在这样的元素，返回 `false`

> 代码

```Java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        // 若数组为null或者空，直接返回false
        if(matrix==null||matrix.length==0||matrix[0].length==0){
            return false;
        }
        int rows=matrix.length,columns=matrix[0].length;
        // 初始化行下标为0，列下标为二维数组的列数减1
        int row=0;
        int column=columns-1;
        // 不超出边界的情况下
        while(row<rows&&column>=0){
            int num=matrix[row][column];
            if(num==target){// 当前下标元素和target相等
                return true;
            }else if(num>target){// 当前下标元素大于target，列左移
                column--;
            }else{ // 当前下标元素小于target，行下移
                row++;
            }
        }
        return false;
    }
}
```

> 复杂度分析

- 时间复杂度：O(n+m)。访问到的下标的行最多增加 n 次，列最多减少 m 次，因此循环体最多执行 n + m 次。

- 空间复杂度：O(1)。



#### 二叉搜索树

> 分析

如下图所示，我们将矩阵逆时针旋转 45° ，并将其转化为图形式，发现其类似于 二叉搜索树 ，即对于每个元素，其左分支元素更小、右分支元素更大。因此，通过从 “根节点” 开始搜索，遇到比 `target` 大的元素就向左，反之向右，即可找到目标值 `target` 。

![Picture1.png](C:\Lenovo\qzkuan-Thinkpad\git\learning\algorithm\LeetCode.assets\6584ea93812d27112043d203ea90e4b0950117d45e0452d0c630fcb247fbc4af-Picture1.png)

“根节点” 对应的是矩阵的 “左下角” 和 “右上角” 元素，本文称之为 标志数 ，以` matrix` 中的 左下角元素 为标志数 `flag` ，则有:

1. 若 `flag` > `target` ，则 `target` 一定在 `flag` 所在 行的上方 ，即 `flag` 所在行可被消去。
2. 若 `flag` < `target` ，则 `target` 一定在 `flag` 所在 列的右方 ，即 `flag` 所在列可被消去。

**算法流程：**

1. 从矩阵 matrix 左下角元素（索引设为 (i, j) ）开始遍历，并与目标值对比：
   - 当 `matrix[i][j] > target` 时，执行 i-- ，即消去第 i 行元素；
   - 当 `matrix[i][j] < target` 时，执行 j++ ，即消去第 j 列元素；
   - 当 `matrix[i][j] = target` 时，返回 truetrue ，代表找到目标值。

2. 若行索引或列索引越界，则代表矩阵中无目标值，返回 falsefalse 。
   每轮 i 或 j 移动后，相当于生成了“消去一行（列）的新矩阵”， 索引(i,j) 指向新矩阵的左下角元素（标志数），因此可重复使用以上性质消去行（列）。

> 复杂度分析：

- 时间复杂度 O(M+N)O(M+N) ：其中，NN 和 MM 分别为矩阵行数和列数，此算法最多循环 M+NM+N 次。
- 空间复杂度 O(1)O(1) : i, j 指针使用常数大小额外空间。

> 代码：

**Python**

```python
class Solution:
    def findNumberIn2DArray(self, matrix: List[List[int]], target: int) -> bool:
        i, j = len(matrix) - 1, 0
        while i >= 0 and j < len(matrix[0]):
            if matrix[i][j] > target: i -= 1
            elif matrix[i][j] < target: j += 1
            else: return True
        return False
```

**Java**

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        int i = matrix.length - 1, j = 0;
        while(i >= 0 && j < matrix[0].length)
        {
            if(matrix[i][j] > target) i--;
            else if(matrix[i][j] < target) j++;
            else return true;
        }
        return false;
    }
}
```

C++

```c++
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target) {
        int i = matrix.size() - 1, j = 0;
        while(i >= 0 && j < matrix[0].size())
        {
            if(matrix[i][j] > target) i--;
            else if(matrix[i][j] < target) j++;
            else return true;
        }
        return false;
    }
};
```
















































