想必做开发的，尤其是PHP或Java的，很多年前就都会注意到这个情况：时区中没有asia/beijing，只有asia/shanghai和asia/chongqing。以前看到不少这种抱怨的帖子，毕竟和心理预期不一样，还会导致程序出错或程序员浪费时间调试。大家会猜测这是不是老外故意和北京捣乱。我认为不是。

首先，我想确这一点，在JDK诞生之前，国际标准时区就没有Asia/Beijing，只有Asia/Shanghai或Asia/Chongqing？（One thing I want to be confirmed here is, before JDK's birth, the international standard TimeZone name for PRC is Asia/Shanghai or Asia/Irkutsk? not Asia/Beijing?）

其次，在1986年到1991年期间，中华人民共和国采用了夏时制。为了能够处理任何给定的时间格式，时区就需要知道是否历史上使用夏令时。daylight就是表明这一点的字段。因此，在JDK中使用daylight字段是不对的。

但根据他后面说的，我认为sun似乎有必要更改时区，毕竟sun认为时区的api应该支持同时代的时区。Asia/Shanghai和Asia/Chongqing代表中国，都是国民党时期的老黄历了：

sun的策略是只支持同时代（contemporary）的时区，而不是历史上（historical）的某一个时区。TimeZone和DateFormat code只应该对于当前的时区工作。最后，Fixed in JDK1.2 FCS-M, but the value is Asia/Shanghai, better one is Asia/Beijing.
