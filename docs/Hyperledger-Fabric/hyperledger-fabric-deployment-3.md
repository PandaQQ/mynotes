# 基于 Hyperledger Fabric v2.2.1 的多机部署 (三)
>A. 部署 Orderer 在 orderer.example.com
>B. 部署 Org1 Peer0 CA 和 Org2 Peer0 CA
>C. 创建 Org1 和 Org2 的CCP json 和 yaml 文件

## A. 使用 fabric-ca 构建 Orderer CA Server
### 1. 使用Docker构建 CA Order Server 并注册用户
```
# Create CA dir for CA Order Container Usage
mkdir -p ~/organizations/fabric-ca/ordererOrg

# CA Order Server
docker run --rm -d \
--name ca_orderer \
-p 7054:7054 \
-e FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server \
-e FABRIC_CA_SERVER_CA_NAME=ca-orderer \
-e FABRIC_CA_SERVER_TLS_ENABLED=true \
-e FABRIC_CA_SERVER_PORT=7054 \
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

# Export fabric ca client path
export FABRIC_CA_CLIENT_HOME=~/organizations/ordererOrganizations/example.com

# Enroll the CA Admin
fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-orderer --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem
# INFO
2020/11/11 11:48:26 [INFO] Created a default configuration file at /root/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2020/11/11 11:48:26 [INFO] TLS Enabled
2020/11/11 11:48:26 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 11:48:26 [INFO] encoded CSR
2020/11/11 11:48:26 [INFO] Stored client certificate at /root/organizations/ordererOrganizations/example.com/msp/signcerts/cert.pem
2020/11/11 11:48:26 [INFO] Stored root CA certificate at /root/organizations/ordererOrganizations/example.com/msp/cacerts/localhost-7054-ca-orderer.pem
2020/11/11 11:48:26 [INFO] Stored Issuer public key at /root/organizations/ordererOrganizations/example.com/msp/IssuerPublicKey
2020/11/11 11:48:26 [INFO] Stored Issuer revocation public key at /root/organizations/ordererOrganizations/example.com/msp/IssuerRevocationPublicKey
# EO INFO

# Create CA config.yaml file
echo 'NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-orderer.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-orderer.pem
    OrganizationalUnitIdentifier: peer
  AdminOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-orderer.pem
    OrganizationalUnitIdentifier: admin
  OrdererOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-orderer.pem
    OrganizationalUnitIdentifier: orderer' >~/organizations/ordererOrganizations/example.com/msp/config.yaml

# "Register orderer"
fabric-ca-client register --caname ca-orderer --id.name orderer --id.secret ordererpw --id.type orderer --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem
# IFNO
2020/11/11 11:56:36 [INFO] Configuration file location: /root/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2020/11/11 11:56:36 [INFO] TLS Enabled
2020/11/11 11:56:36 [INFO] TLS Enabled
Password: ordererpw
# EO INFO

# "Register the orderer admin"
fabric-ca-client register --caname ca-orderer --id.name ordererAdmin --id.secret ordererAdminpw --id.type admin --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem
# INFO
2020/11/11 13:52:45 [INFO] Configuration file location: /root/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2020/11/11 13:52:45 [INFO] TLS Enabled
2020/11/11 13:52:45 [INFO] TLS Enabled
Password: ordererAdminpw
# EO INFO

# "Generate the orderer msp"
fabric-ca-client enroll -u https://orderer:ordererpw@localhost:7054 --caname ca-orderer -M ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp --csr.hosts orderer.example.com --csr.hosts localhost --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem
# INFO
2020/11/11 13:53:57 [INFO] TLS Enabled
2020/11/11 13:53:57 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 13:53:57 [INFO] encoded CSR
2020/11/11 13:53:57 [INFO] Stored client certificate at /root/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/signcerts/cert.pem
2020/11/11 13:53:57 [INFO] Stored root CA certificate at /root/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/localhost-7054-ca-orderer.pem
2020/11/11 13:53:57 [INFO] Stored Issuer public key at /root/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/IssuerPublicKey
2020/11/11 13:53:57 [INFO] Stored Issuer revocation public key at /root/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/IssuerRevocationPublicKe
# EO INFO

cp ~/organizations/ordererOrganizations/example.com/msp/config.yaml ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/config.yaml

# "Generate the orderer-tls certificates"
fabric-ca-client enroll -u https://orderer:ordererpw@localhost:7054 --caname ca-orderer -M ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls --enrollment.profile tls --csr.hosts orderer.example.com --csr.hosts localhost --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem

# INFO
2020/11/11 13:57:37 [INFO] TLS Enabled
2020/11/11 13:57:37 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 13:57:37 [INFO] encoded CSR
2020/11/11 13:57:37 [INFO] Stored client certificate at /root/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/signcerts/cert.pem
2020/11/11 13:57:37 [INFO] Stored TLS root CA certificate at /root/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/tlscacerts/tls-localhost-7054-ca-orderer.pem
2020/11/11 13:57:37 [INFO] Stored Issuer public key at /root/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/IssuerPublicKey
2020/11/11 13:57:37 [INFO] Stored Issuer revocation public key at /root/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/IssuerRevocationPublicKey
# EO INFO

cp ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/tlscacerts/* ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
cp ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/signcerts/* ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
cp ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/keystore/* ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
cp ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/tlscacerts/* ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
cp ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/tlscacerts/* ~/organizations/ordererOrganizations/example.com/msp/tlscacerts/tlsca.example.com-cert.pem

# "Generate the admin msp"
fabric-ca-client enroll -u https://ordererAdmin:ordererAdminpw@localhost:7054 --caname ca-orderer -M ~/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp --tls.certfiles ~/organizations/fabric-ca/ordererOrg/tls-cert.pem

# INFO
2020/11/11 14:25:30 [INFO] TLS Enabled
2020/11/11 14:25:30 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 14:25:30 [INFO] encoded CSR
2020/11/11 14:25:30 [INFO] Stored client certificate at /root/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/cert.pem
2020/11/11 14:25:30 [INFO] Stored root CA certificate at /root/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/cacerts/localhost-7054-ca-orderer.pem
2020/11/11 14:25:30 [INFO] Stored Issuer public key at /root/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/IssuerPublicKey
2020/11/11 14:25:30 [INFO] Stored Issuer revocation public key at /root/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/IssuerRevocationPublicKey
# EOINF
cp ~/organizations/ordererOrganizations/example.com/msp/config.yaml ~/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/config.yaml
```
### 2. CA Orderer Server 生成证书文件结构
```
ordererOrganizations/
└── example.com
    ├── fabric-ca-client-config.yaml
    ├── msp
    │   ├── IssuerPublicKey
    │   ├── IssuerRevocationPublicKey
    │   ├── cacerts
    │   │   └── localhost-7054-ca-orderer.pem
    │   ├── config.yaml
    │   ├── keystore
    │   │   └── a15773637ad69ab6c664ee419e813348351fb920c444050d5e1b98906a92d2ae_sk
    │   ├── signcerts
    │   │   └── cert.pem
    │   ├── tlscacerts
    │   │   └── tlsca.example.com-cert.pem
    │   └── user
    ├── orderers
    │   ├── example.com
    │   └── orderer.example.com
    │       ├── msp
    │       │   ├── IssuerPublicKey
    │       │   ├── IssuerRevocationPublicKey
    │       │   ├── cacerts
    │       │   │   └── localhost-7054-ca-orderer.pem
    │       │   ├── config.yaml
    │       │   ├── keystore
    │       │   │   └── de1e31972c65b12261c363a883cf42c283694841a4c6612622dac4509b787a62_sk
    │       │   ├── signcerts
    │       │   │   └── cert.pem
    │       │   ├── tlscacerts
    │       │   │   └── tlsca.example.com-cert.pem
    │       │   └── user
    │       └── tls
    │           ├── IssuerPublicKey
    │           ├── IssuerRevocationPublicKey
    │           ├── ca.crt
    │           ├── cacerts
    │           ├── keystore
    │           │   └── ee4f582d8e0c6a0bc417eaba11c7e3aaebf1dd477a412f7b817f9c38506eadc5_sk
    │           ├── server.crt
    │           ├── server.key
    │           ├── signcerts
    │           │   └── cert.pem
    │           ├── tlscacerts
    │           │   └── tls-localhost-7054-ca-orderer.pem
    │           └── user
    └── users
        └── Admin@example.com
            └── msp
                ├── IssuerPublicKey
                ├── IssuerRevocationPublicKey
                ├── cacerts
                │   └── localhost-7054-ca-orderer.pem
                ├── config.yaml
                ├── keystore
                │   └── eb78527c98c4d5db44df53df1bc4906f07a02790cbb65d863b4122825a30361c_sk
                ├── signcerts
                │   └── cert.pem
                └── user

29 directories, 29 files
```

