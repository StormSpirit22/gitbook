# 布隆过滤器

布隆过滤器的原理可参考: [详解布隆过滤器的原理，使用场景和注意事项](https://zhuanlan.zhihu.com/p/43263751)



## 布隆过滤器bloom

布隆过滤器，可以判断某元素在不在集合里面,因为存在一定的误判和删除复杂问题,一般的使用场景是:防止缓存击穿(防止恶意攻击)、 垃圾邮箱过滤、cache digests 、模型检测器等、判断是否存在某行数据,用以减少对磁盘访问，提高服务的访问性能。 go-zero 提供的简单的缓存封装 bloom.bloom，就是借助 redis bitmap (其实就是通过 redis string 来实现) 的 [setbit](https://redis.io/commands/setbit) 方法来实现 bloom。bloom filter 的数据结构如下：

```go
// A Filter is a bloom filter.
Filter struct {
  bits   uint
  bitSet bitSetProvider
}

bitSetProvider interface {
  check([]uint) (bool, error)
  set([]uint) error
}

// redisBitSet 实现了 bitSetProvider 接口
type redisBitSet struct {
	store *redis.Redis			// redis 实例
	key   string						// 布隆过滤器的 key，也就是在 redis 中通过这个 key 来获取到布隆过滤器
	bits  uint							// 一共可以用多少 bits 
}

```

简单使用方式如下：

```go
// 初始化 redisBitSet
store := redis.NewRedis("redis 地址", redis.NodeType)
// 声明一个bitSet, key="test_key"名且bits是1024位
bitSet := newRedisBitSet(store, "test_key", 1024)
// 判断第0位bit存不存在
isSetBefore, err := bitSet.check([]uint{0})

// 对第512位设置为1
err = bitSet.set([]uint{512})
// 3600秒后过期 
err = bitSet.expire(3600)

// 删除该bitSet
err = bitSet.del()
```

其中真正的bloom实现， 对元素hash 定位核心代码：

```go
// 对元素进行hash 14次(const maps=14),每次都在元素后追加byte(0-13),然后进行hash.
// 将locations[0-13] 进行取模,最终返回locations.
// maps 是表示需要多少种 hash 算法（go-zero 里设置了 14），bits is how many bits will be used
// best practices:
// elements - means how many actual elements
// when maps = 14, formula: 0.7*(bits/maps), bits = 20*elements, the error rate is 0.000067 < 1e-4
// for detailed error rate table, see http://pages.cs.wisc.edu/~cao/papers/summary-cache/node8.html
func (f *BloomFilter) getLocations(data []byte) []uint {
    locations := make([]uint, maps)
    for i := uint(0); i < maps; i++ {
        hashValue := hash.Hash(append(data, byte(i)))
        locations[i] = uint(hashValue % uint64(f.bits))
    }

    return locations
}
```

向bloom里面add 元素

```go
// 我们可以发现 add方法使用了getLocations和bitSet的set方法。
// 我们将元素进行hash成长度14的uint切片,然后进行set操作存到redis的bitSet里面。
func (f *BloomFilter) Add(data []byte) error {
    locations := f.getLocations(data)
    err := f.bitSet.set(locations)
    if err != nil {
        return err
    }
    return nil
}
```

检查bloom里面是否有某元素

```go
// 我们可以发现 Exists方法使用了getLocations和bitSet的check方法
// 我们将元素进行hash成长度14的uint切片,然后进行bitSet的check验证,存在返回true,不存在或者check失败返回false
func (f *BloomFilter) Exists(data []byte) (bool, error) {
    locations := f.getLocations(data)
    isSet, err := f.bitSet.check(locations)
    if err != nil {
        return false, err
    }
    if !isSet {
        return false, nil
    }

    return true, nil
}
```

其中 set 和 check 方法：

```go
const (
	// for detailed error rate table, see http://pages.cs.wisc.edu/~cao/papers/summary-cache/node8.html
	// maps as k in the error rate table
	maps      = 14
  
  // 这里 setbit 命令就是上面说的 redis 命令
	setScript = `
for _, offset in ipairs(ARGV) do
	redis.call("setbit", KEYS[1], offset, 1)
end
`
	testScript = `
for _, offset in ipairs(ARGV) do
	if tonumber(redis.call("getbit", KEYS[1], offset)) == 0 then
		return false
	end
end
return true
`
)

func (r *redisBitSet) set(offsets []uint) error {
	args, err := r.buildOffsetArgs(offsets)	 // 主要是检测偏移量数组的值是否超过 bits，然后转化成 string 数组
	if err != nil {
		return err
	}

	_, err = r.store.Eval(setScript, []string{r.key}, args)
	if err == redis.Nil {
		return nil
	}

	return err
}

func (r *redisBitSet) check(offsets []uint) (bool, error) {
	args, err := r.buildOffsetArgs(offsets)
	if err != nil {
		return false, err
	}

	resp, err := r.store.Eval(testScript, []string{r.key}, args)
	if err == redis.Nil {
		return false, nil
	} else if err != nil {
		return false, err
	}

	exists, ok := resp.(int64)
	if !ok {
		return false, nil
	}

	return exists == 1, nil
}
```



## 总结

go-zero 的布隆过滤器就是通过 redis 的 bitmap 来实现的，默认使用 14 种 hash 算法，可以自定义一共使用多少位来存储。

向 bloom 添加元素时需要使用 14 种 hash 算法将数据（[]byte 类型）hash 得出 14 个偏移量，再通过 redis  setbit 方法将 bitmap 对应偏移量位置的值设置为 1。检查 bloom 是否有某元素时也是一样，得出 14 个偏移量，再通过 redis getbit 方法来检查这 14 个位置是否都是 1，有一个位置不是 1 就表示不存在，都是 1 就表示存在。

