go test 竟然有缓存
ok      command-line-arguments  0.024s
ok      command-line-arguments  (cached)
如果代码没有发生变化，但是外部依赖（mysq，redis）发生变化，就坑了
go test -count=1 -v .
