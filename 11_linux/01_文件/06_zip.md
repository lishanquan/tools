# zip压缩命令

1. ### 压缩指定文件到目标压缩文件

   这种方法适用于压缩指定的的几个文件到一个新的目标压缩文件中。

   ```shell
   zip target.zip file1 file2 file3
   ```

   

2. ### 压缩指定目录及其子目录下的所有文件到目标压缩文件

   - ##### 压缩到一个新的目标压缩文件中

     ```shell
     zip -r target.zip folder/
     ```

   

3. ### 将压缩文件中的文件解压到指定目录

   ```shell
   unzip -d destination_folder source.zip
   ```

   

4. ### 压缩时排除指定文件或目录

   ```shell
   zip -r target.zip folder -x "*.txt" -x "*.log"
   ```

   

5. 