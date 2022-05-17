```json
{
  "date": "2022.05.17 19:50",
  "tags": ["Golang","学习笔记"],
  "description":"我们在学习Go基础时，大部分人都会发现Map的遍历是随机顺序，然而很多人都知其所以然，不知其背后缘由。再深入细想，如果有遍历map必须有序的需求时，我们又该怎么实现呢？",
  "musicId":"1813512345"
}
```

![](./images/images.jpeg)

### 一、原生Map为什么无序？

了解 Map 底层实现原理的同学知道，遍历 Map 时，是按顺序遍历 bucket ，然后再按顺序遍历 bucket 中的 key （bucket 里面是一个链表）。然而，Map 在扩容时，key 会发生迁移，从旧 bucket 迁移到新 bucket，这种场景下，是做不到有序遍历的。

所以，Go 官方为了一开始就不让用户产生时而有序时而无序的烦恼，直接在 Map 遍历开始时，使用 fastrand 随机选择一个 bucket 作为起始桶。 

### 二、有序遍历的思路

1.思路一（实现快）：将 `Map` 中的 key 拿出来放入 `slice` 中做排序

```GO
func main() {
  m := make(map[string]string)
  m["name"] = "cogo"
  m["age"] = "25"
  m["gender"] = "male"
  
  keys := make([]string, 0)
  for k := range m {
    keys = append(keys, k)
  }
  sort.Strings(keys)
  for _, key := range keys {
    fmt.Println(m[key])
  }
}
```



2.思路二（一劳永逸）： 使用官方库中 `list`(链表) 封装一个结构体，实现一个有序的 `K-V` 存储结构，在里面维护一个 keys 的 list

追问：

为什么使用 list 而不是 slice 来保证有序性呢？ 因为 list 可以实现 insertBefore 、insertAfter 类似的方法，而 slice 实现相对复杂

参考现在一个比较流行的库注释讲解下：[github.com/elliotchanc…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Felliotchance%2Forderedmap)

```go
package orderedmap
import "container/list"

//......更多细节请看源码
//OrderedMap 核心数据结构
type OrderedMap struct {
   //存储 k-v,使用 *list.Element 当做 value 是利用 map O(1) 的性能找到 list 中的 element
   kv map[interface{}]*list.Element 
   //按顺序存储 k-v，保证插入、删除的时间复杂度O(1)
   ll *list.List
}
//......更多细节请看源码
```



### 总结

`Golang` 中 `Map` 之所以放弃有序遍历，也是出于性能和复杂度的考虑。这也是为什么上面的思路都或多或少地都比原生`Map`使用起来复杂了许多。有兴趣的同学还可以了解`PHP`之类的数组如何实现。