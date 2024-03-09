# MySQL分页

通往地狱的路上，挤满了原本想去天堂的人。

入职两个多月了，收获3次downtime，挺好玩的。发现问题，解决问题，挺好。如果每天都做一样的事情，那么过一天和过一生又有什么区别呢，做不一样的事情，多一些经历和体验。

感觉好多资料还是看英文版的效果好一些，，好多东西翻译过来之后丢失信息，而且有时候是非常重要的信息，有时候很久没有搞明白的问题，找到英文版，瞬间就搞明白了。比如MySQL，官方文档就相当不错了，<https://dev.mysql.com/doc/relnotes/mysql/5.7/en/>

MySQL分页分页问题，以前没有多想，工作上碰到了，看看吧。

## Sakila Sample Database

<https://dev.mysql.com/doc/sakila/en/>

<https://downloads.mysql.com/docs/sakila-db.zip>

``` sql
CREATE TABLE rental (
  id INT NOT NULL AUTO_INCREMENT,
  rental_date DATETIME NOT NULL,
  inventory_id MEDIUMINT UNSIGNED NOT NULL,
  customer_id SMALLINT UNSIGNED NOT NULL,
  return_date DATETIME DEFAULT NULL,
  staff_id TINYINT UNSIGNED NOT NULL,
  last_update TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  UNIQUE KEY  (rental_date,inventory_id,customer_id),
  KEY idx_fk_inventory_id (inventory_id),
  KEY idx_fk_customer_id (customer_id),
  KEY idx_fk_staff_id (staff_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## ORDER BY Optimization

<https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html>

In some cases, MySQL may use an index to satisfy an ORDER BY clause and avoid the extra sorting involved in performing a filesort operation.

The index may also be used even if the ORDER BY does not match the index exactly, as long as all unused portions of the index and all extra ORDER BY columns are constants in the WHERE clause. If the index does not contain all columns accessed by the query, the index is used only if index access is cheaper than other access methods.

In this query, the index on (key_part1, key_part2) enables the optimizer to avoid sorting:

``` sql
SELECT * FROM t1
  ORDER BY key_part1, key_part2;
```

However, the query uses SELECT \*, which may select more columns than key_part1 and key_part2. In that case, scanning an entire index and looking up table rows to find columns not in the index may be more expensive than scanning the table and sorting the results. If so, the optimizer probably will not use the index. If SELECT * selects only the index columns, the index will be used and sorting avoided.

If t1 is an InnoDB table, the table primary key is implicitly part of the index, and the index can be used to resolve the ORDER BY for this query:

``` sql
SELECT pk, key_part1, key_part2 FROM t1
  ORDER BY key_part1, key_part2;
```

## LIMIT Query Optimization

<https://dev.mysql.com/doc/refman/5.7/en/limit-optimization.html>

If multiple rows have identical values in the ORDER BY columns, the server is free to return those rows in any order, and may do so differently depending on the overall execution plan. In other words, the sort order of those rows is nondeterministic with respect to the nonordered columns.

One factor that affects the execution plan is LIMIT, so an ORDER BY query with and without LIMIT may return rows in different orders.

If it is important to ensure the same row order with and without LIMIT, include additional columns in the ORDER BY clause to make the order deterministic.

## default sort order

[Re: What is The Default Sort Order of SELECTS with no ORDER BY Clause?](https://forums.mysql.com/read.php?21,239471,239688)

* Do not depend on order when ORDER BY is missing.
* Always specify ORDER BY if you want a particular order -- in some situations the engine can eliminate the ORDER BY because of how it does some other step.
* GROUP BY forces ORDER BY. (This is a violation of the standard. It can be avoided by using ORDER BY NULL.)

SELECT * FROM tbl -- this will do a "table scan". If the table has never had any DELETEs/REPLACEs/UPDATEs, the records will happen to be in the insertion order, hence what you observed.

If you had done the same statement with an InnoDB table, they would have been delivered in PRIMARY KEY order, not INSERT order. Again, this is an artifact of the underlying implementation, not something to depend on.

## 实验

附录代码测试结果：

    4.13 6.04 10.17
    5.14 5.08 10.23
    0.37 0.88 1.25
    0.41 1.02 1.44

数据集比较小，所以limit offset, rows到后面下降不是很严重，线上数据量比较大的时候，性能下降特别明显，不过差距已经比较大了。两个结论：基于书签记录的分页比limit offset, rows性能好很多（理论上来说不存在到后面性能下降的问题），select id比select \*效果好很多，这个很好理解了，select *多了一次回表。

其实还应该分析explain的，略复杂，先不看了。

## 实践

* 不指定ORDER BY时候顺序是不确定的，如果有需要，指定ORDER BY
* 基于书签记录做分页（limit offset, rows在数据集变化的情况下结果要么重叠要么缺失）
* 如果需要select *，可以做一个自身内连接，延迟关联，防止性能下降
* 如果需要select一个非主键的东西，一定要放入联合索引

## 附录

``` go
package main

