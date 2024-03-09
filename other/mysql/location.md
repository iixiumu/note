show variables like "%time_zone%";

set global time_zone = '+8:00';  ##修改mysql全局时区为北京时间，即我们所在的东8区
set time_zone = '+8:00';  ##修改当前会话时区
flush privileges;  #立即生效

vim /etc/my.cnf  ##在[mysqld]区域中加上
default-time_zone = '+8:00'
