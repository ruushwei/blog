---
title: mysql创建100张表
date: 2019-04-02T10:54:24+02:00
tags: mysql
categories: mysql
---



使用存储过程，创建100张表

<!--more-->

使用存储过程，创建100张表

```mysql
DELIMITER ;;

create PROCEDURE createTable(a INT)
begin

	DECLARE i INT default 0;
	while i < a do

		set @table_name = concat('xxx_log_',concat(i,''));
		set @sql_text=CONCAT('CREATE TABLE IF NOT EXISTS ', @table_name, '(
      id bigint(20) COMMENT "主键" PRIMARY KEY NOT NULL AUTO_INCREMENT,
      xxx_id int(10) COMMENT "xxxId"
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8');

		PREPARE stmt FROM @sql_text;
		EXECUTE stmt;
		DEALLOCATE PREPARE stmt;
		set i=i+1;
	end while;
end
;;


# 删除存储过程
#drop PROCEDURE createTables;

# 调用存储过程
call createTable(100)

# 列出当前的存储过程
#show procedure status; 

# 显示存储过程的具体内容
#show create procedure createTables 
```







<!--more-->





