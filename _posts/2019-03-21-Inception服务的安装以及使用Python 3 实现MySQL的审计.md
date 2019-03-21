# Inception服务的安装以及使用Python 3 实现MySQL的审计

>https://www.cnblogs.com/wy123/archive/2018/01/21/8322162.html

Inception的备份语句生成原理分析  
>http://www.bubuko.com/infodetail-2320381.html

Inception 配置文件

touch inc.cnf
```
[inception]
general_log=1
general_log_file=inception.log
port=6669
socket=/tmp/inc.socket
character-set-client-handshake=0
character-set-server=utf8
inception_remote_system_password=test
inception_remote_system_user=test
inception_remote_backup_port=3306
inception_remote_backup_host=10.101.130.248
inception_support_charset=utf8mb4
inception_enable_nullable=0
inception_check_primary_key=1
inception_check_column_comment=1
inception_check_table_comment=1
inception_osc_min_table_size=1
inception_osc_bin_dir=/data/temp
inception_osc_chunk_time=0.1
inception_enable_blob_type=1
inception_check_column_default_value=1

#Inception 审核规则
inception_check_autoincrement_datatype=1        #建表时，自增列的类型不为int或者bigint
inception_check_autoincrement_init_value=1      #建表时，自增列的值指定的不为1，则报错
inception_check_autoincrement_name=1            #建表时，如果指定的自增列的名字不为ID
inception_check_column_comment=1                        #建表时，列没有注释
inception_check_column_default_value=0          #检查在建表、修改列、新增列时，列属性是否有默认值
inception_check_dml_limit=1                                     #在DML语句中使用了LIMIT
inception_check_dml_orderby=1                           #在DML语句中使用了Order By
inception_check_dml_where=1                                     #在DML语句中没有WHERE条件
inception_check_identifier=1                            #SQL语句名字检查，如果发现名字中存在除数字、字母、下划线之外的字符时，会报Identifier "invalidname" is invalid, valid options: [a-z,A-Z,0-9,_].
inception_check_index_prefix=1                          #是否检查索引名字前缀为"idx_"，检查唯一索引前缀是不是"uniq_"
inception_check_insert_field=1                          #是否检查插入语句中的列链表的存在性
inception_check_primary_key=1                           #检查是否有主键
inception_check_table_comment=0                         #检查表是否有注释
inception_check_timestamp_default=0                     #检查表是否有timestamp类型指定默认值
inception_enable_autoincrement_unsigned=1       #检查自增列是不是为无符号型
inception_enable_blob_type=0                            #检查是不是支持BLOB字段，包括建表、修改列、新增列操作 默认开启
inception_enable_column_charset=0                       #允许列自己设置字符集
inception_enable_enum_set_bit=0                         #是否支持enum,set,bit数据类型
inception_enable_foreign_key=0                          #是否支持外键
inception_enable_identifer_keyword=0            #SQL语句是否有标识符被写成MySQL的关键字
inception_enable_not_innodb=0                           #存储引擎是否指定为Innodb
inception_enable_nullable=0                                     #创建或者新增列是否允许为NULL
inception_enable_orderby_rand=0                         #是否允许order by rand
inception_enable_partition_table=0                      #是否支持分区表
inception_enable_select_star=0                          #是否允许 Select*
inception_enable_sql_statistic=1                        #备库实例是否存储sql统计信息
inception_max_char_length=16                            #当char类型的长度大于这个值时，是否提示转换为VARCHAR
inception_max_key_parts=5                                       #一个索引中，列的最大个数，超过这个数目则报错
inception_max_keys=16                                           #一个表中，最大的索引数目，超过这个数则报错
inception_max_update_rows=10000                         #在一个修改语句中，预计影响的最大行数，超过这个数就报错
inception_merge_alter_table=1                           #在多个改同一个表的语句出现是，是否提示合成一个
```
