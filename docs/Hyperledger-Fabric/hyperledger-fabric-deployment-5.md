# 基于 Hyperledger Fabric v2.2.1 的多机部署 (五)
> 部署 CouchDB 和 Peer 在 Org1 和 Org2 服务器上。

## 使用Docker 创建 Org1 服务

| Nodes  |  Internal IP |  Public IP |
| :----  |  :---------  |  :------   |
|peer0.org1.example.com | 192.168.0.129 | 119.8.56.178 |

### 1. Docker 安装 Org1 CouchDB
```
# Create couchdb data path
mkdir -p ~/couchdb-data

# Run CouchDB Container
docker run --rm -d \
--name couchdb_peer0_org1 \
-e COUCHDB_USER=admin \
-e COUCHDB_PASSWORD=adminpw \
-p 5984:5984 \
-v /home/couchdb-data:/opt/couchdb/data \
couchdb:3.1
```

### 2. Docker 安装 Org1 Peer

```
# Create Docker Org1 Volume Dir 
mkdir -p ~/peer0.org1.example.com

# Docker Run Org1 Peer
docker run --rm -d \
--name peer0.org1.example.com \
-e CORE_LEDGER_STATE_STATEDATABASE=CouchDB \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=peer0.org1.example.com:5984 \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=adminpw \
-e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock \
-e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=artifacts_default \
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


# Deployment success logs
2020-11-13 06:51:43.556 UTC [nodeCmd] serve -> INFO 001 Starting peer:
 Version: 2.2.1
 Commit SHA: 344fda6
 Go version: go1.14.4
 OS/Arch: linux/amd64
 Chaincode:
  Base Docker Label: org.hyperledger.fabric
  Docker Namespace: hyperledger
2020-11-13 06:51:43.556 UTC [peer] getLocalAddress -> INFO 002 Auto-detected peer address: 172.17.0.4:7051
2020-11-13 06:51:43.557 UTC [peer] getLocalAddress -> INFO 003 Returning peer0.org1.example.com:7051
2020-11-13 06:51:43.560 UTC [nodeCmd] initGrpcSemaphores -> INFO 004 concurrency limit for endorser service is 2500
2020-11-13 06:51:43.560 UTC [nodeCmd] initGrpcSemaphores -> INFO 005 concurrency limit for deliver service is 2500
2020-11-13 06:51:43.560 UTC [nodeCmd] serve -> INFO 006 Starting peer with TLS enabled
2020-11-13 06:51:43.571 UTC [certmonitor] trackCertExpiration -> INFO 007 The enrollment certificate will expire on 2021-11-11 09:35:00 +0000 UTC
2020-11-13 06:51:43.571 UTC [certmonitor] trackCertExpiration -> INFO 008 The server TLS certificate will expire on 2021-11-11 09:37:00 +0000 UTC
2020-11-13 06:51:43.573 UTC [ledgermgmt] NewLedgerMgr -> INFO 009 Initializing LedgerMgr
2020-11-13 06:51:43.594 UTC [leveldbhelper] openDBAndCheckFormat -> INFO 00a DB is empty Setting db format as 2.0
2020-11-13 06:51:43.596 UTC [blkstorage] NewProvider -> INFO 00b Creating new file ledger directory at /var/hyperledger/production/ledgersData/chains/chains
2020-11-13 06:51:43.608 UTC [leveldbhelper] openDBAndCheckFormat -> INFO 00c DB is empty Setting db format as 2.0
2020-11-13 06:51:43.650 UTC [couchdb] createDatabaseIfNotExist -> INFO 00d Created state database _users
2020-11-13 06:51:43.667 UTC [couchdb] createDatabaseIfNotExist -> INFO 00e Created state database _replicator
2020-11-13 06:51:43.697 UTC [couchdb] createDatabaseIfNotExist -> INFO 00f Created state database fabric__internal
2020-11-13 06:51:43.722 UTC [ledgermgmt] NewLedgerMgr -> INFO 010 Initialized LedgerMgr
2020-11-13 06:51:43.728 UTC [gossip.service] New -> INFO 011 Initialize gossip with endpoint peer0.org1.example.com:7051
2020-11-13 06:51:43.729 UTC [gossip.gossip] New -> INFO 012 Creating gossip service with self membership of Endpoint: peer0.org1.example.com:7051, InternalEndpoint: peer0.org1.example.com:7051, PKI-ID: f984f7efe616b4a24a4cf8442152a92619a51f316a88c8fbac312a950187304d, Metadata:
2020-11-13 06:51:43.730 UTC [lifecycle] InitializeLocalChaincodes -> INFO 013 Initialized lifecycle cache with 0 already installed chaincodes
2020-11-13 06:51:43.730 UTC [gossip.gossip] start -> INFO 014 Gossip instance peer0.org1.example.com:7051 started
2020-11-13 06:51:43.730 UTC [nodeCmd] computeChaincodeEndpoint -> INFO 015 Entering computeChaincodeEndpoint with peerHostname: peer0.org1.example.com
2020-11-13 06:51:43.730 UTC [nodeCmd] computeChaincodeEndpoint -> INFO 016 Exit with ccEndpoint: peer0.org1.example.com:7052
2020-11-13 06:51:43.734 UTC [sccapi] DeploySysCC -> INFO 017 deploying system chaincode 'lscc'
2020-11-13 06:51:43.734 UTC [sccapi] DeploySysCC -> INFO 018 deploying system chaincode 'cscc'
2020-11-13 06:51:43.734 UTC [sccapi] DeploySysCC -> INFO 019 deploying system chaincode 'qscc'
2020-11-13 06:51:43.734 UTC [sccapi] DeploySysCC -> INFO 01a deploying system chaincode '_lifecycle'
2020-11-13 06:51:43.734 UTC [nodeCmd] serve -> INFO 01b Deployed system chaincodes
2020-11-13 06:51:43.734 UTC [discovery] NewService -> INFO 01c Created with config TLS: true, authCacheMaxSize: 1000, authCachePurgeRatio: 0.750000
2020-11-13 06:51:43.734 UTC [nodeCmd] registerDiscoveryService -> INFO 01d Discovery service activated
2020-11-13 06:51:43.734 UTC [nodeCmd] serve -> INFO 01e Starting peer with ID=[peer0.org1.example.com], network ID=[dev], address=[peer0.org1.example.com:7051]
2020-11-13 06:51:43.734 UTC [nodeCmd] func6 -> INFO 01f Starting profiling server with listenAddress = 0.0.0.0:6060
2020-11-13 06:51:43.734 UTC [nodeCmd] serve -> INFO 020 Started peer with ID=[peer0.org1.example.com], network ID=[dev], address=[peer0.org1.example.com:7051]
2020-11-13 06:51:43.734 UTC [kvledger] LoadPreResetHeight -> INFO 021 Loading prereset height from path [/var/hyperledger/production/ledgersData/chains]
2020-11-13 06:51:43.734 UTC [blkstorage] preResetHtFiles -> INFO 022 No active channels passed
```

