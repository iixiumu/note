#   Uses go test
GoConvey supports Go's native testing package. Neither the web UI nor the DSL are required; you can use either one independently.
#   Terminal or browser
Since GoConvey integrates with go test, you can keep running tests in the terminal or use the auto-updating web UI for test results.
#   Tell your program's story
Though writing tests with Convey() and So() is optional, it's more expressive than t.Errorf().
#   Get started
$ go get github.com/smartystreets/goconvey
$ $GOPATH/bin/goconvey
Then open your browser to localhost:8080. Tests will be run from the working directory on down.

#   Skip
针对想忽略但又不想删掉或注释掉某些断言操作，GoConvey提供了Convey/So的Skip方法：

SkipConvey函数表明相应的闭包函数将不被执行
SkipSo函数表明相应的断言将不被执行

# 小节
import goconvey包时，前面加点号"."，以减少冗余的代码
测试函数的名字必须以Test开头，而且参数类型必须为*testing.T
每个测试用例必须使用Convey函数包裹起来，推荐使用Convey语句的嵌套，即一个函数有一个测试函数，测试函数中嵌套两级Convey语句，第一级Convey语句对应测试函数，第二级Convey语句对应测试用例
Convey语句的第三个参数习惯以闭包的形式实现，在闭包中通过So语句完成断言
使用GoConvey框架的 Web 界面特性，作为命令行的补充
在适当的场景下使用SkipConvey函数或SkipSo函数
当测试中有需要时，可以定制断言函数
