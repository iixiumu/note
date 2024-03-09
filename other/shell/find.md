find [-H] [-L] [-P] [-D debugopts] [-Olevel] [path...] [expression]

atime：访问时间（access time），指的是文件最后被读取的时间，可以使用touch命令更改为当前时间；
ctime：变更时间（change time），指的是文件本身最后被变更的时间，变更动作可以使chmod、chgrp、mv等等；
mtime：修改时间（modify time），指的是文件内容最后被修改的时间，修改动作可以使echo重定向、vi等等；
find . {-atime/-ctime/-mtime/-amin/-cmin/-mmin} [-/+]num

