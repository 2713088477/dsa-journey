# 接雨水

## 题目描述

这道题的测试链接: https://leetcode.cn/problems/trapping-rain-water/

题目的描述大致是:

给定一个高度数组height，这个代表每个柱子的高度，然后每个柱子的宽度是1。

计算按此排列的柱子，下雨之后能接多少雨水？

我们直接看官方的图，比较直观：

![](assets\接雨水官方图.png)

这个图，给定的height = [0,1,0,2,1,0,1,3,2,1,2,1]，这个答案就是6，如图蓝色的格子

## 解法1：自然智慧思考法

我们思考每个柱子，能接雨水的量与什么有关？

比如:

```txt
x
x   x
x x x
```

height = [3,1,2]

中间这个柱子，能接多少雨水？利用自然智慧想象法，是不是只能接一个水(如图：w表示水)

```txt
x
x w x
x x x
```

所以每个柱子能接雨水的量就逐渐清晰了：

> 条件1：左右两边需要有大哥罩住，即：左右两侧需要有比他高的
>
> 条件2：基于短板效应(应该是叫这个名字)，当前能存储水位的最大高度=min(左侧的最大高度，右侧的最大高度) - 当前柱子高度
>
> 这个条件2，来到某个柱子，为啥是min(左侧的最大高度，右侧的最大高度)，这个应该很好想；单看一侧的话，只要有更高的，那个更高的就能把下面的罩住，想象一下盆地。

所以我们的代码思路也出来了：

主播是学习java的，最近也在学习python,所以不出意外的话，后面的代码都会出java和py两个版本。

```java
class Solution {
    public int trap(int[] height) {
        int len = height.length;
        // leftHMax[i]: 表示来到下标为i的柱子，左侧柱子的最大高度
        // rightHMax[i]: 表示来到下标为i的柱子，右侧柱子的最大高度
        int[] leftHMax = new int[len],rightHMax = new int[len];
        //预先填充好这两个数组
        // leftHMax[0] = 0 , 0下标左侧没有柱子，所以就是0
        for(int i = 1;i<len;i++){
            leftHMax[i] = Math.max(leftHMax[i-1],height[i-1]);
        }
        // rightHMax[len-1] = 0 , len-1下标右侧没有柱子，所以就是0
        for(int i = len-2;i>=0;i--){
            rightHMax[i] = Math.max(rightHMax[i+1],height[i+1]);
        }
        // 遍历每个柱子，累加答案
        int ans = 0;
        for(int i=0;i<len;i++){
            int diffH = Math.min(leftHMax[i],rightHMax[i])-height[i];
            ans += diffH > 0 ? diffH : 0;
        }
        return ans;
    }
}
```

```python
class Solution:
    def trap(self, height: list[int]) -> int:
        length = len(height)

        # left_max[i] 表示第i个柱子，左侧柱子中的最大高度
        left_max = [0] * length
        # right_max[i] 表示第i个柱子，右侧柱子中的最大高度
        right_max = [0] * length

        # 填充左侧的最大高度
        for i in range(1,length):
            left_max[i] = max(left_max[i-1],height[i-1])
        # 填充右侧的最大高度
        for i in range(length-2,-1,-1):
            right_max[i] = max(right_max[i+1],height[i+1])

        # 累加每根柱子上方能接的雨水
        ans = 0
        for i in range(length):
            diff = min(left_max[i],right_max[i]) - height[i]
            ans += max(diff,0)
        return ans 
        

```

## 解法2：双指针优化

从解法1，我们知道了怎么计算每个柱子的能接雨水的数量。

这个遍历其实还能够更加优化，我们看py的关键代码:

```python
ans = 0
for i in range(length):
    diff = min(left_max[i],right_max[i]) - height[i]
    ans += max(diff,0)
    return ans 
```

最为关键的就是：这个答案，并不一定要**同时**知道左侧的最大高度和右侧的最大高度，我们只关注左侧的最大高度和右侧的最大高度，哪个更小

所以有一种更为精妙的遍历方式：

左指针位于头，右指针位于尾

用两个变量来记住左侧的最大高度和右侧的最大高度

看此时哪测的最大高度更小，如果左侧更小，就不需要完全知道右侧的高度；如果右侧更小，就不需要完全知道左侧的高度



举个非常容易启发的例子：

height =  [4，2，3，5，6]

left = 1，right = length -2 

当left来到下标为1的位置，也就是height[1] = 2的位置，此时左边的最大高度 leftHMax = 4

此时right来到下标为3的位置，也就是height[3]=5的位置，此时右边的最大高度 rightHMax = 6

所以我们可以确定left的能接雨水的数量，虽然我们此时完全不知道下标为1柱子的右侧柱子的最大高度，但是不需要。

 最后上代码:

```java
class Solution {
    public int trap(int[] height) {
        int length = height.length;
        int left = 1,right = length-2;
        int leftHMax = height[0],rightHMax= height[length-1];
        int ans = 0;
        while(left <= right){
            if(leftHMax < rightHMax){
                // 如果此时的leftHMax 更小，虽然我们不知道这个下标为left右侧的柱子的最大高度，但是不需要了
                // 因为此时下标为left的柱子，左右两侧最大高度柱子的最小值，一定是leftHMax
                // 利用这种遍历之法，可以更加优雅的解决这道题
                ans += Math.max(leftHMax - height[left],0);
                leftHMax = Math.max(leftHMax,height[left]);
                left+=1;
            }else{
                // 同理
                ans += Math.max(rightHMax-height[right],0);
                rightHMax = Math.max(rightHMax,height[right]);
                right -= 1;
            }
        }
        return ans;
    }
}
```

```python
class Solution:
    def trap(self, height: list[int]) -> int:
        length = len(height)
        left,right = 1,length-2
        left_max,right_max = height[0],height[length-1]
        ans = 0
        while(left<=right):
            # 如果此时左边的最大高度更小，我们不需要知道右侧的最大高度
            # 因为答案一定是左侧的最大高度更小
            # 别忘了 rightHMax 是右侧最大高度，此时rightHMax可能不是右侧最大高度，都大于leftHMax,答案只能是leftHMax
            if(left_max < right_max):
                ans += max(left_max-height[left],0)
                left_max = max(left_max,height[left])
                left += 1
            # 这个分支同理
            else:
                ans += max(right_max-height[right],0)
                right_max = max(right_max,height[right])
                right -= 1
        return ans

```



