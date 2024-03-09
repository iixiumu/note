## EXPLAIN Extra

<https://dev.mysql.com/doc/refman/5.7/en/explain-output.html>

<https://dev.mysql.com/doc/refman/8.0/en/explain-output.html>

* Using where (JSON property: attached_condition)

A WHERE clause is used to restrict which rows to match against the next table or send to the client. Unless you specifically intend to fetch or examine all rows from the table, you may have something wrong in your query if the Extra value is not Using where and the table join type is ALL or index.

Using where has no direct counterpart in JSON-formatted output; the attached_condition property contains any WHERE condition used.

* Using index (JSON property: using_index)

The column information is retrieved from the table using only information in the index tree without having to do an additional seek to read the actual row. This strategy can be used when the query uses only columns that are part of a single index.

For InnoDB tables that have a user-defined clustered index, that index can be used even when Using index is absent from the Extra column. This is the case if type is index and key is PRIMARY.

* Using index condition (JSON property: using_index_condition)

Tables are read by accessing index tuples and testing them first to determine whether to read full table rows. In this way, index information is used to defer (“push down”) reading full table rows unless it is necessary. See Section 8.2.1.5, “Index Condition Pushdown Optimization”.

* Using sort_union(...), Using union(...), Using intersect(...) (JSON property: message)

These indicate the particular algorithm showing how index scans are merged for the index_merge join type. See Section 8.2.1.3, “Index Merge Optimization”.

* Using filesort (JSON property: using_filesort)

MySQL must do an extra pass to find out how to retrieve the rows in sorted order. The sort is done by going through all rows according to the join type and storing the sort key and pointer to the row for all rows that match the WHERE clause. The keys then are sorted and the rows are retrieved in sorted order. See Section 8.2.1.14, “ORDER BY Optimization”.

