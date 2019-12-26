创建webchat用户，密码为123456

CREATE USER 'webchat'@'%' IDENTIFIED BY '123456';

授权
GRANT ALL ON *.* TO 'webchat'@'%';



更改用户密码
SET PASSWORD FOR 'webchat'@'%' = PASSWORD("123456");