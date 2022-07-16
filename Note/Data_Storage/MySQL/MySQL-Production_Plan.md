# 查询场景

## 隐藏敏感信息

### 隐藏姓名

```mysql
SELECT `name` AS `before_encrypt`,
	CASE CHAR_LENGTH(`name`)
	WHEN 2 THEN CONCAT('*',SUBSTRING(`name`, 2, 1))
	WHEN 3 THEN CONCAT(SUBSTRING(`name`, 1, 1),'*',SUBSTRING(`name`, -1, 1))
	ELSE CONCAT(SUBSTRING(`name`, 1, CHAR_LENGTH(`name`) - 2),'**')
	END AS `after_encrypt`
FROM `user`
```



### 隐藏手机号

```mysql
SELECT `mobile` AS `before_encrypt`, 
INSERT(`mobile`, CHAR_LENGTH(`mobile`) - 7, 4, '****') AS `after_encrypt`
FROM `user`
```



### 隐藏身份证号

```mysql
SELECT `id_card_no` AS `before_encrypt`, 
INSERT(`id_card_no`, 7, 8, '********') AS `after_encrypt`
FROM `user`
```

