磁盘扩容


fdisk /dev/sda 

(n ..p... t... 8e ... w)


reboot

vgextend centos /dev/sda3


vgdisplay


lvextend -L+183.9G /dev/centos/root


xfs_growfs /dev/centos/root

数据库服务器安装


docker pull mysql:5.7


mkdir --parents /opt/mysql/logs /opt/mysql/conf /opt/mysql/data


docker run --name mysql5.7 -p 3306:3306 -v /opt/mysql/logs/:/var/log/mysql -v /opt/mysql/conf/:/etc/mysql/conf.d -v /opt/mysql/data/:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7 --restart=always


启动 mongo

mkdir --parents /opt/mongo/configdb /opt/mongo/db


docker run  \
--name mongo \
--restart=always \
-p 27017:27017  \
-v /opt/mongo/configdb:/data/configdb/ \
-v /opt/mongo/db/:/data/db/ \
-d mongo --auth

docker exec -it b0e6a338da63 mongo admin

db.createUser({ user: 'wind', pwd: 'wadmin_2019', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });



启动 redis

docker run --name red -d -p 6379:6379 -v /opt/redis/data:/data redis:latest redis-server --appendonly yes --requirepass "Gomrwind123" 

启动 kafka

docker pull wurstmeister/kafka
docker pull wurstmeister/zookeeper


docker run --name zoo -p 2181:2181 -d wurstmeister/zookeeper --restart=always


docker run -d --name kaf --publish 9092:9092 \
--link zoo:zookeeper \
--env KAFKA_BROKER_ID=100 \
--env HOST_IP=10.0.0.50 \
--env KAFKA_ZOOKEEPER_CONNECT=zoo:2181 \
--env KAFKA_ADVERTISED_PORT=9092 \
--env KAFKA_ADVERTISED_HOST_NAME=10.0.0.50 \
--env KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.0.0.50:9092 \
--restart=always \
--volume /etc/localtime:/etc/localtime \
wurstmeister/kafka

ZSH


sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"


ZSH_THEME="suvash"



echo "export LC_ALL=en_US.UTF-8"  >>  /etc/profile 
source /etc/profile



alias k=kubectl
alias ka="k get pods --all-namespaces"
alias kao="ka -o wide"
alias kl="k logs -f"
alias kn="k get nodes --all-namespaces"
alias kno="k get nodes --all-namespaces -o wide"
alias ks="k get service --all-namespaces"
alias kso="k get service --all-namespaces -o wide"
alias kd="k delete"
alias kb="k describe"

连接VPN

安装 nmcli 的l2tp 组件
yum -y install peel-release
yum install NetworkManager-l2tp

使用 nmcli 创建l2tp连接
nmcli connection add con-name hongkong type vpn vpn-type org.freedesktop.NetworkManager.l2tp  ifname  ens192


修改连接文件
vim /etc/NetworkManager/system-connections/hongkong

[vpn]
gateway=47.56.5.57
ipsec-psk=dubai
password-flags=0
user=devops
service-type=org.freedesktop.NetworkManager.l2tp

[vpn-secrets]
password=UIojvX


重启网卡
/etc/init.d/network restart



连接
nmcli con up hongkong