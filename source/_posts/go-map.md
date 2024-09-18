---
title: go-map
date: 2024-09-07 18:32:22
categories:
- golang
tags:
---

# map
## 底层结构
底层hash表，map实际就是一个指针，指向hmap结构体

```golang
type hmap struct {
  count     int              // 存储的键值对数目
  flags     uint8            // 状态标志（是否处于正在写入的状态等）
  B         uint8            // 桶的数目 2^B
  noverflow uint16           // 使用的溢出桶的数量
  hash0     uint32           // 生成hash的随机数种子

  buckets    unsafe.Pointer  // bucket数组指针，数组的大小为2^B（桶）
  oldbuckets unsafe.Pointer  // 扩容阶段用于记录旧桶用到的那些溢出桶的地址
  nevacuate  uintptr         // 记录渐进式扩容阶段下一个要迁移的旧桶编号
  extra *mapextra            // 指向mapextra结构体里边记录的都是溢出桶相关的信息
}
// buckets则是指向hash表节点bmap即bucket的指针，go中一个桶最多装8个key
type bmap struct {
    tophash [bucketCnt]uint8        
    // len为8的数组，用来快速定位key是否在这个bmap中
    // 一个桶最多8个槽位，如果key所在的tophash值在tophash中，则代表该key在这个桶中。hash 值低8位用来定位 bucket，高8位定位 tophash。
}
```

当编译过后，runtime.bmap会扩展成以下结构
```golang
type bmap struct{
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr        // 内存对齐使用，可能不需要
    overflow uintptr        // 当bucket 的8个key 存满了之后
    // overflow 指向下一个溢出桶 bmap，
    // overflow是uintptr而不是*bmap类型，保证bmap完全不含指针，是为了减少gc，溢出桶存储到extra字段中
}
```

## 扩容
采用渐进式扩容，原有的key并不会一次性搬迁完毕，每次最多只会搬迁2个bucket，只有在插入或修改，删除key的时候，都会尝试进行搬迁buckets的工作。

- 翻倍扩容
当负载因子达到6.5的时候进行翻倍扩容。
迁移过程中使用与运算发hash&(m-1)，把旧桶迁移到新桶上，用旧桶的hash值跟扩容后的桶的个数m-1的值相与。
- 等量扩容
没有超过负载因子，使用溢出桶过多，触发等量扩容。通常这种情况发生在很多键值对被删除的情况，造成overflow的bucket数量增加，但负载因子不高。当重新迁移后，会重新排列，使得排列更紧凑。

## map的顺序打印
乱序的原因：
- 不是从固定的0号bucket开始的，是随机的
- 扩容会发生key的搬迁
因为golang中map访问的时候每次是乱序的，怎么设计一个有序的。注意map不是线程安全的，sync.map是线程安全的。

```golang
// 设计一个map顺序打印
// 使用切片（slice）保存键的顺序：为了保持插入键的顺序，我们可以使用一个切片保存 map 中的键。在写入 map 时，将键按照插入顺序放入该切片中，这样在读取时就可以按照插入顺序读取。
//
// 使用 sync.RWMutex 实现并发安全：Go 的 map 本身不是线程安全的，若在多个 goroutine 中读写，需要加锁。我们可以使用 sync.RWMutex 来实现读写锁，保证顺序读写时的并发安全。
type OrderedMap struct {
	data  map[string]interface{}
	keys  []string
	mutex sync.RWMutex
}

// NewOrderedMap 创建一个有序的 map
func NewOrderedMap() *OrderedMap {
	return &OrderedMap{
		data: make(map[string]interface{}),
		keys: make([]string, 0),
	}
}

// Set 插入键值对，保持顺序
func (om *OrderedMap) Set(key string, value interface{}) {
	om.mutex.Lock()
	defer om.mutex.Unlock()

	// 检查是否是新键
	if _, exists := om.data[key]; !exists {
		om.keys = append(om.keys, key)
	}
	om.data[key] = value
}

// Get 获取键的值
func (om *OrderedMap) Get(key string) (interface{}, bool) {
	om.mutex.RLock()
	defer om.mutex.RUnlock()

	value, exists := om.data[key]
	return value, exists
}

// Delete 删除键
func (om *OrderedMap) Delete(key string) {
	om.mutex.Lock()
	defer om.mutex.Unlock()

	// 删除数据
	if _, exists := om.data[key]; exists {
		delete(om.data, key)
		// 删除顺序中的键
		for i, k := range om.keys {
			if k == key {
				om.keys = append(om.keys[:i], om.keys[i+1:]...)
				break
			}
		}
	}
}

// Keys 返回所有键的顺序
func (om *OrderedMap) Keys() []string {
	om.mutex.RLock()
	defer om.mutex.RUnlock()

	// 返回键的副本，防止外部修改
	return append([]string{}, om.keys...)
}

// Iterate 顺序遍历所有键值对
func (om *OrderedMap) Iterate() {
	om.mutex.RLock()
	defer om.mutex.RUnlock()

	for _, key := range om.keys {
		fmt.Printf("Key: %s, Value: %v\n", key, om.data[key])
	}
}

func main() {
	// 创建有序 map
	orderedMap := NewOrderedMap()

	// 顺序写入
	orderedMap.Set("a", 1)
	orderedMap.Set("b", 2)
	orderedMap.Set("c", 3)

	// 顺序读取
	orderedMap.Iterate()

	// 获取单个值
	if val, ok := orderedMap.Get("b"); ok {
		fmt.Printf("Get b: %v\n", val)
	}

	// 删除一个键
	orderedMap.Delete("b")

	// 顺序读取
	orderedMap.Iterate()

	// 打印所有键
	fmt.Println("Keys:", orderedMap.Keys())
}

```

