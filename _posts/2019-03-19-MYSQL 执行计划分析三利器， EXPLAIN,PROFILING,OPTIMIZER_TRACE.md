# MYSQL 执行计划分析三利器， EXPLAIN,PROFILING,OPTIMIZER_TRACE

## step 1

 使用explain extend 查看执行计划， 5.6后可以加参数 explain format=json xxx 输出json格式的信息
 使用explain 查看执行计划， 5.7 之后直接包含了 filter
 
 查看优化器改写后的SQL
 explain select count(*) from table t ;
 show warnings;
 
##　step 2 

使用profiling详细的列出在每一个步骤消耗的时间，前提是先执行一遍语句。

#打开profiling 的设置
SET profiling = 1;
SHOW VARIABLES LIKE '%profiling%';

#查看队列的内容
show profiles;  
#来查看统计信息
show profile block io,cpu for query 1;
```
[root@localhost tmp]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1908
Server version: 5.7.22-22-log Percona Server (GPL), Release 22, Revision f62d93c

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

[root@mysql.sock][(none)]> SET profiling = 1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

[root@mysql.sock][(none)]> SHOW VARIABLES LIKE '%profiling%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| have_profiling         | YES   |
| profiling              | ON    |
| profiling_history_size | 15    |
+------------------------+-------+
3 rows in set (0.00 sec)

[root@mysql.sock][(none)]> show profiles; 
+----------+------------+-----------------------------------+
| Query_ID | Duration   | Query                             |
+----------+------------+-----------------------------------+
|        1 | 0.00294900 | SHOW VARIABLES LIKE '%profiling%' |
+----------+------------+-----------------------------------+
1 row in set, 1 warning (0.00 sec)

[root@mysql.sock][(none)]> show profile block io,cpu for query 1;
+----------------------+----------+----------+------------+--------------+---------------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+----------------------+----------+----------+------------+--------------+---------------+
| starting             | 0.000068 |     NULL |       NULL |         NULL |          NULL |
| checking permissions | 0.000013 |     NULL |       NULL |         NULL |          NULL |
| Opening tables       | 0.000056 |     NULL |       NULL |         NULL |          NULL |
| init                 | 0.000057 |     NULL |       NULL |         NULL |          NULL |
| System lock          | 0.000006 |     NULL |       NULL |         NULL |          NULL |
| optimizing           | 0.000002 |     NULL |       NULL |         NULL |          NULL |
| optimizing           | 0.000002 |     NULL |       NULL |         NULL |          NULL |
| statistics           | 0.000017 |     NULL |       NULL |         NULL |          NULL |
| preparing            | 0.000014 |     NULL |       NULL |         NULL |          NULL |
| statistics           | 0.000006 |     NULL |       NULL |         NULL |          NULL |
| preparing            | 0.000009 |     NULL |       NULL |         NULL |          NULL |
| executing            | 0.000012 |     NULL |       NULL |         NULL |          NULL |
| Sending data         | 0.000006 |     NULL |       NULL |         NULL |          NULL |
| executing            | 0.000001 |     NULL |       NULL |         NULL |          NULL |
| Sending data         | 0.002486 |     NULL |       NULL |         NULL |          NULL |
| end                  | 0.000004 |     NULL |       NULL |         NULL |          NULL |
| query end            | 0.000005 |     NULL |       NULL |         NULL |          NULL |
| closing tables       | 0.000004 |     NULL |       NULL |         NULL |          NULL |
| removing tmp table   | 0.000144 |     NULL |       NULL |         NULL |          NULL |
| closing tables       | 0.000006 |     NULL |       NULL |         NULL |          NULL |
| freeing items        | 0.000018 |     NULL |       NULL |         NULL |          NULL |
| cleaning up          | 0.000017 |     NULL |       NULL |         NULL |          NULL |
+----------------------+----------+----------+------------+--------------+---------------+
22 rows in set, 1 warning (0.00 sec)

[root@mysql.sock][(none)]> 

```

## step 3  

 

Optimizer trace是MySQL5.6添加的新功能，可以看到大量的内部查询计划产生的信息, 先打开设置，然后执行一次sql,最后查看`information_schema`.`OPTIMIZER_TRACE`的内容

#打开设置
SET optimizer_trace='enabled=on';  
#最大内存根据实际情况而定， 可以不设置
SET OPTIMIZER_TRACE_MAX_MEM_SIZE=1000000;
SET END_MARKERS_IN_JSON=ON;
SET optimizer_trace_limit = 1;
SHOW VARIABLES LIKE '%optimizer_trace%';

