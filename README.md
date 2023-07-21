
## Back log
**back_log**
The number of outstanding connection requests MySQL can have.  
This option is useful when the main MySQL thread gets many connection requests in a very short time. It then takes some time (although very little) for the main thread to check the connection and start a new thread. The back_log value indicates how many requests can be stacked during this short time before MySQL momentarily stops answering new requests. You need to increase this only if you expect a large number of connections in a short period of time.

## Redo Log
**redo_log** This configuration is already present in MySQL this can be reconfigured using the following:  
- In mysql, set innodb_fast_shutdown to 1  
- In mysql.conf file, add the following: (default is mentioned below)  

  		innodb_log_group_home_dir=./  
		innodb_log_file_size=50MB
		innodb_log_buffer_size=8MB
		innodb_log_files_in_group=2   
- Rule: Total log size (file_size * files_in_group) < innodb_buffer_pool_size  
  	
**Timeouts**  
Some of the important timeouts to be written in mysql.cnf are: (Not needed for us since we dont use transaction locking)  
- innodb_rollback_on_timeout=OFF (since there is no transaction lock taking place)  
- innodb_lock_wait_timeout=60   
- interactive_timeout=28800  
		time an application can stay idle (time between previous query) before connection is closed  
- wait_timeout=28800  
		time an node app can stay idle before connection is closed   
- net_read_timeout=30  
- net_write_timeout=60   
		Rule: net_write_timeout must be greater for max_allowed_packet  
  		
## lower_case_table_names
- table name casing is checked for Unix based systems and not for Windows based systems. for replication set the following  **lower_case_table_names=1**  
## Other Notes:		
- We can set up multiple instances (Parallel running)of MySQL server in linux server using mysqld_multi.  
- Learning about SQL mode options can be helpful for data validation
- Some of the variables for tuning takes up **memory per connection**. Normally MySQL would have 1000 active connections. They are
	- **join_buffer_size**
   	- **sort_buffer_size**
   	- **read_buffer_size**
   	- **read_rnd_buffer_size** (Regarding bringing out sorted data)

## For Linux:  
-  Set open_files_limit present under [mysqld_safe] in my.cnf file to 4096. Sets maximum amount of files openable by Linux.  
-  There are 4 types of I/O scheduler, Test various schedulers and benchmark performance by changing settings found in /etc/rc.d/rc.local  
-  Decrease paging (memory swapping to hard disk) - basic is set to 60, reduce it by smaller percentages to make mysql use more RAM and less hard disk 
-  Dont run any other kind of background services in the mysql server (stuff like Apache, nginx etc.) 
	
## mysql.cnf configuration:  
- **tmp_table_size** = Max amount of memory which can be written in memory to create temporary tables after which is written to disk
- Table is created in disk, if temporary tables are created which contain TEXT or BLOB data type.  
- **memlock** option can be turned on to disable disk writing entirely, but may crash and corrupt data if memory is full  
- **innodb_buffer_pool_size** must be about 50 - 70% of memory available  
Calculate best usage of buffer pool size with the following steps:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. Run the below query:  
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```SHOW GLOBAL STATUS LIKE 'innodb_buffer_pool_page%'; ```   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. Use the formula to get ratio:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```Innodb_buffer_pool_pages_free / Innodb_buffer_pool_pages_total```    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. If ratio is close to 1, then the buffer size is set too high. Reduce it, restart and check performance again  
- **innodb_flush_log_at_trx_commit** - best option set to 0 (durability will suffer if mysqld is crashed), alternative is 2 (durability will suffer if OS is crashed) which is not ACID compliant  
**innodb_flush_method** - "0_Direct" method can be used if RAID with battery_cache is enabled. Else use "0_Sync"  
- **innodb_log_file_size** - Set this to large size if Database is write intensive.
- **innodb_max_dirty_pages_pct** - Value is recorded in %. Higher percentage will improve performance. Default is 90% 
- **tmp_table_size & max_heap_table_size**  
Calculate best usage of tmp_table_size with the following steps:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. Run the below query:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```SHOW GLOBAL STATUS LIKE "%tmp%";```     
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. Use the formula to get ratio on how much data is being written to disk   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```Created_tmp_disk_tables / Created_tmp_tables```    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. Having a lower ratio implies that more data is being written to Disk
- **table_definition_cache** - cache for storing table information data

## MySQL Load Testing:
- Load testing of MySQL is possible with the help of an internal tool in mysql called **mysqlslap**. The help file can be viewed using the following command  
    `mysqlslap --help`
- A more advanced version of load testing can be acheived with the help of **Sysbench**
- **Sysbench** provides general testing of DB regarding CPU performance, I/O performance, mutex (mutual exclusion) connection, Memory Speed, thread performance and database performance.
- [Link to Sysbench Documentation](https://manpages.debian.org/testing/sysbench/sysbench.1.en.html)

## MySQL Profiling:
- Some of the tools used for profiling include
1. mysqltuner - Gives recommendations for best settings [Link to website](www.mysqltuner.com)
2. mysqlreport - Gives record [Link to website](hackmysql.com/mysqlreport)
3. mk-query-profiler - Gives profiled record of a bunch of SQL statements [Link to website](www.maatkit.org)
- Some tools to work with slow queries include
1. mysqldumpslow - Just reads and shows queries which were slow to run
2. mysqlsla
