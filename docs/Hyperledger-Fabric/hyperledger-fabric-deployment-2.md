# 基于 Hyperledger Fabric v2.2.1 的多机部署 (二)
> 多机部署环境搭建

## 主机配置
```
4vCPUs | 8GB | c3.xlarge.2
Ubuntu 20.04 server 64bit
100 GB Storage
```

## 主机列表
| Nodes  |  Internal IP |  Public IP |
| :----  |  :---------  |  :------   |
|order.example.com | 192.168.0.181 | * |
|peer0.org1.example.com | 192.168.0.129 | * |
|peer0.org2.example.com | 192.168.0.70 |  * |

### Hints: 华为云 ECS内网域名解析:
- [内网域名解析](https://support.huaweicloud.com/productdesc-dns/dns_pd_0005.html)
- [为云服务器配置内网域名](https://support.huaweicloud.com/bestpractice-dns/dns_bestprac_0002.html#dns_bestprac_0002__table11364645122020)


## 基本环境配置
> 以下依赖安装没有特别说明需要在各个主机中安装操作
### 1. Docker 安装
> 十分推荐，一键安装
```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
### 2. docker-compose 安装
```
curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```
### 3. Golang 安装
```
wget https://dl.google.com/go/go1.13.4.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.13.4.linux-amd64.tar.gz

# 设置 go 环境变量
vi ~/.bashrc

# 添加
export PATH=$PATH:/usr/local/go/bin

# 执行
source ~/.bashrc
```
### 4. Node.js 安装
```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
apt install nodejs
```
### 5. 安装示例、二进制文件和 Docker 镜像
```
mkdir fabric && cd fabric && curl -sSL https://bit.ly/2ysbOFE | bash -s

# 设置 fabric bin
vi ~/.bashrc

# 将 bin 目录加入到 PATH:
# export PATH=$PATH:~/fabric/bin
export PATH=/root/fabric/fabric-samples/bin:$PATH

# 执行
source ~/.bashrc
```