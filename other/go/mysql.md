https://github.com/go-sql-driver/mysql#timetime-support
*	loc
Type:           string
Valid Values:   <escaped name>
Default:        UTC
Sets the location for time.Time values (when using parseTime=true). "Local" sets the system's location. See time.LoadLocation for details.

Note that this sets the location for time.Time values but does not change MySQL's time_zone setting. For that see the time_zone system variable, which can also be set as a DSN parameter.

Please keep in mind, that param values must be url.QueryEscape'ed. Alternatively you can manually replace the / with %2F. For example US/Pacific would be loc=US%2FPacific.

*	parseTime
Type:           bool
Valid Values:   true, false
Default:        false
parseTime=true changes the output type of DATE and DATETIME values to time.Time instead of []byte / string The date or datetime like 0000-00-00 00:00:00 is converted into zero value of time.Time.

https://github.com/go-sql-driver/mysql#unicode-support
*   charset
Type:           string
Valid Values:   <name>
Default:        none
Sets the charset used for client-server interaction ("SET NAMES <value>"). If multiple charsets are set (separated by a comma), the following charset is used if setting the charset failes. This enables for example support for utf8mb4 (introduced in MySQL 5.5.3) with fallback to utf8 for older servers (charset=utf8mb4,utf8).

Usage of the charset parameter is discouraged because it issues additional queries to the server. Unless you need the fallback behavior, please use collation instead.

*   collation
Type:           string
Valid Values:   <name>
Default:        utf8mb4_general_ci
Sets the collation used for client-server interaction on connection. In contrast to charset, collation does not issue additional queries. If the specified collation is unavailable on the target server, the connection will fail.

A list of valid charsets for a server is retrievable with SHOW COLLATION.

The default collation (utf8mb4_general_ci) is supported from MySQL 5.5. You should use an older collation (e.g. utf8_general_ci) for older MySQL.

Collations for charset "ucs2", "utf16", "utf16le", and "utf32" can not be used (ref).
