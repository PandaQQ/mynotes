# 基于 Hyperledger Fabric v2.2.1 的多机部署 (一)
> 开启CA服务并注册用戶

## 主机配置
```
4vCPUs | 8GB | c3.xlarge.2
Ubuntu 20.04 server 64bit
100 GB Storage
```

## 主机列表
| Nodes  |  Internal IP |  Public IP |
| :----  |  :---------  |  :------   |
|order.example.com | 192.168.0.181 | 119.8.237.150 |
|peer0.org1.example.com | 192.168.0.129 | 119.8.56.178 |
|peer0.org2.example.com | 192.168.0.70 |  119.8.123.249 |
|ca.example.com | 192.168.0.99 | 159.138.148.225 |

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

## 使用 Fabric-ca 构建 Orderer, Org1 and Org2 CA Server
### 1. 使用Docker构建 CA Order Server 并注册用户
```
# Create CA dir for each CA Order, CA ORG1 & ORG2
mkdir -p ~/organizations/fabric-ca/ordererOrg
mkdir -p ~/organizations/fabric-ca/org1
mkdir -p ~/organizations/fabric-ca/org2

# CA Order Server
docker run --rm -d \
--name ca_orderer \
-p 9054:9054 \
-e FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server \
-e FABRIC_CA_SERVER_CA_NAME=ca-orderer \
-e FABRIC_CA_SERVER_TLS_ENABLED=true \
-e FABRIC_CA_SERVER_PORT=9054 \
-v ~/organizations/fabric-ca/ordererOrg:/etc/hyperledger/fabric-ca-server \
hyperledger/fabric-ca:1.4.9 sh -c 'fabric-ca-server start -b admin:adminpw -d'

# Create Orderer dir for msp usage
mkdir -p ~/organizations/ordererOrganizations/example.com/msp
mkdir -p ~/organizations/ordererOrganizations/example.com/orderers
mkdir -p ~/organizations/ordererOrganizations/example.com/orderers/example.com
mkdir -p ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com


mkdir -p ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts
mkdir -p ~/organizations/ordererOrganizations/example.com/msp/tlscacerts
mkdir -p ~/organizations/ordererOrganizations/example.com/users
mkdir -p ~/organizations/ordererOrganizations/example.com/users/Admin@example.com


# Register Order CA

export FABRIC_CA_CLIENT_HOME=~/organizations/ordererOrganizations/example.com

# Enroll the CA admin
fabric-ca-client enroll -u https://admin:adminpw@localhost:9054 --caname ca-orderer --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem

echo 'NodeOUs:
Enable: true
ClientOUIdentifier:
    Certificate: cacerts/localhost-9054-ca-orderer.pem
    OrganizationalUnitIdentifier: client
PeerOUIdentifier:
    Certificate: cacerts/localhost-9054-ca-orderer.pem
    OrganizationalUnitIdentifier: peer
AdminOUIdentifier:
    Certificate: cacerts/localhost-9054-ca-orderer.pem
    OrganizationalUnitIdentifier: admin
OrdererOUIdentifier:
    Certificate: cacerts/localhost-9054-ca-orderer.pem
    OrganizationalUnitIdentifier: orderer' >~/organizations/ordererOrganizations/example.com/msp/config.yaml

# "Register orderer"
fabric-ca-client register --caname ca-orderer --id.name orderer --id.secret ordererpw --id.type orderer --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem

# "Register the orderer admin"
fabric-ca-client register --caname ca-orderer --id.name ordererAdmin --id.secret ordererAdminpw --id.type admin --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem

# "Generate the orderer msp"
fabric-ca-client enroll -u https://orderer:ordererpw@localhost:9054 --caname ca-orderer -M ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp --csr.hosts orderer.example.com --csr.hosts localhost --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem


cp ~/organizations/ordererOrganizations/example.com/msp/config.yaml ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/config.yaml

# "Generate the orderer-tls certificates"
fabric-ca-client enroll -u https://orderer:ordererpw@localhost:9054 --caname ca-orderer -M ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls --enrollment.profile tls --csr.hosts orderer.example.com --csr.hosts localhost --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem


cp ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/tlscacerts/* ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
cp ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/signcerts/* ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
cp ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/keystore/* ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
cp ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/tlscacerts/* ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
cp ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/tlscacerts/* ~/organizations/ordererOrganizations/example.com/msp/tlscacerts/tlsca.example.com-cert.pem


# "Generate the admin msp"
fabric-ca-client enroll -u https://ordererAdmin:ordererAdminpw@localhost:9054 --caname ca-orderer -M ~/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem

cp ~/organizations/ordererOrganizations/example.com/msp/config.yaml ~/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/config.yaml



```
### 2. 使用Docker构建 CA Org1 Server 并注册用户
```
# CA Org1 Server
docker run --rm -d \
--name ca_org1 \
-p 7054:7054 \
-e FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server \
-e FABRIC_CA_SERVER_CA_NAME=ca-org1 \
-e FABRIC_CA_SERVER_TLS_ENABLED=true \
-e FABRIC_CA_SERVER_PORT=7054 \
-v ~/organizations/fabric-ca/org1:/etc/hyperledger/fabric-ca-server \
hyperledger/fabric-ca:1.4.9 sh -c 'fabric-ca-server start -b admin:adminpw -d'


# Create Org1 dir for msp usage
mkdir -p ~/organizations/peerOrganizations/org1.example.com/msp
mkdir -p ~/organizations/peerOrganizations/org1.example.com/peers
mkdir -p ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com
mkdir -p ~/organizations/peerOrganizations/org1.example.com/msp/tlscacerts
mkdir -p ~/organizations/peerOrganizations/org1.example.com/tlsca
mkdir -p ~/organizations/peerOrganizations/org1.example.com/ca
mkdir -p ~/organizations/peerOrganizations/org1.example.com/users    
mkdir -p ~/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com
mkdir -p ~/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com


# Register Org1 CA
export FABRIC_CA_CLIENT_HOME=~/organizations/peerOrganizations/org1.example.com/

# "Enroll the CA admin"
fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-org1 --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

echo 'NodeOUs:
Enable: true
ClientOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-org1.pem
    OrganizationalUnitIdentifier: client
PeerOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-org1.pem
    OrganizationalUnitIdentifier: peer
AdminOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-org1.pem
    OrganizationalUnitIdentifier: admin
OrdererOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-org1.pem
    OrganizationalUnitIdentifier: orderer' >~/organizations/peerOrganizations/org1.example.com/msp/config.yaml

# "Register peer0"
fabric-ca-client register --caname ca-org1 --id.name peer0 --id.secret peer0pw --id.type peer --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

# "Register user"
fabric-ca-client register --caname ca-org1 --id.name user1 --id.secret user1pw --id.type client --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

# "Register the org admin"
fabric-ca-client register --caname ca-org1 --id.name org1admin --id.secret org1adminpw --id.type admin --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

# "Generate the peer0 msp"
fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org1 -M ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp --csr.hosts peer0.org1.example.com --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

cp ~/organizations/peerOrganizations/org1.example.com/msp/config.yaml ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/config.yaml

# "Generate the peer0-tls certificates"
fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org1 -M ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls --enrollment.profile tls --csr.hosts peer0.org1.example.com --csr.hosts localhost --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem


cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/signcerts/* ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/keystore/* ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org1.example.com/msp/tlscacerts/ca.crt
cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem
cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/cacerts/* ~/organizations/peerOrganizations/org1.example.com/ca/ca.org1.example.com-cert.pem


#  "Generate the user msp"
fabric-ca-client enroll -u https://user1:user1pw@localhost:7054 --caname ca-org1 -M ~/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

cp ~/organizations/peerOrganizations/org1.example.com/msp/config.yaml ~/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/config.yaml

# "Generate the org admin msp"
fabric-ca-client enroll -u https://org1admin:org1adminpw@localhost:7054 --caname ca-org1 -M ~/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

cp ~/organizations/peerOrganizations/org1.example.com/msp/config.yaml ~/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/config.yaml
```
### 3. 使用Docker构建 CA Org2 Server 并注册用户
```
# CA Org2 Server
docker run --rm -d \
--name ca_org2 \
-p 8054:8054 \
-e FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server \
-e FABRIC_CA_SERVER_CA_NAME=ca-org2 \
-e FABRIC_CA_SERVER_TLS_ENABLED=true \
-e FABRIC_CA_SERVER_PORT=8054 \
-v ~/organizations/fabric-ca/org2:/etc/hyperledger/fabric-ca-server \
hyperledger/fabric-ca:1.4.9 sh -c 'fabric-ca-server start -b admin:adminpw -d'

# Create Org2 Dir for msp usage
mkdir -p ~/organizations/peerOrganizations/org2.example.com/msp
mkdir -p ~/organizations/peerOrganizations/org2.example.com/peers
mkdir -p ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com
mkdir -p ~/organizations/peerOrganizations/org2.example.com/msp/tlscacerts
mkdir -p ~/organizations/peerOrganizations/org2.example.com/tlsca
mkdir -p ~/organizations/peerOrganizations/org2.example.com/ca
mkdir -p ~/organizations/peerOrganizations/org2.example.com/users
mkdir -p ~/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com
mkdir -p ~/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com


# Register Org2 CA
export FABRIC_CA_CLIENT_HOME=~/organizations/peerOrganizations/org2.example.com/

# "Enroll the CA admin"
fabric-ca-client enroll -u https://admin:adminpw@localhost:8054 --caname ca-org2 --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

echo 'NodeOUs:
Enable: true
ClientOUIdentifier:
    Certificate: cacerts/localhost-8054-ca-org2.pem
    OrganizationalUnitIdentifier: client
PeerOUIdentifier:
    Certificate: cacerts/localhost-8054-ca-org2.pem
    OrganizationalUnitIdentifier: peer
AdminOUIdentifier:
    Certificate: cacerts/localhost-8054-ca-org2.pem
    OrganizationalUnitIdentifier: admin
OrdererOUIdentifier:
    Certificate: cacerts/localhost-8054-ca-org2.pem
    OrganizationalUnitIdentifier: orderer' >~/organizations/peerOrganizations/org2.example.com/msp/config.yaml

# "Register peer0"
fabric-ca-client register --caname ca-org2 --id.name peer0 --id.secret peer0pw --id.type peer --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

# "Register user"
fabric-ca-client register --caname ca-org2 --id.name user1 --id.secret user1pw --id.type client --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

# "Register the org admin"
fabric-ca-client register --caname ca-org2 --id.name org2admin --id.secret org2adminpw --id.type admin --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

# "Generate the peer0 msp"
fabric-ca-client enroll -u https://peer0:peer0pw@localhost:8054 --caname ca-org2 -M ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp --csr.hosts peer0.org2.example.com --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

cp ~/organizations/peerOrganizations/org2.example.com/msp/config.yaml ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/config.yaml

# "Generate the peer0-tls certificates"
fabric-ca-client enroll -u https://peer0:peer0pw@localhost:8054 --caname ca-org2 -M ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls --enrollment.profile tls --csr.hosts peer0.org2.example.com --csr.hosts localhost --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/signcerts/* ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt
cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/keystore/* ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key
cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org2.example.com/msp/tlscacerts/ca.crt
cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem
cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/cacerts/* ~/organizations/peerOrganizations/org2.example.com/ca/ca.org2.example.com-cert.pem

# "Generate the user msp"
fabric-ca-client enroll -u https://user1:user1pw@localhost:8054 --caname ca-org2 -M ~/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

cp ~/organizations/peerOrganizations/org2.example.com/msp/config.yaml ~/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/config.yaml

# "Generate the org admin msp"
fabric-ca-client enroll -u https://org2admin:org2adminpw@localhost:8054 --caname ca-org2 -M ~/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

cp ~/organizations/peerOrganizations/org2.example.com/msp/config.yaml ~/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/config.yaml
```