import (
	"fmt"
	"time"

	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
)

type Rental struct {
	ID          int       `gorm:"Column:id" json:"id"`
	RentalDate  time.Time `gorm:"Column:rental_date" json:"rental_date"`
	InventoryID int       `gorm:"Column:inventory_id" json:"inventory_id"`
	CustomerID  int       `gorm:"Column:customer_id" json:"customer_id"`
	ReturnDate  time.Time `gorm:"Column:return_date" json:"return_date"`
	StaffID     int       `gorm:"Column:staff_id" json:"staff_id"`
	LastUpdate  time.Time `gorm:"Column:last_update" json:"last_update"`
}

func (Rental) TableName() string {
	return "rental"
}

var myDB *gorm.DB

func init() {
	var err error
	myDB, err = gorm.Open("mysql", "root:@tcp(127.0.0.1:3306)/test?&charset=utf8mb4&collation=utf8mb4_general_ci&parseTime=true&loc=Asia%2FShanghai")
	if err != nil {
		panic("mysql connect error: " + err.Error())
	}
	myDB.DB().Ping()
	myDB.DB().SetMaxIdleConns(5)
	myDB.DB().SetMaxOpenConns(10)
	myDB.LogMode(false)
}

func panicError(err error) {
	if err != nil {
		panic(err)
	}
}

type IDCount struct {
	ID    int
	Count int
}

func count(key string) []IDCount {
	var idCounts []IDCount
	sql := fmt.Sprintf("select %s as id, count(*) as count from rental group by %s", key, key)
	err := myDB.Raw(sql).Scan(&idCounts).Error
	panicError(err)
	return idCounts
}

const Limit = 10

type Runner interface {
	run(selected string, key string, id, count int)
}

type Raw struct{}

func (Raw) run(selected string, key string, id, count int) {
	for start := 0; start < count; start += Limit {
		sql := fmt.Sprintf("select %s from rental where %s = %d limit %d, %d", selected, key, id, start, Limit)
		rentals := make([]Rental, 0, Limit)
		err := myDB.Raw(sql).Scan(&rentals).Error
		panicError(err)
	}
}

type RawSort struct{}

func (RawSort) run(selected string, key string, id, count int) {
	for start := 0; start < count; start += Limit {
		sql := fmt.Sprintf("select %s from rental where %s = %d order by id limit %d, %d", selected, key, id, start, Limit)
		rentals := make([]Rental, 0, Limit)
		err := myDB.Raw(sql).Scan(&rentals).Error
		panicError(err)
	}
}

type NoSort struct{}

func (NoSort) run(selected string, key string, id, count int) {
	start := 0
	for {
		sql := fmt.Sprintf("select %s from rental where %s = %d and id > %d limit %d", selected, key, id, start, Limit)
		rentals := make([]Rental, 0, Limit)
		err := myDB.Raw(sql).Scan(&rentals).Error
		panicError(err)
		if len(rentals) == 0 {
			break
		}
		start = rentals[len(rentals)-1].ID
	}
}

type Sorted struct{}

func (Sorted) run(selected string, key string, id, count int) {
	start := 0
	for {
		sql := fmt.Sprintf("select %s from rental where %s = %d and id > %d order by id limit %d", selected, key, id, start, Limit)
		rentals := make([]Rental, 0, Limit)
		err := myDB.Raw(sql).Scan(&rentals).Error
		panicError(err)
		if len(rentals) == 0 {
			break
		}
		start = rentals[len(rentals)-1].ID
	}
}

func main() {
	key := "staff_id"
	runners := []Runner{&Raw{}, &RawSort{}, &NoSort{}, &Sorted{}}
	idCounts := count(key)
	for _, runner := range runners {
		t1 := time.Now().UnixNano()
		for _, idCount := range idCounts {
			runner.run("id", key, idCount.ID, idCount.Count)
		}
		t2 := time.Now().UnixNano()
		for _, idCount := range idCounts {
			runner.run("*", key, idCount.ID, idCount.Count)
		}
		t3 := time.Now().UnixNano()
		fmt.Printf("%.2f %.2f %.2f\n", float64(t2-t1)/1e9, float64(t3-t2)/1e9, float64(t3-t1)/1e9)
	}
}
```
