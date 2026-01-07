# LeetCode

## 哈希表

### [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。

示例 1:

```
输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出: [["bat"],["nat","tan"],["ate","eat","tea"]]
```

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        // key: 排序后的字符串, value: 原始字符串集合
        unordered_map<string, vector<string>> mp;
        
        for (string& s : strs) {
            string t = s;
            sort(t.begin(), t.end()); // 排序作为哈希表的键
            mp[t].push_back(s);       // 将原词归类
        }
        
        vector<vector<string>> result;
        for (auto it = mp.begin(); it != mp.end(); ++it) {
            result.push_back(it->second);
        }
        
        return result;
    }
};

// 进阶版
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        // 自定义对 array<int, 26> 类型的哈希函数
        auto arrayHash = [fn = hash<int>{}] (const array<int, 26>& arr) -> size_t {
            return accumulate(arr.begin(), arr.end(), 0u, [&](size_t acc, int num) {
                return (acc << 1) ^ fn(num);
            });
        };

        unordered_map<array<int, 26>, vector<string>, decltype(arrayHash)> mp(0, arrayHash);
        for (string& str: strs) {
            array<int, 26> counts{};
            int length = str.length();
            for (int i = 0; i < length; ++i) {
                counts[str[i] - 'a'] ++;
            }
            mp[counts].emplace_back(str);
        }
        vector<vector<string>> ans;
        for (auto it = mp.begin(); it != mp.end(); ++it) {
            ans.emplace_back(it->second);
        }
        return ans;
    }
};
```

### [128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

**示例 1：**

```
输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。
```

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> numSet;
        for (int num : nums) {
            numSet.insert(num);
        }

        int maxStreak = 0;
        for (int num : numSet) {
            int curStreak = 1;
            // 只有当 num-1 不存在时，才说明 num 是一个连续序列的起点
            if (!numSet.contains(num - 1)) {
                while (numSet.contains(++num)) {
                    curStreak++;
                }
            }
            maxStreak = max(maxStreak, curStreak);
        }

        return maxStreak;
    }
};
```

## 双指针

### [283. 移动零](https://leetcode.cn/problems/move-zeroes/)

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int j = 0;
        int n = nums.size();
        
        // 类似**快排**中的分区逻辑
        for (int i = 0; i < n; i++) {
            if (nums[i] != 0) {
                swap(nums[i], nums[j++]);
            }
        }
    }
};
```

### [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/)

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

![img](D:\Projects\NOTES\images\rainwatertrap.png)

```cpp
class Solution {
public:
    // 核心原理：当前位水量 = min(左边最高, 右边最高) - 当前高度
    int trap(vector<int>& height) {
        int ans = 0;
        int left = 0, right = height.size() - 1;
        int leftMax = 0, rightMax = 0;
        while (left < right) {
            leftMax = max(leftMax, height[left]);
            rightMax = max(rightMax, height[right]);
            if (height[left] < height[right]) {
                // 左边较短，由于 rightMax 至少是 height[right]，所以我们确定对于左侧来说，右边的屏障一定足够高。
                // 此时限制水位的“短板”必然是 leftMax。
                ans += leftMax - height[left];
                ++left;
            } else {
                // 反之，如果右边较短，限制水位的“短板”必然是 rightMax
                ans += rightMax - height[right];
                --right;
            }
        }
        return ans;
    }
};
```

## 滑动窗口

#### [438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

**示例 1:**

```
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。
```

```cpp
class Solution {
   public:
    vector<int> findAnagrams(string s, string p) {
        vector<int> ans;
        int slen = s.length();
        int plen = p.length();

        if (slen < plen) {
            return vector<int>();
        }

        // windowBalance 记录 s 窗口内字母频率与 p 字母频率的差值。
        vector<int> windowBalance = vector<int>(26);
        for (int i = 0; i < plen; i++) {
            windowBalance[s[i] - 'a']++;
            windowBalance[p[i] - 'a']--;
        }

        // 用一个int变量代表 windowBalance 数组中非零元素的个数
        int mismatchCount = 0;
        for (int j = 0; j < 26; j++) {
            if (windowBalance[j] != 0) {
                mismatchCount++;
            }
        }

        if (mismatchCount == 0) {
            ans.push_back(0);
        }

        // 滑动过程，起始索引是i+1
        for (int i = 0; i < slen - plen; i++) {
            // 把左端滑出窗口，判断mismatchCount变化
            windowBalance[s[i] - 'a']--;
            if (windowBalance[s[i] - 'a'] == 0) {
                mismatchCount--;
            } else if (windowBalance[s[i] - 'a'] == -1) {
                mismatchCount++;
            }
            // 把右端滑入，判断mismatchCount变化
            windowBalance[s[i + plen] - 'a']++;
            if (windowBalance[s[i + plen] - 'a'] == 0) {
                mismatchCount--;
            } else if (windowBalance[s[i + plen] - 'a'] == 1) {
                mismatchCount++;
            }

            // 符合条件否
            if (mismatchCount == 0) {
                ans.push_back(i + 1);
            }
        }
        return ans;
    }
};
```



## 子串

### [560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 *该数组中和为 `k` 的子数组的个数* 。

子数组是数组中元素的连续非空序列。

**示例 1：**

```
输入：nums = [1,1,1], k = 2
输出：2
```

```cpp
class Solution {
public:
    // 前缀和解决“连续子数组和”类问题
    int subarraySum(std::vector<int>& nums, int k) {
        // prefix-sum frequency map; mp[0]=1 for the empty prefix
        std::unordered_map<int, int> mp{{0, 1}};  
        int prefix = 0, count = 0;

        for (int num : nums) {
            prefix += num;                 // running prefix sum
            count  += mp[prefix - k];      // how many previous prefixes make (prefix-k)
            ++mp[prefix];                  // record current prefix
        }
        return count;
    }
};
```

### [239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。

 

**示例 1：**

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

```cpp
class Solution {
public:
    // 单调（递减）队列（不用map，前者O(n)后者O(nlogn)）
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> result;
        deque<int> indexDeque;			// 存索引
        
