# MySQL

## 索引
```
CREATE INDEX `INDEX_apply_id` ON `tb_loan` (`apply_id`);
```

## 增加列
```
ALTER TABLE `tb_user` ADD COLUMN `address` VARCHAR(40) NOT NULL DEFAULT '' COMMENT '用户住址';
```
