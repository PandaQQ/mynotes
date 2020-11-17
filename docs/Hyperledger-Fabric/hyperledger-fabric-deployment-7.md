# 基于 Hyperledger Fabric v2.2.1 的多机部署 (七)
> Chaincode 编译和安装到新建的 mychannel 通道 

## 1. 编译 Chanincode 
```
# deploying chaincode on channel 'mychannel'
# Using organization 1

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true

peer lifecycle chaincode package gradingreport.tar.gz --path ./gradingreport-chaincode/ --lang node --label gradingreport_1
```


## 2. 安装 Chanincode 到 Org1 Peer0 和 Org2 Peer0 (各自Peer上安装)
```
# Installing chaincode on peer0.org1...
# Using organization 1

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_TLS_ENABLED=true

peer lifecycle chaincode install gradingreport.tar.gz

2020-11-16 13:51:42.239 CST [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nPgradingreport_1:6a3813d8c2c6764cb889683f7188296a768dee0665fbb37d2ed76d7f61abbced\022\017gradingreport_1" >
2020-11-16 13:51:42.239 CST [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: gradingreport_1:6a3813d8c2c6764cb889683f7188296a768dee0665fbb37d2ed76d7f61abbced


# Install chaincode on peer0.org2...
# Using organization 2
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_TLS_ENABLED=true


peer lifecycle chaincode install gradingreport.tar.gz

2020-11-16 14:02:10.136 CST [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nPgradingreport_1:6a3813d8c2c6764cb889683f7188296a768dee0665fbb37d2ed76d7f61abbced\022\017gradingreport_1" >
2020-11-16 14:02:10.136 CST [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: gradingreport_1:6a3813d8c2c6764cb889683f7188296a768dee0665fbb37d2ed76d7f61abbced

``` 

## 3. 初始化 Chaincode 到 Org1 Peer0 和 Org2 Peer0 (回到 Orderer 服务器上安装)
```
# Using organization 1
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_TLS_ENABLED=true

peer lifecycle chaincode queryinstalled

Installed chaincodes on peer:
Package ID: gradingreport_1:6a3813d8c2c6764cb889683f7188296a768dee0665fbb37d2ed76d7f61abbced, Label: gradingreport_1

# Query installed successful on peer0.org1 on channel
# Using organization 1

peer lifecycle chaincode approveformyorg -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name gradingreport --version 1 --package-id gradingreport_1:6a3813d8c2c6764cb889683f7188296a768dee0665fbb37d2ed76d7f61abbced --sequence 1 --init-required

2020-11-16 14:20:46.680 CST [chaincodeCmd] ClientWait -> INFO 001 txid [50def59f9ad8e58cd02186f91f729b1dd895f17d6c5cd627943cc3251f1240c2] committed with status (VALID) at peer0.org1.example.com:7051

# Using organization 1
# Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...

peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name gradingreport --version 1 --sequence 1 --init-required --output json

{
	"approvals": {
		"Org1MSP": true,
		"Org2MSP": false
	}
}

# Using organization 2
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_TLS_ENABLED=true

peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name gradingreport --version 1 --sequence 1 --init-required --output json

{
	"approvals": {
		"Org1MSP": true,
		"Org2MSP": false
	}
}

peer lifecycle chaincode approveformyorg -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name gradingreport --version 1 --package-id gradingreport_1:6a3813d8c2c6764cb889683f7188296a768dee0665fbb37d2ed76d7f61abbced --sequence 1 --init-required

2020-11-16 15:34:51.797 CST [chaincodeCmd] ClientWait -> INFO 001 txid [699725defa3bad92e9a5d67131e56667a2703d9263ae0be2c36fc10c70682466] committed with status (VALID) at peer0.org2.example.com:7051

peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name gradingreport --version 1 --sequence 1 --init-required --output json

{
	"approvals": {
		"Org1MSP": true,
		"Org2MSP": true
	}
}

# Using organization 1
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_TLS_ENABLED=true

# Using organization 2
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_TLS_ENABLED=true

peer lifecycle chaincode commit -o orderer.example.com:7050 \
                                --ordererTLSHostnameOverride orderer.example.com \
                                --tls --cafile ${PWD}/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
                                --channelID mychannel --name gradingreport \
                                --peerAddresses peer0.org1.example.com:7051 \
                                --tlsRootCertFiles ${PWD}/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
                                --peerAddresses peer0.org2.example.com:7051 \
                                --tlsRootCertFiles ${PWD}/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
                                --version 1 --sequence 1 --init-required

2020-11-16 15:54:23.955 CST [chaincodeCmd] ClientWait -> INFO 001 txid [46bd7e03976eef7ef53663e7864073dcac71e5e2b5e72d49ff1e6ab31eab044b] committed with status (VALID) at peer0.org2.example.com:7051
2020-11-16 15:54:23.955 CST [chaincodeCmd] ClientWait -> INFO 002 txid [46bd7e03976eef7ef53663e7864073dcac71e5e2b5e72d49ff1e6ab31eab044b] committed with status (VALID) at peer0.org1.example.com:7051


# Using organization 1
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_TLS_ENABLED=true

# Querying chaincode definition on peer0.org1 on channel 'mychannel'...
peer lifecycle chaincode querycommitted --channelID mychannel --name gradingreport

Committed chaincode definition for chaincode 'gradingreport' on channel 'mychannel':
Version: 1, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]

# Using organization 2
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_TLS_ENABLED=true

peer lifecycle chaincode querycommitted --channelID mychannel --name gradingreport

Committed chaincode definition for chaincode 'gradingreport' on channel 'mychannel':
Version: 1, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org2 on channel 'mychannel'

```

## 4.调用 ChainCode  

```
# Using organization 1
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_TLS_ENABLED=true

# Using organization 2
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true
export CORE_PEER_TLS_ENABLED=true

peer chaincode invoke -o orderer.example.com:7050 \
                      --ordererTLSHostnameOverride orderer.example.com \
                      --tls --cafile ${PWD}/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
                      -C mychannel \
                      -n gradingreport \
                      --peerAddresses peer0.org1.example.com:7051 \
                      --tlsRootCertFiles ${PWD}/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
                      --peerAddresses peer0.org2.example.com:7051 \
                      --tlsRootCertFiles ${PWD}/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \ 
                      --isInit -c '{"function":"<your function>","Args":[]}'

```

## 结束语
通过本系列文章相信大家已经掌握如何将Hyperledger Fabric 2.2.1部署在多机器的环境下。