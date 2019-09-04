# Channel

记录go里面channel的用法

## chan的读取操作

channel写完后，必须关闭通道。否则在使用range对其进行遍历时发生死锁错误。
```fatal error: all goroutines are asleep - deadlock!```

* 读取有缓冲的channel</br>
读取关闭后的有缓存channel，将缓存数据读取完后，再读取返回值为0和false。

* 读取无缓冲的channel</br>
读取关闭后的无缓冲channel，直接返回0和false。

```go
done := make(chan int)
go func() {
    done <- 1
}()
close(done)
for i := 1; i <= 3; i++ {
    t, ok := <-done
    fmt.Println(i, ":", t, ok)
}

// output:
// 1 : 0 false
// 2 : 0 false
// 3 : 0 false
// panic: send on closed channel
```

## channel的关闭时间

channel的close操作应该紧跟于写完最后一个数据之后

```go
nums := 10
toProcess := make(chan int, nums)
for i := 0; i < nums; i++ {
    toProcess <- i
}
close(toProcess)
```