## 创建 Org1 和 Org2 的CCP json 和 yaml 文件
> 这一步比较简单，使用 fabric-sample 中的脚步可以直接创建。
> 脚本如下:
```
#!/bin/bash
function one_line_pem {
    echo "`awk 'NF {sub(/\\n/, ""); printf "%s\\\\\\\n",$0;}' $1`"
}

function json_ccp {
    local PP=$(one_line_pem $4)
    local CP=$(one_line_pem $5)
    sed -e "s/\${ORG}/$1/" \
        -e "s/\${P0PORT}/$2/" \
        -e "s/\${CAPORT}/$3/" \
        -e "s#\${PEERPEM}#$PP#" \
        -e "s#\${CAPEM}#$CP#" \
        organizations/ccp-template.json
}

function yaml_ccp {
    local PP=$(one_line_pem $4)
    local CP=$(one_line_pem $5)
    sed -e "s/\${ORG}/$1/" \
        -e "s/\${P0PORT}/$2/" \
        -e "s/\${CAPORT}/$3/" \
        -e "s#\${PEERPEM}#$PP#" \
        -e "s#\${CAPEM}#$CP#" \
        organizations/ccp-template.yaml | sed -e $'s/\\\\n/\\\n          /g'
}

ORG=1
P0PORT=7051
CAPORT=7054
PEERPEM=organizations/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem
CAPEM=organizations/peerOrganizations/org1.example.com/ca/ca.org1.example.com-cert.pem

echo "$(json_ccp $ORG $P0PORT $CAPORT $PEERPEM $CAPEM)" > organizations/peerOrganizations/org1.example.com/connection-org1.json
echo "$(yaml_ccp $ORG $P0PORT $CAPORT $PEERPEM $CAPEM)" > organizations/peerOrganizations/org1.example.com/connection-org1.yaml

ORG=2
P0PORT=9051
CAPORT=8054
PEERPEM=organizations/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem
CAPEM=organizations/peerOrganizations/org2.example.com/ca/ca.org2.example.com-cert.pem

echo "$(json_ccp $ORG $P0PORT $CAPORT $PEERPEM $CAPEM)" > organizations/peerOrganizations/org2.example.com/connection-org2.json
echo "$(yaml_ccp $ORG $P0PORT $CAPORT $PEERPEM $CAPEM)" > organizations/peerOrganizations/org2.example.com/connection-org2.yaml

```

