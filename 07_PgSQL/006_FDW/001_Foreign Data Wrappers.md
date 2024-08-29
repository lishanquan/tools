# Foreign Data Wrappers（外部数据包装器）

## 创建

1. ### 安装postgres_fdw扩展与授权

   ```sql
   CREATE EXTENSION postgres_fdw;
   ```

   如果需要授权：

   ```sql
   grant usage on foreign data wrapper postgres_fdw to u1;
   ```

2. ### 创建一个外部服务器

   ```sql
   CREATE SERVER foreign_server 
   FOREIGN DATA WRAPPER postgres_fdw
   OPTIONS (address 'pgsql', port '5432', dbname 'chow_crm');
   ```

3. ### 定义用户映射来标识将在远程服务器上使用的角色

   ```sql
   CREATE USER MAPPING FOR cts
   SERVER foreign_server
   OPTIONS (user 'cts', password 'Cts202402');
   ```

4. ### 创建外部表（从外部导入）

   **全量导入**

   ```sql
   CREATE SCHEMA ft;
   import foreign schema public from server foreign_server into ft;
   ```

   - #### 跳过A表

     ```sql
     import FOREIGN SCHEMA public except (A) from server foreign_server into ft;
     ```

   - #### 只导入B表

     ```sql
     import FOREIGN SCHEMA public limit to (B) from server foreign_server into ft;
     ```

5. ### 相关语句

   ```sql
   select * from pg_foreign_data_wrapper;
   
   select * from pg_foreign_server ;
   
   select * from pg_user_mappings;
   
   ALTER SERVER foreign_server OPTIONS (DROP host, DROP port);
   ```

   

6. 

