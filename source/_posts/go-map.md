---
title: go-map
date: 2024-09-07 18:32:22
categories:
- golang
tags:
---

# golang
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

