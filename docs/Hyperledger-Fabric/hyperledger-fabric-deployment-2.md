# 基于 Hyperledger Fabric v2.2.1 的多机部署 (二)
> 部署 Orderer 和 Peer 服务


## 使用Docker 创建 Orderer 服务

```
# Create Docker Orderer Volume Dir 
mkdir -p ./orderer.example.com

docker run --rm -d \
--name orderer.example.com \
-e FABRIC_LOGGING_SPEC=INFO \
-e ORDERER_GENERAL_LISTENADDRESS=0.0.0.0 \
-e ORDERER_GENERAL_LISTENPORT=7050 \
-e ORDERER_GENERAL_GENESISMETHOD=file \
-e ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block \
-e ORDERER_GENERAL_LOCALMSPID=OrdererMSP \
-e ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp \
-e ORDERER_GENERAL_TLS_ENABLED=true \
-e ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key \
-e ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt \
-e ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt] \
-e ORDERER_KAFKA_TOPIC_REPLICATIONFACTOR=1 \
-e ORDERER_KAFKA_VERBOSE=true \
-e ORDERER_GENERAL_CLUSTER_CLIENTCERTIFICATE=/var/hyperledger/orderer/tls/server.crt \
-e ORDERER_GENERAL_CLUSTER_CLIENTPRIVATEKEY=/var/hyperledger/orderer/tls/server.key \
-e ORDERER_GENERAL_CLUSTER_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt] \
-w /opt/gopath/src/github.com/hyperledger/fabric \
-p 7050:7050 \
-v ~/system-genesis-block/genesis.block:/var/hyperledger/orderer/orderer.genesis.block \
-v ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp \
-v ~/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/:/var/hyperledger/orderer/tls \
-v ~/orderer.example.com:/var/hyperledger/production/orderer \
hyperledger/fabric-orderer:2.2.1 orderer
```


## 使用Docker 创建 Org1 服务

### 1. Docker 安装 Org1 CouchDB
```
docker run --rm -d \
--name couchdb_peer0_org1 \
-e COUCHDB_USER=admin \
-e COUCHDB_PASSWORD=adminpw \
-p 5984:5984 \
couchdb:3.1
```

### 2. Docker 安装 Org1 Peer

```
# Create Docker Org1 Volume Dir 
mkdir -p ./peer0.org1.example.com

# Docker Run Org1 Peer
docker run --rm -d \
--name peer0.org1.example.com \
-e CORE_LEDGER_STATE_STATEDATABASE=CouchDB
-e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb_peer0_org1:5984 \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=adminpw \
-e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock \
-e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_test \
-e FABRIC_LOGGING_SPEC=INFO \
-e CORE_PEER_TLS_ENABLED=true \
-e CORE_PEER_PROFILE_ENABLED=true \
-e CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt \
-e CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key \
-e CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt \
-e CORE_PEER_ID=peer0.org1.example.com \
-e CORE_PEER_ADDRESS=peer0.org1.example.com:7051 \
-e CORE_PEER_LISTENADDRESS=0.0.0.0:7051 \
-e CORE_PEER_CHAINCODEADDRESS=peer0.org1.example.com:7052 \
-e CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052 \
-e CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051 \
-e CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051 \
-e CORE_PEER_LOCALMSPID=Org1MSP \
-w /opt/gopath/src/github.com/hyperledger/fabric/peer \
-v /var/run/:/host/var/run/ \
-v ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/fabric/msp \
-v ~/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls:/etc/hyperledger/fabric/tls \
-v ~/peer0.org1.example.com:/var/hyperledger/production \
-p 7051:7051 \
hyperledger/fabric-peer:2.2.1 peer node start
```


## 使用Docker 创建 Org2 服务
### 1. Docker 安装 Org2 CouchDB
```
docker run --rm -d \
--name couchdb_peer0_org2 \
-e COUCHDB_USER=admin \
-e COUCHDB_PASSWORD=adminpw \
-p 5984:5984 \
couchdb:3.1
```

### 2. Docker 安装 Org2 Peer

```
# Create Docker Org2 Volume Dir 
mkdir -p ./peer0.org2.example.com

# Docker Run Org1 Peer
docker run --rm -d \
--name peer0.org2.example.com \
-e CORE_LEDGER_STATE_STATEDATABASE=CouchDB \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb_peer0_org2:5984 \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=adminpw \
-e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock \
-e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_test \
-e FABRIC_LOGGING_SPEC=INFO \
-e CORE_PEER_TLS_ENABLED=true \
-e CORE_PEER_PROFILE_ENABLED=true \
-e CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt \
-e CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key \
-e CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt \
-e CORE_PEER_ID=peer0.org2.example.com \
-e CORE_PEER_ADDRESS=peer0.org2.example.com:7051 \
-e CORE_PEER_LISTENADDRESS=0.0.0.0:7051 \
-e CORE_PEER_CHAINCODEADDRESS=peer0.org2.example.com:7052 \
-e CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052 \
-e CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org2.example.com:7051 \
-e CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org2.example.com:7051 \
-e CORE_PEER_LOCALMSPID=Org2MSP \
-v /var/run/:/host/var/run/ \
-v ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp:/etc/hyperledger/fabric/msp \
-v ~/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls:/etc/hyperledger/fabric/tls \
-v ~/peer0.org2.example.com:/var/hyperledger/production \
-w /opt/gopath/src/github.com/hyperledger/fabric/peer \
-p 7051:7051 \
hyperledger/fabric-peer:2.2.1
```