## 使用Docker 创建 Org2 服务

| Nodes  |  Internal IP |  Public IP |
| :----  |  :---------  |  :------   |
|peer0.org2.example.com | 192.168.0.70 |  119.8.123.249 |


### 1. Docker 安装 Org2 CouchDB
```
# Create couchdb data path
mkdir -p ~/couchdb-data

# Run CouchDB Container
docker run --rm -d \
--name couchdb_peer0_org2 \
-e COUCHDB_USER=admin \
-e COUCHDB_PASSWORD=adminpw \
-p 5984:5984 \
-v /home/couchdb-data:/opt/couchdb/data \
couchdb:3.1
```

### 2. Docker 安装 Org2 Peer
```
# Create Docker Org2 Volume Dir 
mkdir -p ~/peer0.org2.example.com

# Docker Run Org2 Peer
docker run --rm -d \
--name peer0.org2.example.com \
-e CORE_LEDGER_STATE_STATEDATABASE=CouchDB \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=peer0.org2.example.com:5984 \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=adminpw \
-e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock \
-e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=artifacts_default \
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
hyperledger/fabric-peer:2.2.1 peer node start


## Deployment Success Logs
2020-11-13 07:01:42.925 UTC [nodeCmd] serve -> INFO 001 Starting peer:
 Version: 2.2.1
 Commit SHA: 344fda6
 Go version: go1.14.4
 OS/Arch: linux/amd64
 Chaincode:
  Base Docker Label: org.hyperledger.fabric
  Docker Namespace: hyperledger
2020-11-13 07:01:42.926 UTC [peer] getLocalAddress -> INFO 002 Auto-detected peer address: 172.17.0.4:7051
2020-11-13 07:01:42.926 UTC [peer] getLocalAddress -> INFO 003 Returning peer0.org2.example.com:7051
2020-11-13 07:01:42.930 UTC [nodeCmd] initGrpcSemaphores -> INFO 004 concurrency limit for endorser service is 2500
2020-11-13 07:01:42.930 UTC [nodeCmd] initGrpcSemaphores -> INFO 005 concurrency limit for deliver service is 2500
2020-11-13 07:01:42.930 UTC [nodeCmd] serve -> INFO 006 Starting peer with TLS enabled
2020-11-13 07:01:42.937 UTC [certmonitor] trackCertExpiration -> INFO 007 The enrollment certificate will expire on 2021-11-12 06:11:00 +0000 UTC
2020-11-13 07:01:42.937 UTC [certmonitor] trackCertExpiration -> INFO 008 The server TLS certificate will expire on 2021-11-12 06:12:00 +0000 UTC
2020-11-13 07:01:42.939 UTC [ledgermgmt] NewLedgerMgr -> INFO 009 Initializing LedgerMgr
2020-11-13 07:01:42.956 UTC [leveldbhelper] openDBAndCheckFormat -> INFO 00a DB is empty Setting db format as 2.0
2020-11-13 07:01:42.958 UTC [blkstorage] NewProvider -> INFO 00b Creating new file ledger directory at /var/hyperledger/production/ledgersData/chains/chains
2020-11-13 07:01:42.969 UTC [leveldbhelper] openDBAndCheckFormat -> INFO 00c DB is empty Setting db format as 2.0
2020-11-13 07:01:43.042 UTC [couchdb] createDatabaseIfNotExist -> INFO 00d Created state database _users
2020-11-13 07:01:43.059 UTC [couchdb] createDatabaseIfNotExist -> INFO 00e Created state database _replicator
2020-11-13 07:01:43.105 UTC [couchdb] createDatabaseIfNotExist -> INFO 00f Created state database fabric__internal
2020-11-13 07:01:43.128 UTC [ledgermgmt] NewLedgerMgr -> INFO 010 Initialized LedgerMgr
2020-11-13 07:01:43.135 UTC [gossip.service] New -> INFO 011 Initialize gossip with endpoint peer0.org2.example.com:7051
2020-11-13 07:01:43.135 UTC [gossip.gossip] New -> INFO 012 Creating gossip service with self membership of Endpoint: peer0.org2.example.com:7051, InternalEndpoint: peer0.org2.example.com:7051, PKI-ID: f967d856996c783e826e587c632585b495a2033fa40d7bcca549d744c141ab54, Metadata:
2020-11-13 07:01:43.136 UTC [lifecycle] InitializeLocalChaincodes -> INFO 013 Initialized lifecycle cache with 0 already installed chaincodes
2020-11-13 07:01:43.136 UTC [gossip.gossip] start -> INFO 014 Gossip instance peer0.org2.example.com:7051 started
2020-11-13 07:01:43.136 UTC [nodeCmd] computeChaincodeEndpoint -> INFO 015 Entering computeChaincodeEndpoint with peerHostname: peer0.org2.example.com
2020-11-13 07:01:43.136 UTC [nodeCmd] computeChaincodeEndpoint -> INFO 016 Exit with ccEndpoint: peer0.org2.example.com:7052
2020-11-13 07:01:43.140 UTC [sccapi] DeploySysCC -> INFO 017 deploying system chaincode 'lscc'
2020-11-13 07:01:43.140 UTC [sccapi] DeploySysCC -> INFO 018 deploying system chaincode 'cscc'
2020-11-13 07:01:43.140 UTC [sccapi] DeploySysCC -> INFO 019 deploying system chaincode 'qscc'
2020-11-13 07:01:43.140 UTC [sccapi] DeploySysCC -> INFO 01a deploying system chaincode '_lifecycle'
2020-11-13 07:01:43.140 UTC [nodeCmd] serve -> INFO 01b Deployed system chaincodes
2020-11-13 07:01:43.140 UTC [discovery] NewService -> INFO 01c Created with config TLS: true, authCacheMaxSize: 1000, authCachePurgeRatio: 0.750000
2020-11-13 07:01:43.140 UTC [nodeCmd] registerDiscoveryService -> INFO 01d Discovery service activated
2020-11-13 07:01:43.140 UTC [nodeCmd] serve -> INFO 01e Starting peer with ID=[peer0.org2.example.com], network ID=[dev], address=[peer0.org2.example.com:7051]
2020-11-13 07:01:43.140 UTC [nodeCmd] func6 -> INFO 01f Starting profiling server with listenAddress = 0.0.0.0:6060
2020-11-13 07:01:43.140 UTC [nodeCmd] serve -> INFO 020 Started peer with ID=[peer0.org2.example.com], network ID=[dev], address=[peer0.org2.example.com:7051]
2020-11-13 07:01:43.141 UTC [kvledger] LoadPreResetHeight -> INFO 021 Loading prereset height from path [/var/hyperledger/production/ledgersData/chains]
2020-11-13 07:01:43.141 UTC [blkstorage] preResetHtFiles -> INFO 022 No active channels passed 

```