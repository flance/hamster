### 本文所有内容仅针对oracle 12c版本
---

#### 创建用户ipmpdbadmin并且设定密码为ipmpdbadmin
create user `ipmpdbadmin` identified by `ipmpdbadmin`;

----------

#### 授权所有权限给用户ipmpdbadmin
grant create session, resource,create any table, create any view ,create any index, create any procedure,
alter any table, alter any procedure,create sequence,
drop any table, drop any view, drop any index, drop any procedure,
select any table, insert any table, update any table, delete any table
to `ipmpdbadmin`;

----------

#### 创建临时表空间和数据表空间，并更改用户的表空间
-- 创建临时空间  
	create temporary tablespace `pdbadmin_temp`  
	tempfile `'F:\oracle\oradata\orcl\pdbadmin_temp01.dbf'`  
	size 32m
	autoextend on
	next 32m MAXSIZE unlimited  
	extent management local;


-- 创建数据表空间  
	create tablespace `pdbadmin_data`  
	logging
	datafile `'F:\oracle\oradata\orcl\pdbadmin_data01.dbf'`
	size 10240m
	autoextend on
	next 100m MAXSIZE unlimited  
	extent management local;

-- 更改用户默认表空间  
	alter user `ipmpdbadmin` default tablespace `pdbadmin_data` temporary tablespace `pdbadmin_temp`;  
	//alter user `ipmpdbadmin` quota unlimited on users;--这条语句可能有问题  
	grant unlimited tablespace to `ipmpdbadmin`;

----------

#### 删除用户所有数据，但不包含表空间
drop user `ipmpdbadmin` CASCADE;