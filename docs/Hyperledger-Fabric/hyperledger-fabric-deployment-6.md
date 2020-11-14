# 基于 Hyperledger Fabric v2.2.1 的多机部署 (六)
> 创建自定义通道，并且将Org1和Org2加入到新建的通道中。


## 1. 在 Orderering Service 这台机器上创建 mychannel
```
# Generating channel-artifacts
mkdir ~/organizations/channel-artifacts

# Generating channel create transaction 'mychannel.tx'
configtxgen -configPath ./configtx.yaml -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/mychannel.tx -channelID mychannel

2020-11-14 09:56:30.359 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-11-14 09:56:30.375 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /root/organizations/configtx.yaml
2020-11-14 09:56:30.375 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 003 Generating new channel configtx
2020-11-14 09:56:30.377 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 004 Writing new channel tx

```

## 2. 在 Orderering Service 这台机器上创建 Org1 和 Org2的 Anchor Peer Transaction
```
# Generating anchor peer update transactions
# Generating anchor peer update transaction for Org1MSP
configtxgen -configPath ./configtx.yaml -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP

2020-11-14 10:00:43.053 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-11-14 10:00:43.069 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /root/organizations/configtx.yaml
2020-11-14 10:00:43.069 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Generating anchor peer update
2020-11-14 10:00:43.070 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 004 Writing anchor peer update

# Generating anchor peer update transaction for Org2MSP
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP

2020-11-14 10:01:54.303 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-11-14 10:01:54.319 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /root/organizations/configtx.yaml
2020-11-14 10:01:54.319 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Generating anchor peer update
2020-11-14 10:01:54.320 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 004 Writing anchor peer update
```

## 3. 创建mychannel
```
export FABRIC_CFG_PATH=${PWD}

# Using Orderer MSP
export CORE_PEER_LOCALMSPID="OrdererMSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export CORE_PEER_MSPCONFIGPATH=${PWD}/ordererOrganizations/example.com/users/Admin@example.com/msp

# Using organization 1

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true


peer channel create -o orderer.example.com:7050 -c mychannel --ordererTLSHostnameOverride orderer.example.com -f ./channel-artifacts/mychannel.tx --outputBlock ./channel-artifacts/mychannel.block --tls --cafile ${PWD}/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2020-11-14 11:13:24.821 CST [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2020-11-14 11:13:24.833 CST [cli.common] readBlock -> INFO 002 Expect block, but got status: &{NOT_FOUND}
2020-11-14 11:13:24.836 CST [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2020-11-14 11:13:25.037 CST [cli.common] readBlock -> INFO 004 Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-11-14 11:13:25.040 CST [channelCmd] InitCmdFactory -> INFO 005 Endorser and orderer connections initialized
2020-11-14 11:13:25.240 CST [cli.common] readBlock -> INFO 006 Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-11-14 11:13:25.243 CST [channelCmd] InitCmdFactory -> INFO 007 Endorser and orderer connections initialized
2020-11-14 11:13:25.444 CST [cli.common] readBlock -> INFO 008 Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-11-14 11:13:25.447 CST [channelCmd] InitCmdFactory -> INFO 009 Endorser and orderer connections initialized
2020-11-14 11:13:25.647 CST [cli.common] readBlock -> INFO 00a Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-11-14 11:13:25.651 CST [channelCmd] InitCmdFactory -> INFO 00b Endorser and orderer connections initialized
2020-11-14 11:13:25.853 CST [cli.common] readBlock -> INFO 00c Received block: 0


peer channel join -b ./channel-artifacts/mychannel.block

2020-11-14 13:29:51.455 CST [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2020-11-14 13:29:51.571 CST [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel

# Using organization 2

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org1.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051


peer channel join -b ./channel-artifacts/mychannel.block

```

## 4. 更新 Anchor Peers

```
# Updating anchor peers for org1...
# Using organization 1
export FABRIC_CFG_PATH=${PWD}
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_PROFILE_ENABLED=true

peer channel update -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile ${PWD}/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2020-11-14 13:53:38.809 CST [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2020-11-14 13:53:38.821 CST [channelCmd] update -> INFO 002 Successfully submitted channel update



# Updating anchor peers for org2...
# Using organization 2

export FABRIC_CFG_PATH=${PWD}
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/peerOrganizations/org1.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051

peer channel update -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile ${PWD}/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2020-11-14 13:58:19.627 CST [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2020-11-14 13:58:19.638 CST [channelCmd] update -> INFO 002 Successfully submitted channel update
```