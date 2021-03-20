# map

map是引用类型，可以使用如下声明：

```go
var map1 map[keytype]valuetype
var map1 map[string]int
```

（`[keytype]` 和 `valuetype` 之间允许有空格，但是 gofmt 移除了空格）

map在声明的时候不需要直到map的长度，map是可以动态增长的

未初始化的map值是nil

key 可以是任意可以用 == 或者 != 操作符比较的类型，比如 string、int、float。所以数组、切片和结构体不能作为 key，但是指针和接口类型可以。

如果要用结构体作为 key 可以提供 `Key()` 和 `Hash()` 方法，这样可以通过结构体的域计算出唯一的数字或者字符串的 key。

value 可以是任意类型的；通过使用空接口类型（详见第 11.9 节），我们可以存储任意值，但是使用这种类型作为值时需要先做一次类型断言（详见第 11.3 节）。

```go
package main
import "fmt"

func main() {
    var mapLit map[string]int
    //var mapCreated map[string]float32
    var mapAssigned map[string]int

    mapLit = map[string]int{"one": 1, "two": 2}
    mapCreated := make(map[string]float32)
    mapAssigned = mapLit

    mapCreated["key1"] = 4.5
    mapCreated["key2"] = 3.14159
    mapAssigned["two"] = 3

    fmt.Printf("Map literal at \"one\" is: %d\n", mapLit["one"])
    fmt.Printf("Map created at \"key2\" is: %f\n", mapCreated["key2"])
    fmt.Printf("Map assigned at \"two\" is: %d\n", mapLit["two"])
    fmt.Printf("Map literal at \"ten\" is: %d\n", mapLit["ten"])
}

// 输出结果
Map literal at "one" is: 1
Map created at "key2" is: 3.14159
Map assigned at "two" is: 3
Mpa literal at "ten" is: 0
```

上面这段代码让我们看到了map的几种初始化方式

- map 可以用 `{key1: val1, key2: val2}` 的描述方法来初始化，就像数组和结构体一样。

- map 是 **引用类型** 的： 内存用 make 方法来分配。

  map 的初始化：`var map1[keytype]valuetype = make(map[keytype]valuetype)`。

  或者简写为：`map1 := make(map[keytype]valuetype)`。**不要使用 new，永远用 make 来构造 map**

map中的value可以是任意类型的，如下面这段代码：

```go
package main
import "fmt"

func main() {
    mf := map[int]func() int{
        1: func() int { return 10 },
        2: func() int { return 20 },
        5: func() int { return 50 },
    }
    fmt.Println(mf)
}
```

## map容量

和数组不同，map 可以根据新增的 key-value 对动态的伸缩，因此它不存在固定长度或者最大限制。

**但是你也可以选择标明 map 的初始容量 `capacity`**

## 测试键值对是否存在及删除元素

我们可以使用`val = map[key]`的方式来获取key对应的value，但是当map中不存在key的时候，也会返回0值，因此我们需要额外的结果来判断是否map中是否有key值`val, isPresent = map[key]`，isPresent 返回一个 bool 值

```go
_, ok := map1[key1] // 如果key1存在则ok == true，否在ok为false
```

或者和 if 混合使用：

```go
if _, ok := map1[key1]; ok {
    // ...
}
```

**从map中删除key， 直接`delete(map, key)`**就可以，如果key不存在，该操作不会产生错误

```go
package main
import "fmt"

func main() {
    var value int
    var isPresent bool

    map1 := make(map[string]int)
    map1["New Delhi"] = 55
    map1["Beijing"] = 20
    map1["Washington"] = 25
    value, isPresent = map1["Beijing"]
    if isPresent {
        fmt.Printf("The value of \"Beijing\" in map1 is: %d\n", value)
    } else {
        fmt.Printf("map1 does not contain Beijing")
    }

    value, isPresent = map1["Paris"]
    fmt.Printf("Is \"Paris\" in map1 ?: %t\n", isPresent)
    fmt.Printf("Value is: %d\n", value)

    // delete an item:
    delete(map1, "Washington")
    value, isPresent = map1["Washington"]
    if isPresent {
        fmt.Printf("The value of \"Washington\" in map1 is: %d\n", value)
    } else {
        fmt.Println("map1 does not contain Washington")
    }
}

// 输出结果
The value of "Beijing" in map1 is: 20
Is "Paris" in map1 ?: false
Value is: 0
map1 does not contain Washington
```