#执行所需sql后，查看该表信息即可看到详细的执行过程
SELECT * FROM `information_schema`.`OPTIMIZER_TRACE`;


MySQL索引选择不正确并详细解析OPTIMIZER_TRACE格式

http://blog.csdn.net/melody_mr/article/details/48950601

一 表结构如下: 

CREATE TABLE t_audit_operate_log (
  Fid bigint(16) AUTO_INCREMENT,
  Fcreate_time int(10) unsigned NOT NULL DEFAULT '0',
  Fuser varchar(50) DEFAULT '',
  Fip bigint(16) DEFAULT NULL,
  Foperate_object_id bigint(20) DEFAULT '0',
  PRIMARY KEY (Fid),
  KEY indx_ctime (Fcreate_time),
  KEY indx_user (Fuser),
  KEY indx_objid (Foperate_object_id),
  KEY indx_ip (Fip)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

执行查询:

MySQL> explain select count(*) from t_audit_operate_log where Fuser='XX@XX.com' and Fcreate_time>=1407081600 and Fcreate_time<=1407427199\G

*************************** 1. row ***************************

id: 1

select_type: SIMPLE

table: t_audit_operate_log

type: ref

possible_keys: indx_ctime,indx_user

key: indx_user

key_len: 153

ref: const

rows: 2007326

Extra: Using where

 

发现,使用了一个不合适的索引, 不是很理想，于是改成指定索引：

mysql> explain select count(*) from t_audit_operate_log use index(indx_ctime) where Fuser='CY6016@cyou-inc.com' and Fcreate_time>=1407081600 and Fcreate_time<=1407427199\G

*************************** 1. row ***************************

id: 1

select_type: SIMPLE

table: t_audit_operate_log

type: range

possible_keys: indx_ctime

key: indx_ctime

key_len: 5

ref: NULL

rows: 670092

Extra: Using where

实际执行耗时，后者比前者快了接近10

问题: 很奇怪，优化器为何不选择使用 indx_ctime 索引，而选择了明显会扫描更多行的 indx_user 索引。

分析2个索引的数据量如下:  两个条件的唯一性对比：

select count(*) from t_audit_operate_log where Fuser='XX@XX.com';
+----------+
| count(*) |
+----------+
| 1238382 | 
+----------+

select count(*) from t_audit_operate_log where Fcreate_time>=1407254400 and Fcreate_time<=1407427199;
+----------+
| count(*) |
+----------+
| 198920 | 
+----------+

显然,使用索引indx_ctime好于indx_user,但MySQL却选择了indx_user. 为什么?

于是,使用 OPTIMIZER_TRACE进一步探索.

 

二  OPTIMIZER_TRACE的过程说明

以本处事例简要说明OPTIMIZER_TRACE的过程.

查看OPTIMIZER_TRACE方法：

1.set optimizer_trace='enabled=on';    --- 开启trace

2.set optimizer_trace_max_mem_size=1000000;    --- 设置trace大小

3.set end_markers_in_json=on;    --- 增加trace中注释

4.select * from information_schema.optimizer_trace\G;

 ```

[plain] view plain copy 
 
{\  
  "steps": [\  
    {\  
      "join_preparation": {\  ---优化准备工作  
        "select#": 1,\  
        "steps": [\  
          {\  
            "expanded_query": "/* select#1 */ select count(0) AS `count(*)` from `t_audit_operate_log` where ((`t_audit_operate_log`.`Fuser` = 'XX@XX.com') and (`t_audit_operate_log`.`Fcreate_time` >= 1407081600) and (`t_audit_operate_log`.`Fcreate_time` <= 1407427199))"\  
          }\  
        ] /* steps */\  
      } /* join_preparation */\  
    },\  
    {\  
      "join_optimization": {\  ---优化工作的主要阶段,包括逻辑优化和物理优化两个阶段  
        "select#": 1,\  
        "steps": [\  ---优化工作的主要阶段, 逻辑优化阶段  
          {\  
            "condition_processing": {\  ---逻辑优化,条件化简  
              "condition": "WHERE",\  
              "original_condition": "((`t_audit_operate_log`.`Fuser` = 'XX@XX.com') and (`t_audit_operate_log`.`Fcreate_time` >= 1407081600) and (`t_audit_operate_log`.`Fcreate_time` <= 1407427199))",\  
              "steps": [\  
                {\  
                  "transformation": "equality_propagation",\  ---逻辑优化,条件化简,等式处理  
                  "resulting_condition": "((`t_audit_operate_log`.`Fuser` = 'XX@XX.com') and (`t_audit_operate_log`.`Fcreate_time` >= 1407081600) and (`t_audit_operate_log`.`Fcreate_time` <= 1407427199))"\  
                },\  
                {\  
                  "transformation": "constant_propagation",\  ---逻辑优化,条件化简,常量处理  
                  "resulting_condition": "((`t_audit_operate_log`.`Fuser` = 'XX@XX.com') and (`t_audit_operate_log`.`Fcreate_time` >= 1407081600) and (`t_audit_operate_log`.`Fcreate_time` <= 1407427199))"\  
                },\  
                {\  
                  "transformation": "trivial_condition_removal",\  ---逻辑优化,条件化简,条件去除  
                  "resulting_condition": "((`t_audit_operate_log`.`Fuser` = 'XX@XX.com') and (`t_audit_operate_log`.`Fcreate_time` >= 1407081600) and (`t_audit_operate_log`.`Fcreate_time` <= 1407427199))"\  
                }\  
              ] /* steps */\  
            } /* condition_processing */\  
          },\  ---逻辑优化,条件化简,结束  
          {\  
            "table_dependencies": [\  ---逻辑优化, 找出表之间的相互依赖关系. 非直接可用的优化方式.   
              {\  
                "table": "`t_audit_operate_log`",\  
                "row_may_be_null": false,\  
                "map_bit": 0,\  
                "depends_on_map_bits": [\  
                ] /* depends_on_map_bits */\  
              }\  
            ] /* table_dependencies */\  
          },\  
          {\  
            "ref_optimizer_key_uses": [\   ---逻辑优化,  找出备选的索引  
              {\  
                "table": "`t_audit_operate_log`",\  
                "field": "Fuser",\  
                "equals": "'XX@XX.com'",\  
                "null_rejecting": false\  
              }\  
            ] /* ref_optimizer_key_uses */\  
          },\  
          {\  
            "rows_estimation": [\   ---逻辑优化, 估算每个表的元组个数. 单表上进行全表扫描和索引扫描的代价估算. 每个索引都估算索引扫描代价  
              {\  
                "table": "`t_audit_operate_log`",\  
                "range_analysis": {\  
                  "table_scan": {\---逻辑优化, 估算每个表的元组个数. 单表上进行全表扫描的代价  
                    "rows": 8150516,\  
                    "cost": 1.73e6\  
                  } /* table_scan */,\  
                  "potential_range_indices": [\ ---逻辑优化, 列出备选的索引. 后续版本字符串变为potential_range_indexes  
                    {\  
                      "index": "PRIMARY",\---逻辑优化, 本行表明主键索引不可用  
                      "usable": false,\  
                      "cause": "not_applicable"\  
                    },\  
                    {\  
                      "index": "indx_ctime",\---逻辑优化, 索引indx_ctime  
                      "usable": true,\  
                      "key_parts": [\  
                        "Fcreate_time",\  
                        "Fid"\  
                      ] /* key_parts */\  
                    },\  
                    {\  
                      "index": "indx_user",\---逻辑优化, 索引indx_user  
                      "usable": true,\  
                      "key_parts": [\  
                        "Fuser",\  
                        "Fid"\  
                      ] /* key_parts */\  
                    },\  
                    {\  
                      "index": "indx_objid",\---逻辑优化, 索引  
                      "usable": false,\  
                      "cause": "not_applicable"\  
                    },\  
                    {\  
                      "index": "indx_ip",\---逻辑优化, 索引  
                      "usable": false,\  
                      "cause": "not_applicable"\  
                    }\  
                  ] /* potential_range_indices */,\  
                  "setup_range_conditions": [\ ---逻辑优化, 如果有可下推的条件,则带条件考虑范围查询  
                  ] /* setup_range_conditions */,\  
                  "group_index_range": {\---逻辑优化, 如带有GROUPBY或DISTINCT,则考虑是否有索引可优化这种操作. 并考虑带有MIN/MAX的情况  
                    "chosen": false,\  
                    "cause": "not_group_by_or_distinct"\  
                  } /* group_index_range */,\  
                  "analyzing_range_alternatives": {\---逻辑优化,开始计算每个索引做范围扫描的花费(等值比较是范围扫描的特例)  
                    "range_scan_alternatives": [\  
                      {\  
                        "index": "indx_ctime",\ ---[A]  
                        "ranges": [\  
                          "1407081600 <= Fcreate_time <= 1407427199"\  
                        ] /* ranges */,\  
                        "index_dives_for_eq_ranges": true,\  
                        "rowid_ordered": false,\  
                        "using_mrr": true,\  
                        "index_only": false,\  
                        "rows": 688362,\  
                        "cost": 564553,\ ---逻辑优化,这个索引的代价最小  
                        "chosen": true\ ---逻辑优化,这个索引的代价最小,被选中. (比前面的table_scan 和其他索引的代价都小)  
                      },\  
                      {\  
                        "index": "indx_user",\  
                        "ranges": [\  
                          "XX@XX.com <= Fuser <= XX@XX.com"\  
                        ] /* ranges */,\  
                        "index_dives_for_eq_ranges": true,\  
                        "rowid_ordered": true,\  
                        "using_mrr": true,\  
                        "index_only": false,\  
                        "rows": 1945894,\  
                        "cost": 1.18e6,\  
                        "chosen": false,\  
                        "cause": "cost"\  
                      }\  
                    ] /* range_scan_alternatives */,\  
                    "analyzing_roworder_intersect": {\  
                      "usable": false,\  
                      "cause": "too_few_roworder_scans"\  
                    } /* analyzing_roworder_intersect */\  
                  } /* analyzing_range_alternatives */,\---逻辑优化,开始计算每个索引做范围扫描的花费. 这项工作结算  
                  "chosen_range_access_summary": {\---逻辑优化,开始计算每个索引做范围扫描的花费. 总结本阶段最优的.  
                    "range_access_plan": {\  
                      "type": "range_scan",\  
                      "index": "indx_ctime",\  
                      "rows": 688362,\  
                      "ranges": [\  
                        "1407081600 <= Fcreate_time <= 1407427199"\  
                      ] /* ranges */\  
                    } /* range_access_plan */,\  
                    "rows_for_plan": 688362,\  
                    "cost_for_plan": 564553,\  
                    "chosen": true\    -- 这里看到的cost和rows都比 indx_user 要来的小很多---这个和[A]处是一样的,是信息汇总.  
                  } /* chosen_range_access_summary */\  
                } /* range_analysis */\  
              }\  
            ] /* rows_estimation */\ ---逻辑优化, 估算每个表的元组个数. 行估算结束  
          },\  
          {\  
            "considered_execution_plans": [\ ---物理优化, 开始多表连接的物理优化计算  
              {\  
                "plan_prefix": [\  
                ] /* plan_prefix */,\  
                "table": "`t_audit_operate_log`",\  
                "best_access_path": {\  
                  "considered_access_paths": [\  
                    {\  
                      "access_type": "ref",\ ---物理优化, 计算indx_user索引上使用ref方查找的花费,  
                      "index": "indx_user",\  
                      "rows": 1.95e6,\  
                      "cost": 683515,\  
                      "chosen": true\  
                    },\ ---物理优化, 本应该比较所有的可用索引,即打印出多个格式相同的但索引名不同的内容,这里却没有。推测是bug--没有遍历每一个索引.  
                    {\  
                      "access_type": "range",\---物理优化,猜测对应的是indx_time（没有实例可进行调试，对比5.7的跟踪信息猜测而得）  
                      "rows": 516272,\  
                      "cost": 702225,\---物理优化，代价大于了ref方式的683515，所以没有被选择  
                      "chosen": false\   -- cost比上面看到的增加了很多，但rows没什么变化 ---物理优化，此索引没有被选择  
                    }\  
                  ] /* considered_access_paths */\  
                } /* best_access_path */,\  
                "cost_for_plan": 683515,\ ---物理优化，汇总在best_access_path 阶段得到的结果  
                "rows_for_plan": 1.95e6,\  
                "chosen": true\   -- cost比上面看到的竟然小了很多？虽然rows没啥变化  ---物理优化，汇总在best_access_path 阶段得到的结果  
              }\  
            ] /* considered_execution_plans */\  
          },\  
          {\  
            "attaching_conditions_to_tables": {\---逻辑优化，尽量把条件绑定到对应的表上  
              } /* attaching_conditions_to_tables */\  
          },\  
          {\  
            "refine_plan": [\  
              {\  
                "table": "`t_audit_operate_log`",\---逻辑优化，下推索引条件"pushed_index_condition"；其他条件附加到表上做为过滤条件"table_condition_attached"  
              }\  
            ] /* refine_plan */\  
          }\  
        ] /* steps */\  
      } /* join_optimization */\ \---逻辑优化和物理优化结束  
    },\  
    {\  
      "join_explain": {} /* join_explain */\  
    }\  
  ] /* steps */\  
  
  ```

