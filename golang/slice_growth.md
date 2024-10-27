# Go slice 自动扩容策略

Go slice 扩容策略其实在 Go 1.18 之后就更新过一版，过去你了解的slice扩容机制可能是这样的：

> 当**期望容量**大于旧容量的两倍，则直接**取期望容量**作为slice的新容量，否则判断slice的容量**是否小于1024**, 如果小于1024则**将旧容量x2作为最终容量**，如果大于等于1024则每次增长大约1.25倍。以上计算出的最终容量会再进行一次对齐才是真正的切片容量。

但以上情况仅适用于 Go 1.17 及之前的版本中。充分了解 slice 扩容策略或许会对编码时需要对 slice 初始化长度有帮助。



## 旧规则

直接看代码，runtime/slice.go，省略了部分代码：

```go
// 1.17及以前的版本中
// old指切片的旧容量, cap指期望的新容量
func growslice(et *_type, old slice, cap int) slice {
  	// ...
    newcap := old
    doublecap := newcap + newcap
    // 如果期望容量大于旧容量的2倍，则直接使用期望容量作为最终容量
    if cap > doublecap {
        newcap = cap
    } else {
        // 如果旧容量小于1024，则直接翻倍
        if old < 1024 {
            newcap = doublecap
        } else {
            // 每次增长大约1.25倍
            for 0 < newcap && newcap < cap {
                newcap += newcap / 4
            }
            if newcap <= 0 {
                newcap = cap
            }
        }
    }
    // ...
    return slice{p, old.len, newcap}
}	
```

该规则的问题：

1. 不够平滑。容量小于 1024 就 2 倍扩容，否则降到 1.25 倍扩容。
2. 容量增长不单调。比如容量 1000 的 slice 扩容后大小比 1100 的 slice 扩容后的大小大很多，因为一个是翻倍扩容，一个是 1.25 倍扩容。



## 新规则

可以查看这个 commit：https://github.com/golang/go/commit/2dda92ff6f9f07eeb110ecbf0fc2d7a0ddd27f9d#diff-fb7a84f457b2d4466b5d32940ffc6095d12b649f88bd3e5e762a87631e7a917aR2496

具体就是双倍容量扩容的最大阈值**从1024降为了256**，只要超过了256，就开始进行缓慢的增长。其次是增长比例的调整，之前超过了阈值之后，基本为恒定的1.25 倍增长，而现在超过了阈值之后，增长比例是会动态调整的:

```
初始长度         增长比例
256             2.0
512             1.63
1024            1.44
2048            1.35
4096            1.30
```

可以看到，**随着切片容量的变大，增长比例逐渐向着1.25进行靠拢**。

新代码如下：

```go
func growslice(oldPtr unsafe.Pointer, newLen, oldCap, num int, et *_type) slice {
  // ...
  newcap := oldCap
	doublecap := newcap + newcap
	if newLen > doublecap {
		newcap = newLen
	} else {
		const threshold = 256				// 阈值改为 256
		if oldCap < threshold {
			newcap = doublecap
		} else {
			for 0 < newcap && newcap < newLen {
				// Transition from growing 2x for small slices
				// to growing 1.25x for large slices. This formula
				// gives a smooth-ish transition between the two.
				newcap += (newcap + 3*threshold) / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = newLen
			}
		}
	}
  // ...
  return slice{p, newLen, newcap}
}
```

在容量 256 以后，预留的 buffer 容量为 1/4 的容量 + 3/4 * 256 （如果 oldCap 刚好是 256，则更好也是 2 倍扩容） 。这样当 old.cap < 256 时还是 2 倍扩容，随着容量增大，3/4 * 256 占原容量的比例**逐渐降低，最终收敛于 0**，达到最终也是 1.25 倍且平滑扩容的效果。这种改变使得容量增长更均匀，减少了不必要的数据拷贝。



## 总结

- 1.18以前：
  - 容量小于1024时为2倍扩容，大于等于1024时为1.25倍扩容
- 1.18以后：
  - 小于256时为2倍扩容，大于等于256时的扩容因子逐渐从2减低为1.25
- 不管1.18前后，最终需要按照内存分配块向上修正