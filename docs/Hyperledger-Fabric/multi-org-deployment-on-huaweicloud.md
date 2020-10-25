# Huawei Cloud 部署 Hyperledfer Fabric 2.2.1
> 懷疑人生的部署方式-部署還是失敗了不過可以幫助了解下人生的艱難

## 主机配置
```
4vCPUs | 8GB | c3.xlarge.2
Ubuntu 18.04 server 64bit
100 GB Storage
```

## 主机列表
| Nodes  |  Internal IP |  Public IP |
| :----  |  :---------  |  :------   |
|order.example.com | 192.168.0.181 | *** |
|peer0.org1.example.com | 192.168.0.129 | ***|
|peer0.org2.example.com | 192.168.0.70 | ***|

### 华为云 ECS内网域名解析:
1. [内网域名解析](https://support.huaweicloud.com/productdesc-dns/dns_pd_0005.html)
2. [为云服务器配置内网域名](https://support.huaweicloud.com/bestpractice-dns/dns_bestprac_0002.html#dns_bestprac_0002__table11364645122020)

## 一. 环境依赖安装
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

## 二. 搭建基础网络
### 1. 生成证书素材

```
mkdir ~/work/example/organizations/
cd ~/work/example/organizations/
vim crypto-config.yaml 
```
> 注意当以下EnableNodeOUs: true 身份分类启用后，生成的msp中各自的admincerts中将为空，不再默认添加管理员证书（如Admin@example.com-cert.pem）

- crypto-config.yaml
```
OrdererOrgs:
  - Name: OrdererOrg
    Domain: example.com #此处是根域名不是orderer.example.com
    EnableNodeOUs: true #身份分类启用，此项很关键，若不设置peer可能无法连接到orderer
    Specs:
      - Hostname: orderer
 
 
PeerOrgs:
  - Name: Org1MSP
    Domain: org1.example.com
    EnableNodeOUs: true  #身份分类启用，此项很关键，若不设置peer可能无法连接到orderer
    Template:
      Count: 1
    Users:
      Count: 1
 
 
  - Name: Org2MSP
    Domain: org2.example.com
    EnableNodeOUs: true  #身份分类启用，此项很关键，若不设置peer可能无法连接到orderer
    Template:
      Count: 1
    Users:
      Count: 1

```
- 创建证书命令
```
cryptogen generate --config=crypto-config.yaml --output ./
```
### 2. 生成系统的创始区块
```
mkdir ~/work/example/order
cd ~/work/example/order
vim configtx.yaml
```

- configtx.yaml
```
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#
 
---
################################################################################
#
#   Section: Organizations
#
#   - This section defines the different organizational identities which will
#   be referenced later in the configuration.
#
################################################################################
Organizations:
 
    # SampleOrg defines an MSP using the sampleconfig.  It should never be used
    # in production but may be used as a template for other definitions
    - &OrdererOrg
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: OrdererOrg
 
        # ID to load the MSP definition as
        ID: OrdererMSP
 
        # MSPDir is the filesystem path which contains the MSP configuration
        MSPDir: ../organizations/ordererOrganizations/example.com/msp
 
        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"
 
        OrdererEndpoints:
            - "orderer.example.com:7050"
 
    - &Org1
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: Org1MSP
 
        # ID to load the MSP definition as
        ID: Org1MSP
 
        MSPDir: ../organizations/peerOrganizations/org1.example.com/msp
 
        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org1MSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('Org1MSP.peer')"
 
        # leave this flag set to true.
        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: peer0.org1.example.com
              Port: 7051
 
    - &Org2
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: Org2MSP
 
        # ID to load the MSP definition as
        ID: Org2MSP
 
        MSPDir: ../organizations/peerOrganizations/org2.example.com/msp
 
        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.peer', 'Org2MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org2MSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('Org2MSP.peer')"
 
        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: peer0.org2.example.com
              Port: 7051
 
################################################################################
#
#   SECTION: Capabilities
#
#   - This section defines the capabilities of fabric network. This is a new
#   concept as of v1.1.0 and should not be utilized in mixed networks with
#   v1.0.x peers and orderers.  Capabilities define features which must be
#   present in a fabric binary for that binary to safely participate in the
#   fabric network.  For instance, if a new MSP type is added, newer binaries
#   might recognize and validate the signatures from this type, while older
#   binaries without this support would be unable to validate those
#   transactions.  This could lead to different versions of the fabric binaries
#   having different world states.  Instead, defining a capability for a channel
#   informs those binaries without this capability that they must cease
#   processing transactions until they have been upgraded.  For v1.0.x if any
#   capabilities are defined (including a map with all capabilities turned off)
#   then the v1.0.x peer will deliberately crash.
#
################################################################################
Capabilities:
    # Channel capabilities apply to both the orderers and the peers and must be
    # supported by both.
    # Set the value of the capability to true to require it.
    Channel: &ChannelCapabilities
        # V2_0 capability ensures that orderers and peers behave according
        # to v2.0 channel capabilities. Orderers and peers from
        # prior releases would behave in an incompatible way, and are therefore
        # not able to participate in channels at v2.0 capability.
        # Prior to enabling V2.0 channel capabilities, ensure that all
        # orderers and peers on a channel are at v2.0.0 or later.
        V2_0: true
 
    # Orderer capabilities apply only to the orderers, and may be safely
    # used with prior release peers.
    # Set the value of the capability to true to require it.
    Orderer: &OrdererCapabilities
        # V2_0 orderer capability ensures that orderers behave according
        # to v2.0 orderer capabilities. Orderers from
        # prior releases would behave in an incompatible way, and are therefore
        # not able to participate in channels at v2.0 orderer capability.
        # Prior to enabling V2.0 orderer capabilities, ensure that all
        # orderers on channel are at v2.0.0 or later.
        V2_0: true
 
    # Application capabilities apply only to the peer network, and may be safely
    # used with prior release orderers.
    # Set the value of the capability to true to require it.
    Application: &ApplicationCapabilities
        # V2_0 application capability ensures that peers behave according
        # to v2.0 application capabilities. Peers from
        # prior releases would behave in an incompatible way, and are therefore
        # not able to participate in channels at v2.0 application capability.
        # Prior to enabling V2.0 application capabilities, ensure that all
        # peers on channel are at v2.0.0 or later.
        V2_0: true
 
################################################################################
#
#   SECTION: Application
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for application related parameters
#
################################################################################
Application: &ApplicationDefaults
 
    # Organizations is the list of orgs which are defined as participants on
    # the application side of the network
    Organizations:
 
    # Policies defines the set of policies at this level of the config tree
    # For Application policies, their canonical path is
    #   /Channel/Application/<PolicyName>
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        LifecycleEndorsement:
            Type: ImplicitMeta
            Rule: "MAJORITY Endorsement"
        Endorsement:
            Type: ImplicitMeta
            Rule: "MAJORITY Endorsement"
 
    Capabilities:
        <<: *ApplicationCapabilities
################################################################################
#
#   SECTION: Orderer
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for orderer related parameters
#
################################################################################
Orderer: &OrdererDefaults
 
    # Orderer Type: The orderer implementation to start
    OrdererType: etcdraft
 
    EtcdRaft:
        Consenters:
            - Host: orderer.example.com
              Port: 7050
              ClientTLSCert: ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
              ServerTLSCert: ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
 
    # Batch Timeout: The amount of time to wait before creating a batch
    BatchTimeout: 2s
 
    # Batch Size: Controls the number of messages batched into a block
    BatchSize:
 
        # Max Message Count: The maximum number of messages to permit in a batch
        MaxMessageCount: 10
 
        # Absolute Max Bytes: The absolute maximum number of bytes allowed for
        # the serialized messages in a batch.
        AbsoluteMaxBytes: 99 MB
 
        # Preferred Max Bytes: The preferred maximum number of bytes allowed for
        # the serialized messages in a batch. A message larger than the preferred
        # max bytes will result in a batch larger than preferred max bytes.
        PreferredMaxBytes: 512 KB
 
    # Organizations is the list of orgs which are defined as participants on
    # the orderer side of the network
    Organizations:
 
    # Policies defines the set of policies at this level of the config tree
    # For Orderer policies, their canonical path is
    #   /Channel/Orderer/<PolicyName>
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        # BlockValidation specifies what signatures must be included in the block
        # from the orderer for the peer to validate it.
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"
 
################################################################################
#
#   CHANNEL
#
#   This section defines the values to encode into a config transaction or
#   genesis block for channel related parameters.
#
################################################################################
Channel: &ChannelDefaults
    # Policies defines the set of policies at this level of the config tree
    # For Channel policies, their canonical path is
    #   /Channel/<PolicyName>
    Policies:
        # Who may invoke the 'Deliver' API
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        # Who may invoke the 'Broadcast' API
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        # By default, who may modify elements at this config level
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
 
    # Capabilities describes the channel level capabilities, see the
    # dedicated Capabilities section elsewhere in this file for a full
    # description
    Capabilities:
        <<: *ChannelCapabilities
 
################################################################################
#
#   Profile
#
#   - Different configuration profiles may be encoded here to be specified
#   as parameters to the configtxgen tool
#
################################################################################
Profiles:
 
    TwoOrgsOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2
    TwoOrgsChannel:
        Consortium: SampleConsortium
        <<: *ChannelDefaults
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
            Capabilities:
                <<: *ApplicationCapabilities
```
- 生成系统创始区块
```
export FABRIC_CFG_PATH=${PWD}

configtxgen -profile TwoOrgsOrdererGenesis -channelID system-channel -outputBlock ./system-genesis-block/genesis.block
```
### 3. 创建应用通道创建事务
```
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel1.tx -channelID channel1
```
### 4. 启动order
> 在order目录中放入orderer.yaml
```
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#
 
---
################################################################################
#
#   Orderer Configuration
#
#   - This controls the type and configuration of the orderer.
#
################################################################################
General:
    # Listen address: The IP on which to bind to listen.
    ListenAddress: orderer.example.com
 
    # Listen port: The port on which to bind to listen.
    ListenPort: 7050
 
    # TLS: TLS settings for the GRPC server.
    TLS:
        Enabled: true
        # PrivateKey governs the file location of the private key of the TLS certificate.
        PrivateKey: ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
        # Certificate governs the file location of the server TLS certificate.
        Certificate: ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
        RootCAs:
          - ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
        ClientAuthRequired: false #此处暂时为false，正式环境是否需要设置为true，而且下面的值也不确定填写什么（如只写一个组织的那其它组织怎么办）
        ClientRootCAs:
    # Keepalive settings for the GRPC server.
    Keepalive:
        # ServerMinInterval is the minimum permitted time between client pings.
        # If clients send pings more frequently, the server will
        # disconnect them.
        ServerMinInterval: 60s
        # ServerInterval is the time between pings to clients.
        ServerInterval: 7200s
        # ServerTimeout is the duration the server waits for a response from
        # a client before closing the connection.
        ServerTimeout: 20s
    # Cluster settings for ordering service nodes that communicate with other ordering service nodes
    # such as Raft based ordering service.
    Cluster:
        # SendBufferSize is the maximum number of messages in the egress buffer.
        # Consensus messages are dropped if the buffer is full, and transaction
        # messages are waiting for space to be freed.
        SendBufferSize: 10
        # ClientCertificate governs the file location of the client TLS certificate
        # used to establish mutual TLS connections with other ordering service nodes.
        ClientCertificate: ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
        # ClientPrivateKey governs the file location of the private key of the client TLS certificate.
        ClientPrivateKey: ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
        # The below 4 properties should be either set together, or be unset together.
        # If they are set, then the orderer node uses a separate listener for intra-cluster
        # communication. If they are unset, then the general orderer listener is used.
        # This is useful if you want to use a different TLS server certificates on the
        # client-facing and the intra-cluster listeners.
 
        # ListenPort defines the port on which the cluster listens to connections.
        ListenPort:
        # ListenAddress defines the IP on which to listen to intra-cluster communication.
        ListenAddress:
        # ServerCertificate defines the file location of the server TLS certificate used for intra-cluster
        # communication.
        ServerCertificate:
        # ServerPrivateKey defines the file location of the private key of the TLS certificate.
        ServerPrivateKey:
 
    # Bootstrap method: The method by which to obtain the bootstrap block
    # system channel is specified. The option can be one of:
    #   "file" - path to a file containing the genesis block or config block of system channel
    #   "none" - allows an orderer to start without a system channel configuration
    BootstrapMethod: file
 
    # Bootstrap file: The file containing the bootstrap block to use when
    # initializing the orderer system channel and BootstrapMethod is set to
    # "file".  The bootstrap file can be the genesis block, and it can also be
    # a config block for late bootstrap of some consensus methods like Raft.
    # Generate a genesis block by updating $FABRIC_CFG_PATH/configtx.yaml and
    # using configtxgen command with "-outputBlock" option.
    # Defaults to file "genesisblock" (in $FABRIC_CFG_PATH directory) if not specified.
    BootstrapFile: ./system-genesis-block/genesis.block
 
    # LocalMSPDir is where to find the private crypto material needed by the
    # orderer. It is set relative here as a default for dev environments but
    # should be changed to the real location in production.
    LocalMSPDir: ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/
 
    # LocalMSPID is the identity to register the local MSP material with the MSP
    # manager. IMPORTANT: The local MSP ID of an orderer needs to match the MSP
    # ID of one of the organizations defined in the orderer system channel's
    # /Channel/Orderer configuration. The sample organization defined in the
    # sample configuration provided has an MSP ID of "SampleOrg".
    LocalMSPID: OrdererMSP #注意这里写ID不要写NAME很容易错
 
    # Enable an HTTP service for Go "pprof" profiling as documented at:
    # https://golang.org/pkg/net/http/pprof
    Profile:
        Enabled: false
        Address: 0.0.0.0:6060
 
    # BCCSP configures the blockchain crypto service providers.
    BCCSP:
        # Default specifies the preferred blockchain crypto service provider
        # to use. If the preferred provider is not available, the software
        # based provider ("SW") will be used.
        # Valid providers are:
        #  - SW: a software based crypto provider
        #  - PKCS11: a CA hardware security module crypto provider.
        Default: SW
 
        # SW configures the software based blockchain crypto provider.
        SW:
            # TODO: The default Hash and Security level needs refactoring to be
            # fully configurable. Changing these defaults requires coordination
            # SHA2 is hardcoded in several places, not only BCCSP
            Hash: SHA2
            Security: 256
            # Location of key store. If this is unset, a location will be
            # chosen using: 'LocalMSPDir'/keystore
            FileKeyStore:
                KeyStore:
 
        # Settings for the PKCS#11 crypto provider (i.e. when DEFAULT: PKCS11)
        PKCS11:
            # Location of the PKCS11 module library
            Library:
            # Token Label
            Label:
            # User PIN
            Pin:
            Hash:
            Security:
            FileKeyStore:
                KeyStore:
 
    # Authentication contains configuration parameters related to authenticating
    # client messages
    Authentication:
        # the acceptable difference between the current server time and the
        # client's time as specified in a client request message
        TimeWindow: 15m
 
 
################################################################################
#
#   SECTION: File Ledger
#
#   - This section applies to the configuration of the file or json ledgers.
#
################################################################################
FileLedger:
 
    # Location: The directory to store the blocks in.
    # NOTE: If this is unset, a new temporary location will be chosen every time
    # the orderer is restarted, using the prefix specified by Prefix.
    Location: production/orderer/
 
    # The prefix to use when generating a ledger directory in temporary space.
    # Otherwise, this value is ignored.
    Prefix: hyperledger-fabric-ordererledger
 
################################################################################
#
#   SECTION: Kafka
#
#   - This section applies to the configuration of the Kafka-based orderer, and
#     its interaction with the Kafka cluster.
#
################################################################################
Kafka:
 
    # Retry: What do if a connection to the Kafka cluster cannot be established,
    # or if a metadata request to the Kafka cluster needs to be repeated.
    Retry:
        # When a new channel is created, or when an existing channel is reloaded
        # (in case of a just-restarted orderer), the orderer interacts with the
        # Kafka cluster in the following ways:
        # 1. It creates a Kafka producer (writer) for the Kafka partition that
        # corresponds to the channel.
        # 2. It uses that producer to post a no-op CONNECT message to that
        # partition
        # 3. It creates a Kafka consumer (reader) for that partition.
        # If any of these steps fail, they will be re-attempted every
        # <ShortInterval> for a total of <ShortTotal>, and then every
        # <LongInterval> for a total of <LongTotal> until they succeed.
        # Note that the orderer will be unable to write to or read from a
        # channel until all of the steps above have been completed successfully.
        ShortInterval: 5s
        ShortTotal: 10m
        LongInterval: 5m
        LongTotal: 12h
        # Affects the socket timeouts when waiting for an initial connection, a
        # response, or a transmission. See Config.Net for more info:
        # https://godoc.org/github.com/Shopify/sarama#Config
        NetworkTimeouts:
            DialTimeout: 10s
            ReadTimeout: 10s
            WriteTimeout: 10s
        # Affects the metadata requests when the Kafka cluster is in the middle
        # of a leader election.See Config.Metadata for more info:
        # https://godoc.org/github.com/Shopify/sarama#Config
        Metadata:
            RetryBackoff: 250ms
            RetryMax: 3
        # What to do if posting a message to the Kafka cluster fails. See
        # Config.Producer for more info:
        # https://godoc.org/github.com/Shopify/sarama#Config
        Producer:
            RetryBackoff: 100ms
            RetryMax: 3
        # What to do if reading from the Kafka cluster fails. See
        # Config.Consumer for more info:
        # https://godoc.org/github.com/Shopify/sarama#Config
        Consumer:
            RetryBackoff: 2s
    # Settings to use when creating Kafka topics.  Only applies when
    # Kafka.Version is v0.10.1.0 or higher
    Topic:
        # The number of Kafka brokers across which to replicate the topic
        ReplicationFactor: 3
    # Verbose: Enable logging for interactions with the Kafka cluster.
    Verbose: false
 
    # TLS: TLS settings for the orderer's connection to the Kafka cluster.
    TLS:
 
      # Enabled: Use TLS when connecting to the Kafka cluster.
      Enabled: false
 
      # PrivateKey: PEM-encoded private key the orderer will use for
      # authentication.
      PrivateKey:
        # As an alternative to specifying the PrivateKey here, uncomment the
        # following "File" key and specify the file name from which to load the
        # value of PrivateKey.
        #File: path/to/PrivateKey
 
      # Certificate: PEM-encoded signed public key certificate the orderer will
      # use for authentication.
      Certificate:
        # As an alternative to specifying the Certificate here, uncomment the
        # following "File" key and specify the file name from which to load the
        # value of Certificate.
        #File: path/to/Certificate
 
      # RootCAs: PEM-encoded trusted root certificates used to validate
      # certificates from the Kafka cluster.
      RootCAs:
        # As an alternative to specifying the RootCAs here, uncomment the
        # following "File" key and specify the file name from which to load the
        # value of RootCAs.
        #File: path/to/RootCAs
 
    # SASLPlain: Settings for using SASL/PLAIN authentication with Kafka brokers
    SASLPlain:
      # Enabled: Use SASL/PLAIN to authenticate with Kafka brokers
      Enabled: false
      # User: Required when Enabled is set to true
      User:
      # Password: Required when Enabled is set to true
      Password:
 
    # Kafka protocol version used to communicate with the Kafka cluster brokers
    # (defaults to 0.10.2.0 if not specified)
    Version:
 
################################################################################
#
#   Debug Configuration
#
#   - This controls the debugging options for the orderer
#
################################################################################
Debug:
 
    # BroadcastTraceDir when set will cause each request to the Broadcast service
    # for this orderer to be written to a file in this directory
    BroadcastTraceDir:
 
    # DeliverTraceDir when set will cause each request to the Deliver service
    # for this orderer to be written to a file in this directory
    DeliverTraceDir:
 
################################################################################
#
#   Operations Configuration
#
#   - This configures the operations server endpoint for the orderer
#
################################################################################
Operations: #生产环境此处该如何设置
    # host and port for the operations server
    ListenAddress: 127.0.0.1:8443
 
    # TLS configuration for the operations endpoint
    TLS:
        # TLS enabled
        Enabled: false
 
        # Certificate is the location of the PEM encoded TLS certificate
        Certificate:
 
        # PrivateKey points to the location of the PEM-encoded key
        PrivateKey:
 
        # Most operations service endpoints require client authentication when TLS
        # is enabled. ClientAuthRequired requires client certificate authentication
        # at the TLS layer to access all resources.
        ClientAuthRequired: false
 
        # Paths to PEM encoded ca certificates to trust for client authentication
        ClientRootCAs: []
 
################################################################################
#
#   Metrics  Configuration
#
#   - This configures metrics collection for the orderer
#
################################################################################
Metrics:
    # The metrics provider is one of statsd, prometheus, or disabled
    Provider: disabled
 
    # The statsd configuration
    Statsd:
      # network type: tcp or udp
      Network: udp
 
      # the statsd server address
      Address: 127.0.0.1:8125
 
      # The interval at which locally cached counters and gauges are pushed
      # to statsd; timings are pushed immediately
      WriteInterval: 30s
 
      # The prefix is prepended to all emitted statsd metrics
      Prefix:
 
################################################################################
#
#   Consensus Configuration
#
#   - This section contains config options for a consensus plugin. It is opaque
#     to orderer, and completely up to consensus implementation to make use of.
#
################################################################################
Consensus:
    # The allowed key-value pairs here depend on consensus plugin. For etcd/raft,
    # we use following options:
 
    # WALDir specifies the location at which Write Ahead Logs for etcd/raft are
    # stored. Each channel will have its own subdir named after channel ID.
    WALDir: etcdraft/wal
 
    # SnapDir specifies the location at which snapshots for etcd/raft are
    # stored. Each channel will have its own subdir named after channel ID.
    SnapDir: etcdraft/snapshot
```

- 启动orderer
```
orderer start
```
### 5. 启动peer
> 在192.168.0.75 上部署Org1MSP的peer   peer0.org1.example.com
```
cd ~/work/example
mkdir organizations/peerOrganizations
cd organizations/peerOrganizations
scp -r root@orderer.example.com:/root/work/example/organizations/peerOrganizations/org1.example.com org1.example.com

cd ~/work/example/peer
vim core.yaml
```

[Peer0 Org1 的core.yaml文件](./Org1/core.yaml)
### 6. 创建应用通道
> 复制所需通信证书和通道创建事务
```
# 返回peer目录
cd ~/work/example/peer
scp -r root@orderer.example.com:/root/work/example/order/channel-artifacts channel-artifacts

# 自定义orderer的tls ca证书变量
scp -r root@orderer.example.com:/root/work/example/organizations/ordererOrganizations/example.com/msp/tlscacerts/tlsca.example.com-cert.pem tlsca.example.com-cert.pem
export ORDERER_TLSCA=${PWD}/../organizations/order_cert/tlsca.example.com-cert.pem


export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/../organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/../organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051


# 创建通道
peer channel create -o orderer.example.com:7050  -c channel1 -f ./channel-artifacts/channel1.tx --outputBlock ./channel-artifacts/channel1.block --tls --cafile $ORDERER_TLSCA

# 报错则将日志调整为DEBUG级别，查看原因
export FABRIC_LOGGING_SPEC=DEBUG

# 将peer加入到channel中
peer channel join -b ./channel-artifacts/channel1.block
 
# 验证peer是否已加入到通道中，命令会列出区块高度和最新的块的哈希值
peer channel getinfo -c channel1
```
### 7. 将org2的peer加入通道

```
# 复制org2的证书素材
mkdir -p /work/example/organizations/peerOrganizations
cd ~/work/example/organizations/peerOrganizations

scp -r root@orderer.example.com:/root/work/example/organizations/peerOrganizations/org2.example.com org2.example.com

# 复制order的tls证书
cd ~/work/example/organizations/
mkdir orderer.example.com
cd orderer.example.com

scp -r root@orderer.example.com:/root/work/example/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts tlscacerts
```

> 加入配置文件core.yaml
```
cd ~/work/example/peer
vim core.yaml
 
对照org1的core.yaml做以下修改
 
15 peer.id: peer0.org2.example.com
162 peer.gossip.externalEndpoint: peer0.org2.example.com:7051
 
254 peer.tls.cert.file: ../organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt
 
258 peer.tls.key.file: ../organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key
 
261 peer.tls.rootcert.file: 
../organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
 
265 peer.tls.clientRootCAs.files: 
- ../organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
 
314 peer.mspConfigPath: ../organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp
 
323 peer.localMspId: Org2MSP
```
[org2的设置文件](./Org2/core.yaml)

> 设置变量，并启动peer
```
export FABRIC_CFG_PATH=$PWD
启动peer
peer node start >> log_peer.log 2>&1 &
 
！注意，不要写以下环境变量，容易和peer其它操作所需变量混淆，如CORE_PEER_MSPCONFIGPATH应是peer的msp而其它peer命令多数为admin的msp
# export CORE_PEER_TLS_ENABLED=true
# export CORE_PEER_LOCALMSPID="Org2MSP"
# export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/../organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/ca.crt
# export CORE_PEER_MSPCONFIGPATH=${PWD}/../organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
# export CORE_PEER_ADDRESS=peer0.org2.example.com:7051

```
> 系统已经有了通道channel1，Org2从Orderer获取通道的创始区块，命令中0来指定要获取的是创始区块。
```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/../organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/../organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
 
自定义orderer的tls ca证书变量
export ORDERER_TLSCA=${PWD}/../organizations/orderer.example.com/tlscacerts/tlsca.example.com-cert.pem
 
mkdir channel-artifacts
peer channel fetch 0 ./channel-artifacts/channel_org2.block -o orderer.example.com:7050  -c channel1 --tls --cafile $ORDERER_TLSCA

# 使用这个区块将Org2的peer加入通道channel1
peer channel join -b ./channel-artifacts/channel_org2.block

```

### 8.设置锚节点
> 一个组织至少需要一个peer成为锚节点，最好设置多个锚节点以备冗余。组织的锚节点的信息在通道的配置中，组织通过升级通道来指定自己的锚节点。流程和通道更新的步骤相似。
#### 先设置Org1的锚节点

```
# 第一步，设置环境变量，并拉取最新的通道配置区块
cd work/example/peer
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/../organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/../organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export ORDERER_TLSCA=${PWD}/../organizations/order_cert/tlsca.example.com-cert.pem

peer channel fetch config channel-artifacts/config_block.pb -o orderer.example.com:7050  -c channel1 --tls --cafile $ORDERER_TLSCA

# 因为最新的通道配置区块是通道创始区块，所以返回block 0

cd channel-artifacts
# 第二步，使用configtxlator工具来操作通道配置，先将区块从protobuf转译到JSON，去掉不能理解的区块数据，只留下通道配置

configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
jq .data.data[0].payload.data.config config_block.json > config.json

# 第三步，编辑配置文件

cp config.json config_copy.json

# 用jq工具来将Org1锚节点加入到通道配置文件。注意下方.channel_group.groups.Application.groups.Org1MSP.values中Org1MSP是组织的Name不是组织的ID
jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' config_copy.json > modified_config.json

# 第四步，将原来的配置文件和修改过的配置文件转到protobuf格式，并核算它们之间的不同。

configtxlator proto_encode --input config.json --type common.Config --output config.pb
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
configtxlator compute_update --channel_id channel1 --original config.pb --updated modified_config.pb --output config_update.pb

# 第五步，将修改的protobuf格式的配置channel_update.pb封装在事务中，并创建通道升级事务。

configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
echo '{"payload":{"header":{"channel_header":{"channel_id":"channel1", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output config_update_in_envelope.pb

# 第六步，更新通道配置，因为修改部分只影响Org1，另一个通道不需要签名本次升级。
cd ..
peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel1 -o orderer.example.com:7050 --tls --cafile $ORDERER_TLSCA

```

至此，Org1设置锚节点完成。
#### 设置Org2的锚节点
> 下面设置Org2
```


#export FABRIC_CFG_PATH=$PWD，不写此变量

cd work/example/peer
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/../organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/../organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051

export ORDERER_TLSCA=${PWD}/../organizations/orderer.example.com/tlscacerts/tlsca.example.com-cert.pem


cd channel-artifacts
configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
jq .data.data[0].payload.data.config config_block.json > config.json
cp config.json config_copy.json

# 注意此处修改的内容host和port，要改为实际的
jq '.channel_group.groups.Application.groups.Org2MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org2.example.com","port": 7051}]},"version": "0"}}' config_copy.json > modified_config.json

configtxlator proto_encode --input config.json --type common.Config --output config.pb
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
configtxlator compute_update --channel_id channel1 --original config.pb --updated modified_config.pb --output config_update.pb

configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
echo '{"payload":{"header":{"channel_header":{"channel_id":"channel1", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output config_update_in_envelope.pb

cd ..
peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel1 -o orderer.example.com:7050 --tls --cafile $ORDERER_TLSCA

# 确认是否已升级
peer channel getinfo -c channel1

# 返回：
Blockchain info: {"height":3,"currentBlockHash":"/qQQZawOXtAwCtJ+C0+JTHaIzNdYmksrDZVLNbKsdBg=","previousBlockHash":"J9G9J4Zuk0PV4QEuPaNrFCzefqYpLvenorudu2MfrDk="}
```

## 三.部署链码
> 首先将链码放入自建目录中，先从Org2的peer上操作
```
cd ~/work/example/peer
mkdir chaincode
# scp path/xxx.cds root@peer0.org1.example.com:/work/example/peer/chaincode/xxx.cds
# scp path/xxx.cds root@peer0.org2.example.com:/work/example/peer/chaincode/xxx.cds

cp -r /root/fabric/fabric-samples/chaincode/sacc chaincode/sacc
```
### 1. 在Org1和Org2的peer上安装链码 
> 在Org1和Org2的peer中，分别执行以下步骤，将链码安装到各自的peer上。将链码sacc.go放入 ~/work/example/peer/chaincode/sacc中。执行以下代码配置好依赖包
```
cd chaincode/sacc/
go env -w GO111MODULE=on
go mod init sacc
go mod tidy
go mod vendor

# 打包链码
cd ~/work/example/peer
# !注意此时不要设置多余的变量，以免因msp指错而出现权限问题
peer lifecycle chaincode package chaincode/sacc.tar.gz --path chaincode/sacc --lang golang --label sacc_1

```
> 在peer上安装链码 - Org1上安装

```

#export FABRIC_CFG_PATH=$PWD，不写此变量
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/../organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/../organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
 
export ORDERER_TLSCA=${PWD}/../organizations/order_cert/tlsca.example.com-cert.pem

peer lifecycle chaincode install chaincode/sacc.tar.gz
peer lifecycle chaincode queryinstalled 查询确认一下
```
> 在peer上安装链码 - Org2上安装
```
#export FABRIC_CFG_PATH=$PWD，不写此变量
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/../organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/../organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
 
export ORDERER_TLSCA=${PWD}/../organizations/orderer.example.com/tlscacerts/tlsca.example.com-cert.pem
 
peer lifecycle chaincode install chaincode/sacc.tar.gz
peer lifecycle chaincode queryinstalled # 查询确认一下
```

### 2. 签名（审议）链码定义
> Org1和Org2的peer分别执行以下操作

获取Package ID
```
peer lifecycle chaincode queryinstalled
```
> 将Package ID放入环境变量（相同的链码，在不同的peer上，PackageID的值是一样的。但只要链码中有一点变化比如多个空格多个注释，PackageID就会不同，但不同的peer上有不同的PackageID值并不影响链码提交上线）
```
export CC_PACKAGE_ID=sacc_1:b33357c4012471d8bd96ba48fd2a12ada5fedfbfd6d623590295778500a0368d
```
> 

## Reference:
1. [Hyperledger Fabric 多机部署](https://yuanxuxu.com/2019/12/17/hyperledger-fabric-%E5%A4%9A%E6%9C%BA%E9%83%A8%E7%BD%B2/)
2. [在本地 kubernetes 上快速部署 Hyperledger Fabric 网络](https://www.arcblock.io/zh/post/2018/10/29/fabric-kubernetes)
3. [Fabric 区块链的多节点部署](https://developer.ibm.com/zh/articles/cl-lo-hyperledger-fabric-practice-analysis3/)
4. [Hyperledger Fabric 2.2实战记录](https://blog.csdn.net/zhanglingge/article/details/107334524)
5. [Hyperledger-fabric分布式部署](http://qjpcpu.github.io/blog/2018/07/23/hyperledger-fabricfen-bu-shi-bu-shu/)
6. [Hyperledger Fabric自动化部署工具集](https://github.com/zhayujie/fabric-tools)