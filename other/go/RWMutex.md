``` go
type RWMutex struct {
	w           Mutex  // held if there are pending writers
	writerSem   uint32 // semaphore for writers to wait for completing readers
	readerSem   uint32 // semaphore for readers to wait for completing writers
	readerCount int32  // number of pending readers
	readerWait  int32  // number of departing readers
}
```
RLock
    rw.readerCount + 1
    如果小于0，表明有writer执行或者正在等待
    等待信号量，runtime_SemacquireMutex rw.readerSem

RUnlock
    rw.readerCount - 1
    如果小于0，表明有writer正在等待
    rw.readerWait - 1
    如果等于0，表明writer等待的reader全部结束
    唤醒writer，runtime_Semrelease rw.writerSem

Lock
    获取锁
    rw.readerCount - rwmutexMaxReaders
    如果之前的rw.readerCount不等于0，而且
    rw.readerWait + 之前的rw.readerCount之后不等于0
    表明有reader，需要等待，