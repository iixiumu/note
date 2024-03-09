mysql> CHECK  TABLE new12306;
+----------------+-------+----------+------------------------------------------------------+
| Table          | Op    | Msg_type | Msg_text                                             |
+----------------+-------+----------+------------------------------------------------------+
| xiumu.new12306 | check | warning  | Table is marked as crashed and last repair failed    |
| xiumu.new12306 | check | warning  | Size of indexfile is: 204308480      Should be: 4096 |
| xiumu.new12306 | check | error    | Record-count is not ok; is 7141154   Should be: 0    |
| xiumu.new12306 | check | warning  | Found 7141154 key parts. Should be: 0                |
| xiumu.new12306 | check | error    | Corrupt                                              |
+----------------+-------+----------+------------------------------------------------------+
5 rows in set (2.98 sec)

mysql> REPAIR TABLE new12306;
+----------------+--------+----------+------------------------------------------+
| Table          | Op     | Msg_type | Msg_text                                 |
+----------------+--------+----------+------------------------------------------+
| xiumu.new12306 | repair | warning  | Number of rows changed from 0 to 7141154 |
| xiumu.new12306 | repair | status   | OK                                       |
+----------------+--------+----------+------------------------------------------+
2 rows in set (4.78 sec)