*   AUTO_INCREMENT不生效
需要这样写ID uint64 `gorm:"type:bigint(20) unsigned auto_increment;not null;primary_key"`

when you set sql type, you need to include all types.
For this example, it's primary key, so you need to include AUTO_INCREMENT, so the type should looks like
sql:"type:int unsigned NOT NULL AUTO_INCREMENT PRIMARY KEY"

https://github.com/jinzhu/gorm/issues/2190
https://github.com/jinzhu/gorm/issues/201