> connection的json样本文件:
```
{
    "name": "test-network-org${ORG}",
    "version": "1.0.0",
    "client": {
        "organization": "Org${ORG}",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300"
                }
            }
        }
    },
    "organizations": {
        "Org${ORG}": {
            "mspid": "Org${ORG}MSP",
            "peers": [
                "peer0.org${ORG}.example.com"
            ],
            "certificateAuthorities": [
                "ca.org${ORG}.example.com"
            ]
        }
    },
    "peers": {
        "peer0.org${ORG}.example.com": {
            "url": "grpcs://localhost:${P0PORT}",
            "tlsCACerts": {
                "pem": "${PEERPEM}"
            },
            "grpcOptions": {
                "ssl-target-name-override": "peer0.org${ORG}.example.com",
                "hostnameOverride": "peer0.org${ORG}.example.com"
            }
        }
    },
    "certificateAuthorities": {
        "ca.org${ORG}.example.com": {
            "url": "https://localhost:${CAPORT}",
            "caName": "ca-org${ORG}",
            "tlsCACerts": {
                "pem": ["${CAPEM}"]
            },
            "httpOptions": {
                "verify": false
            }
        }
    }
}

```

> connection的yaml样本文件:
```
---
name: test-network-org${ORG}
version: 1.0.0
client:
  organization: Org${ORG}
  connection:
    timeout:
      peer:
        endorser: '300'
organizations:
  Org${ORG}:
    mspid: Org${ORG}MSP
    peers:
    - peer0.org${ORG}.example.com
    certificateAuthorities:
    - ca.org${ORG}.example.com
peers:
  peer0.org${ORG}.example.com:
    url: grpcs://localhost:${P0PORT}
    tlsCACerts:
      pem: |
          ${PEERPEM}
    grpcOptions:
      ssl-target-name-override: peer0.org${ORG}.example.com
      hostnameOverride: peer0.org${ORG}.example.com
certificateAuthorities:
  ca.org${ORG}.example.com:
    url: https://localhost:${CAPORT}
    caName: ca-org${ORG}
    tlsCACerts:
      pem: 
        - |
          ${CAPEM}
    httpOptions:
      verify: false

```