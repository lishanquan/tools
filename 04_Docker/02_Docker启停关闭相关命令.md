```bash
# 查看docker版本信息
docker -V 

# 查看docker命令帮助信息
docker --help 

# 启动docker
systemctl start docker.service 

# 停止docker
systemctl stop docker.service 

# 重启docker
systemctl restart docker.service 

# 查看docker当前状态
systemctl status docker 

# 允许docker开机自启动
systemctl enable docker 

# 查看防火墙当前状态
systemctl status firewalld 

# 关闭防火墙（临时关闭）
systemctl stop firewalld.service 

# 永久关闭防火墙
systemctl disable firewalld.service 

# 重启防火墙
systemctl enable firewalld.service 
```