        auto push = [&](int i) {
            // 保持单调递减
            while (!indexDeque.empty() && nums[i] >= nums[indexDeque.back()]) {
                indexDeque.pop_back();
            }
            indexDeque.push_back(i);
        };
        
        for (int i = 0; i < n; ++i) {
            // 入队
            push(i);
            
            // 过期处理，检查队头元素是否已超出当前窗口范围
 			if (indexDeque.front() <= i - k) {
                indexDeque.pop_front();
            }
            
			// 只有窗口完整（达到 k 个元素）后才开始记录
            if (i >= k - 1) {
                result.push_back(nums[indexDeque.front()]);
            }
        }
        
        return result;
    }
};
```

### [76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)

给定两个字符串 `s` 和 `t`，长度分别是 `m` 和 `n`，返回 s 中的 **最短窗口 子串**，使得该子串包含 `t` 中的每一个字符（**包括重复字符**）。如果没有这样的子串，返回空字符串 `""`。

测试用例保证答案唯一。

**示例 1：**

```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
解释：最小覆盖子串 "BANC" 包含来自字符串 t 的 'A'、'B' 和 'C'。
```

```cpp
class Solution {
public:
    // **贪心法**：先不断扩张窗口右侧，满足需求后再收缩窗口左侧（重点关注"Step-4"）
    std::string minWindow(const std::string& s, const std::string& t)
    {
        if (s.empty() || t.empty()) return "";

        std::unordered_map<char, int> need;
        for (char c : t) ++need[c];

        std::size_t needCnt = t.size();          // how many chars still missing
        std::size_t i = 0;                       // left boundary of window
        std::size_t bestL = 0, bestR = SIZE_MAX; // record the best window [bestL, bestR]

        for (std::size_t j = 0; j < s.size(); ++j)
        {
            char c = s[j];
            if (need[c] > 0)                      // this char is still needed
                --needCnt;
            --need[c];                            // put char c into current window

            // Step-1: the current window already covers all chars in t
            if (needCnt == 0)
            {
                // Step-2: shrink the left side as much as possible
                while (true)
                {
                    char leftChar = s[i];
                    if (need[leftChar] == 0)      // removing it would break the coverage
                        break;
                    ++need[leftChar];             // put the char back to "needed"
                    ++i;                          // move left boundary right
                }

                // Step-3: update the best window
                if (j - i < bestR - bestL)
                {
                    bestL = i;
                    bestR = j;
                }

                // **Step-4: move left boundary one step to look for a new window** (打破现状，寻找更优解)
                ++need[s[i]];
                ++needCnt;
                ++i;
            }
        }

        return bestR == SIZE_MAX ? "" : s.substr(bestL, bestR - bestL + 1);
    }
};
```



## 普通数组



## 矩阵



## 链表



## 二叉树



## 图论



## 回溯



## 二分查找



## 堆



## 栈



## 贪心



## 动态规划



## 多维动态规划



## 技巧