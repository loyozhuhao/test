Harbor（镜像仓库）
镜像仓库使用harbor搭建，部署在gitlab服务器上，主目录为/root/harbor，结合了https证书. 目前存放了一些镜像，包括es，golang等. harbor的配置文件harbor.cfg中有管理员账号密码等信息.

harbor使用docker-compose部署:

# 登录gitlab服务器
ssh gitlab 
cd /root/harbor/harbor 

# 停止harbor服务
docker-compose down

# 启用harbor服务
docker-compose up -d