## B. 部署 CA Server 到 Org1 Peer0 和 Org2 Peer0 服务器
#### 3.1 主机列表
| Nodes  |  Internal IP |  Public IP |
| :----  |  :---------  |  :------   |
|peer0.org1.example.com | 192.168.0.129 | 119.8.56.178 |
|peer0.org2.example.com | 192.168.0.70 |  119.8.123.249 |

#### 3.2  部署 CA Server 到 Org1 Peer0
```
# Create CA dir CA ORG1
mkdir -p ~/organizations/fabric-ca/org1

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

#INFO
2020/11/11 17:10:48 [INFO] Created a default configuration file at /root/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2020/11/11 17:10:48 [INFO] TLS Enabled
2020/11/11 17:10:48 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 17:10:48 [INFO] encoded CSR
2020/11/11 17:10:48 [INFO] Stored client certificate at /root/organizations/peerOrganizations/org1.example.com/msp/signcerts/cert.pem
2020/11/11 17:10:48 [INFO] Stored root CA certificate at /root/organizations/peerOrganizations/org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2020/11/11 17:10:48 [INFO] Stored Issuer public key at /root/organizations/peerOrganizations/org1.example.com/msp/IssuerPublicKey
2020/11/11 17:10:48 [INFO] Stored Issuer revocation public key at /root/organizations/peerOrganizations/org1.example.com/msp/IssuerRevocationPublicKey
#EOINFO

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

# INFO
2020/11/11 17:24:57 [INFO] Configuration file location: /root/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2020/11/11 17:24:57 [INFO] TLS Enabled
2020/11/11 17:24:57 [INFO] TLS Enabled
Password: peer0pw
# EOINFO

# "Register user"
fabric-ca-client register --caname ca-org1 --id.name user1 --id.secret user1pw --id.type client --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

# INFO
2020/11/11 17:32:24 [INFO] Configuration file location: /root/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2020/11/11 17:32:24 [INFO] TLS Enabled
2020/11/11 17:32:24 [INFO] TLS Enabled
Password: user1pw
# EOINFO

# "Register the org admin"
fabric-ca-client register --caname ca-org1 --id.name org1admin --id.secret org1adminpw --id.type admin --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

#INFO
2020/11/11 17:33:39 [INFO] Configuration file location: /root/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2020/11/11 17:33:39 [INFO] TLS Enabled
2020/11/11 17:33:39 [INFO] TLS Enabled
Password: org1adminpw
# EOINFO

# "Generate the peer0 msp"
fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org1 -M ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp --csr.hosts peer0.org1.example.com --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

# INFO
2020/11/11 17:34:58 [INFO] TLS Enabled
2020/11/11 17:34:58 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 17:34:58 [INFO] encoded CSR
2020/11/11 17:34:58 [INFO] Stored client certificate at /root/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/signcerts/cert.pem
2020/11/11 17:34:58 [INFO] Stored root CA certificate at /root/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2020/11/11 17:34:58 [INFO] Stored Issuer public key at /root/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/IssuerPublicKey
2020/11/11 17:34:58 [INFO] Stored Issuer revocation public key at /root/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/IssuerRevocationPublicKey
# EOINFO

cp ~/organizations/peerOrganizations/org1.example.com/msp/config.yaml ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/config.yaml

# "Generate the peer0-tls certificates"
fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org1 -M ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls --enrollment.profile tls --csr.hosts peer0.org1.example.com --csr.hosts localhost --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

# INFO
2020/11/11 17:37:05 [INFO] TLS Enabled
2020/11/11 17:37:05 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 17:37:05 [INFO] encoded CSR
2020/11/11 17:37:05 [INFO] Stored client certificate at /root/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/signcerts/cert.pem
2020/11/11 17:37:05 [INFO] Stored TLS root CA certificate at /root/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlscacerts/tls-localhost-7054-ca-org1.pem
2020/11/11 17:37:05 [INFO] Stored Issuer public key at /root/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/IssuerPublicKey
2020/11/11 17:37:05 [INFO] Stored Issuer revocation public key at /root/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/IssuerRevocationPublicKey
# EOINFO


cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/signcerts/* ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/keystore/* ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org1.example.com/msp/tlscacerts/ca.crt
cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem
cp ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/cacerts/* ~/organizations/peerOrganizations/org1.example.com/ca/ca.org1.example.com-cert.pem

#  "Generate the user msp"
fabric-ca-client enroll -u https://user1:user1pw@localhost:7054 --caname ca-org1 -M ~/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

# INFO
2020/11/11 17:38:23 [INFO] TLS Enabled
2020/11/11 17:38:23 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 17:38:23 [INFO] encoded CSR
2020/11/11 17:38:23 [INFO] Stored client certificate at /root/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/signcerts/cert.pem
2020/11/11 17:38:23 [INFO] Stored root CA certificate at /root/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2020/11/11 17:38:23 [INFO] Stored Issuer public key at /root/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/IssuerPublicKey
2020/11/11 17:38:23 [INFO] Stored Issuer revocation public key at /root/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/IssuerRevocationPublicKey
# EOINFO

cp ~/organizations/peerOrganizations/org1.example.com/msp/config.yaml ~/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/config.yaml

# "Generate the org admin msp"
fabric-ca-client enroll -u https://org1admin:org1adminpw@localhost:7054 --caname ca-org1 -M ~/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp --tls.certfiles ~/organizations/fabric-ca/org1/tls-cert.pem

# INFO
2020/11/11 17:40:22 [INFO] TLS Enabled
2020/11/11 17:40:22 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 17:40:22 [INFO] encoded CSR
2020/11/11 17:40:22 [INFO] Stored client certificate at /root/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/cert.pem
2020/11/11 17:40:22 [INFO] Stored root CA certificate at /root/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2020/11/11 17:40:22 [INFO] Stored Issuer public key at /root/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/IssuerPublicKey
2020/11/11 17:40:22 [INFO] Stored Issuer revocation public key at /root/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/IssuerRevocationPublicKey
# EOINFO

cp ~/organizations/peerOrganizations/org1.example.com/msp/config.yaml ~/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/config.yaml
```
>  CA Org1 Server 生成证书文件结构
```
.
└── org1.example.com
    ├── ca
    │   └── ca.org1.example.com-cert.pem
    ├── fabric-ca-client-config.yaml
    ├── msp
    │   ├── IssuerPublicKey
    │   ├── IssuerRevocationPublicKey
    │   ├── cacerts
    │   │   └── localhost-7054-ca-org1.pem
    │   ├── config.yaml
    │   ├── keystore
    │   │   └── 675e10b4e0d4be8f6485fdd364c15853e590277ddcd17af2055d8e2e145aed02_sk
    │   ├── signcerts
    │   │   └── cert.pem
    │   ├── tlscacerts
    │   │   └── ca.crt
    │   └── user
    ├── peers
    │   └── peer0.org1.example.com
    │       ├── msp
    │       │   ├── IssuerPublicKey
    │       │   ├── IssuerRevocationPublicKey
    │       │   ├── cacerts
    │       │   │   └── localhost-7054-ca-org1.pem
    │       │   ├── config.yaml
    │       │   ├── keystore
    │       │   │   └── 33122ea3aca901114e004c43a253b45a28e2f7e42add91eb34b6f2135b375003_sk
    │       │   ├── signcerts
    │       │   │   └── cert.pem
    │       │   └── user
    │       └── tls
    │           ├── IssuerPublicKey
    │           ├── IssuerRevocationPublicKey
    │           ├── ca.crt
    │           ├── cacerts
    │           ├── keystore
    │           │   └── 8ddebe35b8906f9612c564d99c78452a71db35c2157f4d0e020e1ea30e725bf5_sk
    │           ├── server.crt
    │           ├── server.key
    │           ├── signcerts
    │           │   └── cert.pem
    │           ├── tlscacerts
    │           │   └── tls-localhost-7054-ca-org1.pem
    │           └── user
    ├── tlsca
    │   └── tlsca.org1.example.com-cert.pem
    └── users
        ├── Admin@org1.example.com
        │   └── msp
        │       ├── IssuerPublicKey
        │       ├── IssuerRevocationPublicKey
        │       ├── cacerts
        │       │   └── localhost-7054-ca-org1.pem
        │       ├── config.yaml
        │       ├── keystore
        │       │   └── d6c4dab75919ebba9489e363df768436c886241d30e51ff318997fddf73727f0_sk
        │       ├── signcerts
        │       │   └── cert.pem
        │       └── user
        └── User1@org1.example.com
            └── msp
                ├── IssuerPublicKey
                ├── IssuerRevocationPublicKey
                ├── cacerts
                │   └── localhost-7054-ca-org1.pem
                ├── config.yaml
                ├── keystore
                │   └── 483892241b60cb4d0619d764acae61b582fefc23895ad3089dc1878d39959f59_sk
                ├── signcerts
                │   └── cert.pem
                └── user

35 directories, 36 files
```