## map中的for-range

可以使用for循环来遍历map

```go
for key, value := range map{
    ...
}
```

如果只想获取value，可以：

```go
for _, value := range map{
    fmt.Println(value)
}
```

如果只想获取key，可以：

```go
for key, _ := range map{
    fmt.Println(key)
}
```

注意 map 不是按照 key 的顺序排列的，也不是按照 value 的序排列的。

## map类型的切片

假设我们想获取一个map类型的切片，我们必须使用两次`make()`函数，第一次分配切片，第二次分配切片中每个map元素

```go
package main
import "fmt"

func main() {
    // Version A:
    items := make([]map[int]int, 5)
    for i:= range items {
        items[i] = make(map[int]int, 1)
        items[i][1] = 2
    }
    fmt.Printf("Version A: Value of items: %v\n", items)

    // Version B: NOT GOOD!
    items2 := make([]map[int]int, 5)
    for _, item := range items2 {
        item = make(map[int]int, 1) // item is only a copy of the slice element.
        item[1] = 2 // This 'item' will be lost on the next iteration.
    }
    fmt.Printf("Version B: Value of items: %v\n", items2)
}
```

输出结果：

```
Version A: Value of items: [map[1:2] map[1:2] map[1:2] map[1:2] map[1:2]]
Version B: Value of items: [map[] map[] map[] map[] map[]]
```

需要注意的是，应当像 A 版本那样通过索引使用切片的 map 元素。在 B 版本中获得的项只是 map 值的一个拷贝而已，所以真正的 map 元素没有得到初始化。

## map的排序

map 默认是无序的，不管是按照 key 还是按照 value 默认都不排序

如果你想为 map 排序，需要将 key（或者 value）拷贝到一个切片，再对切片排序

```go
package main
import (
    "fmt"
    "sort"
)

var (
    barVal = map[string]int{"alpha": 34, "bravo": 56, "charlie": 23,
                            "delta": 87, "echo": 56, "foxtrot": 12,
                            "golf": 34, "hotel": 16, "indio": 87,
                            "juliet": 65, "kili": 43, "lima": 98}
)

func main() {
    fmt.Println("unsorted:")
    for k, v := range barVal {
        fmt.Printf("Key: %v, Value: %v / ", k, v)
    }
    keys := make([]string, len(barVal))
    i := 0
    for k, _ := range barVal {
        keys[i] = k
    i++
    }
    sort.Strings(keys)
    fmt.Println()
    fmt.Println("sorted:")
    for _, k := range keys {
        fmt.Printf("Key: %v, Value: %v / ", k, barVal[k])
    }
}
```

输出结果：

```go
unsorted:
Key: bravo, Value: 56 / Key: echo, Value: 56 / Key: indio, Value: 87 / Key: juliet, Value: 65 / Key: alpha, Value: 34 / Key: charlie, Value: 23 / Key: delta, Value: 87 / Key: foxtrot, Value: 12 / Key: golf, Value: 34 / Key: hotel, Value: 16 / Key: kili, Value: 43 / Key: lima, Value: 98 /
sorted:
Key: alpha, Value: 34 / Key: bravo, Value: 56 / Key: charlie, Value: 23 / Key: delta, Value: 87 / Key: echo, Value: 56 / Key: foxtrot, Value: 12 / Key: golf, Value: 34 / Key: hotel, Value: 16 / Key: indio, Value: 87 / Key: juliet, Value: 65 / Key: kili, Value: 43 / Key: lima, Value: 98 / [fangjun@st01-dstream-0001.st01.baidu.com go]$ sz -be sort_map.go
```

## 将map的键值对调

这里倒置是指调换 key 和 value。如果 map 的值类型可以作为 key 且所有的 value 是唯一的，那么通过下面的方法可以简单的做到键值对调。