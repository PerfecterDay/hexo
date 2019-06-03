---
title: mysql 用户权限管理
date: 2019-09-21 09:20:28
tags: mysql
category: sql


---
### 用户管理
1. 创建用户： `CREATE USER 'username'@'ip' IDENTIFIED BY 'passowrd' `;
2. 修改用户密码： `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';` mysql 8  
`update user set password=password('11111111') where xxx;` 8 以前的版本
3. 删除用户： `drop user testuser@'localhost'`


### 授权与收回
SQL标准包括 select、insert、update、delete以及all权限。
1. 授权语句：
    ```
    grant <权限列表>
    on <关系名或视图名>
    to <用户/角色列表>

    grant select on department to public,Amy,Simith
    ```
    public代表系统中所有的当前用户和将来的用户。

2. 收回权限语句：
    ```
    revoke <权限列表>
    on <关系名或视图名>
    from <用户/角色列表>

    revoke select on department from Amy,Simith
    ```