#### 3.3  部署 CA Server 到 Org2 Peer0
```
# Create CA dir for CA ORG2
mkdir -p ~/organizations/fabric-ca/org2

# CA Org2 Server
docker run --rm -d \
--name ca_org2 \
-p 7054:7054 \
-e FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server \
-e FABRIC_CA_SERVER_CA_NAME=ca-org2 \
-e FABRIC_CA_SERVER_TLS_ENABLED=true \
-e FABRIC_CA_SERVER_PORT=7054 \
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
fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-org2 --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem
2020/11/12 14:08:08 [INFO] Created a default configuration file at /root/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2020/11/12 14:08:08 [INFO] TLS Enabled
2020/11/12 14:08:08 [INFO] generating key: &{A:ecdsa S:256}
2020/11/12 14:08:08 [INFO] encoded CSR
2020/11/12 14:08:09 [INFO] Stored client certificate at /root/organizations/peerOrganizations/org2.example.com/msp/signcerts/cert.pem
2020/11/12 14:08:09 [INFO] Stored root CA certificate at /root/organizations/peerOrganizations/org2.example.com/msp/cacerts/localhost-7054-ca-org2.pem
2020/11/12 14:08:09 [INFO] Stored Issuer public key at /root/organizations/peerOrganizations/org2.example.com/msp/IssuerPublicKey
2020/11/12 14:08:09 [INFO] Stored Issuer revocation public key at /root/organizations/peerOrganizations/org2.example.com/msp/IssuerRevocationPublicKey

echo 'NodeOUs:
Enable: true
ClientOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-org2.pem
    OrganizationalUnitIdentifier: client
PeerOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-org2.pem
    OrganizationalUnitIdentifier: peer
AdminOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-org2.pem
    OrganizationalUnitIdentifier: admin
OrdererOUIdentifier:
    Certificate: cacerts/localhost-7054-ca-org2.pem
    OrganizationalUnitIdentifier: orderer' >~/organizations/peerOrganizations/org2.example.com/msp/config.yaml

# "Register peer0"
fabric-ca-client register --caname ca-org2 --id.name peer0 --id.secret peer0pw --id.type peer --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

2020/11/12 14:08:36 [INFO] Configuration file location: /root/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2020/11/12 14:08:36 [INFO] TLS Enabled
2020/11/12 14:08:36 [INFO] TLS Enabled
Password: peer0pw

# "Register user"
fabric-ca-client register --caname ca-org2 --id.name user1 --id.secret user1pw --id.type client --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

2020/11/12 14:09:20 [INFO] Configuration file location: /root/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2020/11/12 14:09:20 [INFO] TLS Enabled
2020/11/12 14:09:20 [INFO] TLS Enabled
Password: user1pw

# "Register the org admin"
fabric-ca-client register --caname ca-org2 --id.name org2admin --id.secret org2adminpw --id.type admin --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

2020/11/12 14:10:01 [INFO] Configuration file location: /root/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2020/11/12 14:10:01 [INFO] TLS Enabled
2020/11/12 14:10:01 [INFO] TLS Enabled
Password: org2adminpw

# "Generate the peer0 msp"
fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org2 -M ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp --csr.hosts peer0.org2.example.com --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

2020/11/12 14:11:19 [INFO] TLS Enabled
2020/11/12 14:11:19 [INFO] generating key: &{A:ecdsa S:256}
2020/11/12 14:11:19 [INFO] encoded CSR
2020/11/12 14:11:19 [INFO] Stored client certificate at /root/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/signcerts/cert.pem
2020/11/12 14:11:19 [INFO] Stored root CA certificate at /root/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/cacerts/localhost-7054-ca-org2.pem
2020/11/12 14:11:19 [INFO] Stored Issuer public key at /root/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/IssuerPublicKey
2020/11/12 14:11:19 [INFO] Stored Issuer revocation public key at /root/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/IssuerRevocationPublicKey

cp ~/organizations/peerOrganizations/org2.example.com/msp/config.yaml ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/config.yaml

# "Generate the peer0-tls certificates"
fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org2 -M ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls --enrollment.profile tls --csr.hosts peer0.org2.example.com --csr.hosts localhost --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

2020/11/12 14:12:24 [INFO] TLS Enabled
2020/11/12 14:12:24 [INFO] generating key: &{A:ecdsa S:256}
2020/11/12 14:12:24 [INFO] encoded CSR
2020/11/12 14:12:24 [INFO] Stored client certificate at /root/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/signcerts/cert.pem
2020/11/12 14:12:24 [INFO] Stored TLS root CA certificate at /root/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlscacerts/tls-localhost-7054-ca-org2.pem
2020/11/12 14:12:24 [INFO] Stored Issuer public key at /root/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/IssuerPublicKey
2020/11/12 14:12:24 [INFO] Stored Issuer revocation public key at /root/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/IssuerRevocationPublicKey

cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/signcerts/* ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt
cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/keystore/* ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key
cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org2.example.com/msp/tlscacerts/ca.crt
cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlscacerts/* ~/organizations/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem
cp ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/cacerts/* ~/organizations/peerOrganizations/org2.example.com/ca/ca.org2.example.com-cert.pem

# "Generate the user msp"
fabric-ca-client enroll -u https://user1:user1pw@localhost:7054 --caname ca-org2 -M ~/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

2020/11/12 14:13:35 [INFO] TLS Enabled
2020/11/12 14:13:35 [INFO] generating key: &{A:ecdsa S:256}
2020/11/12 14:13:35 [INFO] encoded CSR
2020/11/12 14:13:36 [INFO] Stored client certificate at /root/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/signcerts/cert.pem
2020/11/12 14:13:36 [INFO] Stored root CA certificate at /root/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/cacerts/localhost-7054-ca-org2.pem
2020/11/12 14:13:36 [INFO] Stored Issuer public key at /root/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/IssuerPublicKey
2020/11/12 14:13:36 [INFO] Stored Issuer revocation public key at /root/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/IssuerRevocationPublicKey

cp ~/organizations/peerOrganizations/org2.example.com/msp/config.yaml ~/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/config.yaml

# "Generate the org admin msp"
fabric-ca-client enroll -u https://org2admin:org2adminpw@localhost:7054 --caname ca-org2 -M ~/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp --tls.certfiles ~/organizations/fabric-ca/org2/tls-cert.pem

2020/11/12 14:16:59 [INFO] TLS Enabled
2020/11/12 14:16:59 [INFO] generating key: &{A:ecdsa S:256}
2020/11/12 14:16:59 [INFO] encoded CSR
2020/11/12 14:16:59 [INFO] Stored client certificate at /root/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/cert.pem
2020/11/12 14:16:59 [INFO] Stored root CA certificate at /root/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/cacerts/localhost-7054-ca-org2.pem
2020/11/12 14:16:59 [INFO] Stored Issuer public key at /root/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/IssuerPublicKey
2020/11/12 14:16:59 [INFO] Stored Issuer revocation public key at /root/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/IssuerRevocationPublicKey

```

