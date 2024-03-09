make:   slice, map, channel等内置数据结构
new:    获取指向某个类型的指针
``` go
slice := make([]int, 0, 100)
hash := make(map[int]bool, 10)
ch := make(chan int, 5)
```
1.  slice 是一个包含 data、cap 和 len 的结构体；
2.  hash 是一个指向 hmap 结构体的指针；
3.  ch 是一个指向 hchan 结构体的指针；
