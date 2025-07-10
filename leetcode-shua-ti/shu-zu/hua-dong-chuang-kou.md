# 滑动窗口

参考[我写了首诗，把滑动窗口算法算法变成了默写题](https://labuladong.gitee.io/algo/2/21/60/)

滑动窗口算法主要用来解决包含类问题，比如 s1 是否包含 s2 的排列或者子串等。

go 模板：

```go
func minWindow(s string, t string) string {
    need, window := make(map[byte]int), make(map[byte]int)
    // 初始化目标 map
    for i := range t {
        need[t[i]]++
    }
    n := len(s)
    left, right := 0, 0
    // 匹配上的字符
    match := 0
    for right < n {
      	// c 是将移入窗口的字符
        c := s[right]
      	// 右移窗口
        right++
     	 	// 进行窗口内数据的一系列更新
        ...

        /*** debug 输出的位置 ***/
        fmt.Printf("window: [%d, %d)\n", left, right);
        /********************/
      
      	// 判断左侧窗口是否要收缩
        for window needs shrink {
          	// b 是将移出窗口的字符
            b := s[left]
						// 左移窗口
            left++
          	// 进行窗口内数据的一系列更新
            ...
        }
    }
}
```

本文要介绍的题目有：

[76. 最小覆盖子串（困难）](https://leetcode-cn.com/problems/minimum-window-substring)

[567. 字符串的排列（中等）](https://leetcode-cn.com/problems/permutation-in-string)

[438. 找到字符串中所有字母异位词（中等）](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string)

[3. 无重复字符的最长子串（中等）](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters)

## 代码

### 最小覆盖子串

```go
func minWindow(s string, t string) string {
    need, window := make(map[byte]int), make(map[byte]int)
    // 初始化目标 map
    for i := range t {
        need[t[i]]++
    }
    n := len(s)
    left, right := 0, 0
    start, end, length := 0, 0, math.MaxInt32
    // 匹配上的字符
    match := 0
    for right < n {
        c := s[right]
        if strings.Contains(t, string(c)) {
            window[c]++
            // 表示同一个字符的数量相等了，说明这个字符就匹配了
            if window[c] == need[c] {
                match++
            }
        }
        right++

        // 如果 t 里的字符全部匹配了则缩小窗口, left++
        // 注意这里应该是与目标 map 的长度比较，因为 t 里可能有重复的字符
        for match == len(need) {
            if length > right - left {
                length = right - left
                start, end = left, right
            }
            b := s[left]
            
            // 这里也要判断 b 是否在 t 里，不然两个 map 都是 0 也是会 match--
            // 如果 left 位置刚好是匹配的字符，那么窗口缩小，match 就得 -1
            if strings.Contains(t, string(b)) && window[b] == need[b] {
                match--
            }
            window[b]--
            left++
        }
    }
    if start == end {
        return ""
    }
    return s[start:end]
}
```

### 字符串的排列

```go
func checkInclusion(s1 string, s2 string) bool {
    need, window := make(map[byte]int), make(map[byte]int)

    for i := range s1 {
        need[s1[i]] ++
    } 
    left, right := 0, 0
    var match int
    for right < len(s2) {
        c := s2[right]
        if strings.Contains(s1, string(c)) {
            window[c]++
            if window[c] == need[c] {
                match++
            }
        }
        right++ 

        // 长度必须与 s1 相等，不然就不是 s1 的排列
        if right - left == len(s1) {
            if match == len(need) {
                return true
            }
            b := s2[left]
            if strings.Contains(s1, string(b)) && window[b] == need[b] {
                match--
            }
            window[b]--
            left++
        }
    }
    return false
}
```

### 找到字符串中所有字母异位词

```go
func findAnagrams(s string, p string) []int {
    var res []int
    need, window := make(map[byte]int), make(map[byte]int)

    for i := range p {
        need[p[i]]++
    }

    left, right := 0, 0
    match := 0

    for right < len(s) {
        c := s[right]
        if strings.Contains(s, string(c)) {
            window[c]++
            if window[c] == need[c] {
                match++
            }
        }
        right++

        if right - left == len(p) {
            if match == len(need) {
                res = append(res, left)
            }
            b := s[left]
            if strings.Contains(p, string(b)) {
                if window[b] == need[b] {
                    match--
                }
                window[b]--
            }
            left++
        }
    }
    return res
}
```

### 无重复字符的最长子串

```go
func lengthOfLongestSubstring(s string) int {
    left, right, length := 0, 0, 0
    window := make(map[byte]int)

    for right < len(s) {
        c := s[right]
        window[c]++
        right++

        for window[c] > 1 {
            b := s[left]
            window[b]--
            left++
        }
        if length < right-left {
            length = right-left
        }
    }
    return length
}
```
