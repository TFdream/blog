# MySQL

## 列
### 1. 增加列
```
ALTER TABLE `tb_user` ADD COLUMN `address` VARCHAR(40) NOT NULL DEFAULT '' COMMENT '用户住址';
```

### 2. 删除列
```
ALTER TABLE 表名 drop column 列名;
```

### 3. 修改字段长度
```
ALTER TABLE `tb_user` MODIFY COLUMN `single_code` VARCHAR(40);
```

## 索引
### 1.普通索引
```
ALTER TABLE `t_user` ADD KEY INX_biz_id(`loan_id`, `lender_id`, `sub_account_id`, `buy_type`, `phase`);
```

### 2.唯一索引
```
ALTER TABLE `t_user` ADD UNIQUE KEY INX_biz_id(`loan_id`, `lender_id`, `sub_account_id`, `buy_type`, `phase`);
```

### 2.删除索引
```
alter table 表名 drop index 索引名 ;
```


## 有用的脚本
### 1.根据多个字段查找重复数据
```
select * from (select lender_id, loan_id,sub_account_id, buy_type, phase, concat(lender_id, loan_id, sub_account_id, buy_type, phase) as col_t from calendar_repay_1 order by id desc) tmp group by col_t having count(col_t)>1;
```