>  CA Org2 Server 生成证书文件结构
```
.
└── org2.example.com
    ├── ca
    │   └── ca.org2.example.com-cert.pem
    ├── fabric-ca-client-config.yaml
    ├── msp
    │   ├── IssuerPublicKey
    │   ├── IssuerRevocationPublicKey
    │   ├── cacerts
    │   │   └── localhost-7054-ca-org2.pem
    │   ├── config.yaml
    │   ├── keystore
    │   │   └── aa562e45e9ef2cfcb317477b29dc9bf0ef618c2c0e244c0d4acce496e0e29d97_sk
    │   ├── signcerts
    │   │   └── cert.pem
    │   ├── tlscacerts
    │   │   └── ca.crt
    │   └── user
    ├── peers
    │   └── peer0.org2.example.com
    │       ├── msp
    │       │   ├── IssuerPublicKey
    │       │   ├── IssuerRevocationPublicKey
    │       │   ├── cacerts
    │       │   │   └── localhost-7054-ca-org2.pem
    │       │   ├── config.yaml
    │       │   ├── keystore
    │       │   │   └── e3e549ebce1ffdc993f5bdefe6fbc1754e9b3ed64289ba52590ee1ef820c39e2_sk
    │       │   ├── signcerts
    │       │   │   └── cert.pem
    │       │   └── user
    │       └── tls
    │           ├── IssuerPublicKey
    │           ├── IssuerRevocationPublicKey
    │           ├── ca.crt
    │           ├── cacerts
    │           ├── keystore
    │           │   └── aa9ad653fbca0c0feae0d0ac4b6f5f328ad23a01cf58f4f45ebbf6adebc688b1_sk
    │           ├── server.crt
    │           ├── server.key
    │           ├── signcerts
    │           │   └── cert.pem
    │           ├── tlscacerts
    │           │   └── tls-localhost-7054-ca-org2.pem
    │           └── user
    ├── tlsca
    │   └── tlsca.org2.example.com-cert.pem
    └── users
        ├── Admin@org2.example.com
        │   └── msp
        │       ├── IssuerPublicKey
        │       ├── IssuerRevocationPublicKey
        │       ├── cacerts
        │       │   └── localhost-7054-ca-org2.pem
        │       ├── config.yaml
        │       ├── keystore
        │       │   └── de5c1192ae8442bdd3009807867b228e64e94a319c603ae8409b870e78d07634_sk
        │       ├── signcerts
        │       │   └── cert.pem
        │       └── user
        └── User1@org2.example.com
            └── msp
                ├── IssuerPublicKey
                ├── IssuerRevocationPublicKey
                ├── cacerts
                │   └── localhost-7054-ca-org2.pem
                ├── config.yaml
                ├── keystore
                │   └── f7227ea53d1188e963e9ba9df8a106316072ba0f244f86cd88c65cdce99648df_sk
                ├── signcerts
                │   └── cert.pem
                └── user

35 directories, 36 files
```

## C. 创建 Org1 和 Org2 的CCP json 和 yaml 文件
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