root@ecs-d413:~/fabric/fabric-samples/test-network# ./network.sh up createChannel -ca -s couchdb
Creating channel 'mychannel'.
If network is not up, starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'couchdb with crypto from 'Certificate Authorities'
Bringing up network
LOCAL_VERSION=2.2.1
DOCKER_IMAGE_VERSION=2.2.1
CA_LOCAL_VERSION=1.4.9
CA_DOCKER_IMAGE_VERSION=1.4.9
Generate certificates using Fabric CA's
Creating network "net_test" with the default driver
Creating ca_orderer ... done
Creating ca_org1    ... done
Creating ca_org2    ... done
Create Org1 Identities
Enroll the CA admin
+ fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-org1 --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org1/tls-cert.pem
2020/11/11 10:12:08 [INFO] Created a default configuration file at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:08 [INFO] TLS Enabled
2020/11/11 10:12:08 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:08 [INFO] encoded CSR
2020/11/11 10:12:08 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/signcerts/cert.pem
2020/11/11 10:12:08 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2020/11/11 10:12:08 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/IssuerPublicKey
2020/11/11 10:12:08 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/IssuerRevocationPublicKey
Register peer0
+ fabric-ca-client register --caname ca-org1 --id.name peer0 --id.secret peer0pw --id.type peer --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org1/tls-cert.pem
2020/11/11 10:12:08 [INFO] Configuration file location: /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:08 [INFO] TLS Enabled
2020/11/11 10:12:08 [INFO] TLS Enabled
Password: peer0pw
Register user
+ fabric-ca-client register --caname ca-org1 --id.name user1 --id.secret user1pw --id.type client --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org1/tls-cert.pem
2020/11/11 10:12:08 [INFO] Configuration file location: /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:08 [INFO] TLS Enabled
2020/11/11 10:12:08 [INFO] TLS Enabled
Password: user1pw
Register the org admin
+ fabric-ca-client register --caname ca-org1 --id.name org1admin --id.secret org1adminpw --id.type admin --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org1/tls-cert.pem
2020/11/11 10:12:08 [INFO] Configuration file location: /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:08 [INFO] TLS Enabled
2020/11/11 10:12:08 [INFO] TLS Enabled
Password: org1adminpw
Generate the peer0 msp
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org1 -M /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp --csr.hosts peer0.org1.example.com --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org1/tls-cert.pem
2020/11/11 10:12:09 [INFO] TLS Enabled
2020/11/11 10:12:09 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:09 [INFO] encoded CSR
2020/11/11 10:12:09 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/signcerts/cert.pem
2020/11/11 10:12:09 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2020/11/11 10:12:09 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/IssuerPublicKey
2020/11/11 10:12:09 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/IssuerRevocationPublicKey
Generate the peer0-tls certificates
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org1 -M /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls --enrollment.profile tls --csr.hosts peer0.org1.example.com --csr.hosts localhost --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org1/tls-cert.pem
2020/11/11 10:12:09 [INFO] TLS Enabled
2020/11/11 10:12:09 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:09 [INFO] encoded CSR
2020/11/11 10:12:09 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/signcerts/cert.pem
2020/11/11 10:12:09 [INFO] Stored TLS root CA certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlscacerts/tls-localhost-7054-ca-org1.pem
2020/11/11 10:12:09 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/IssuerPublicKey
2020/11/11 10:12:09 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/IssuerRevocationPublicKey
Generate the user msp
+ fabric-ca-client enroll -u https://user1:user1pw@localhost:7054 --caname ca-org1 -M /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org1/tls-cert.pem
2020/11/11 10:12:09 [INFO] TLS Enabled
2020/11/11 10:12:09 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:09 [INFO] encoded CSR
2020/11/11 10:12:09 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/signcerts/cert.pem
2020/11/11 10:12:09 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2020/11/11 10:12:09 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/IssuerPublicKey
2020/11/11 10:12:09 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/IssuerRevocationPublicKey
Generate the org admin msp
+ fabric-ca-client enroll -u https://org1admin:org1adminpw@localhost:7054 --caname ca-org1 -M /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org1/tls-cert.pem
2020/11/11 10:12:09 [INFO] TLS Enabled
2020/11/11 10:12:09 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:09 [INFO] encoded CSR
2020/11/11 10:12:09 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/cert.pem
2020/11/11 10:12:09 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2020/11/11 10:12:09 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/IssuerPublicKey
2020/11/11 10:12:09 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/IssuerRevocationPublicKey
Create Org2 Identities
Enroll the CA admin
+ fabric-ca-client enroll -u https://admin:adminpw@localhost:8054 --caname ca-org2 --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org2/tls-cert.pem
2020/11/11 10:12:09 [INFO] Created a default configuration file at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:09 [INFO] TLS Enabled
2020/11/11 10:12:09 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:09 [INFO] encoded CSR
2020/11/11 10:12:09 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/signcerts/cert.pem
2020/11/11 10:12:09 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2020/11/11 10:12:09 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/IssuerPublicKey
2020/11/11 10:12:09 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/IssuerRevocationPublicKey
Register peer0
+ fabric-ca-client register --caname ca-org2 --id.name peer0 --id.secret peer0pw --id.type peer --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org2/tls-cert.pem
2020/11/11 10:12:09 [INFO] Configuration file location: /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:09 [INFO] TLS Enabled
2020/11/11 10:12:09 [INFO] TLS Enabled
Password: peer0pw
Register user
+ fabric-ca-client register --caname ca-org2 --id.name user1 --id.secret user1pw --id.type client --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org2/tls-cert.pem
2020/11/11 10:12:09 [INFO] Configuration file location: /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:09 [INFO] TLS Enabled
2020/11/11 10:12:09 [INFO] TLS Enabled
Password: user1pw
Register the org admin
+ fabric-ca-client register --caname ca-org2 --id.name org2admin --id.secret org2adminpw --id.type admin --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org2/tls-cert.pem
2020/11/11 10:12:09 [INFO] Configuration file location: /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:09 [INFO] TLS Enabled
2020/11/11 10:12:09 [INFO] TLS Enabled
Password: org2adminpw
Generate the peer0 msp
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:8054 --caname ca-org2 -M /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp --csr.hosts peer0.org2.example.com --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org2/tls-cert.pem
2020/11/11 10:12:09 [INFO] TLS Enabled
2020/11/11 10:12:09 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:10 [INFO] encoded CSR
2020/11/11 10:12:10 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/signcerts/cert.pem
2020/11/11 10:12:10 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2020/11/11 10:12:10 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/IssuerPublicKey
2020/11/11 10:12:10 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/IssuerRevocationPublicKey
Generate the peer0-tls certificates
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:8054 --caname ca-org2 -M /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls --enrollment.profile tls --csr.hosts peer0.org2.example.com --csr.hosts localhost --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org2/tls-cert.pem
2020/11/11 10:12:10 [INFO] TLS Enabled
2020/11/11 10:12:10 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:10 [INFO] encoded CSR
2020/11/11 10:12:10 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/signcerts/cert.pem
2020/11/11 10:12:10 [INFO] Stored TLS root CA certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlscacerts/tls-localhost-8054-ca-org2.pem
2020/11/11 10:12:10 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/IssuerPublicKey
2020/11/11 10:12:10 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/IssuerRevocationPublicKey
Generate the user msp
+ fabric-ca-client enroll -u https://user1:user1pw@localhost:8054 --caname ca-org2 -M /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org2/tls-cert.pem
2020/11/11 10:12:10 [INFO] TLS Enabled
2020/11/11 10:12:10 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:10 [INFO] encoded CSR
2020/11/11 10:12:10 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/signcerts/cert.pem
2020/11/11 10:12:10 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2020/11/11 10:12:10 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/IssuerPublicKey
2020/11/11 10:12:10 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/IssuerRevocationPublicKey
Generate the org admin msp
+ fabric-ca-client enroll -u https://org2admin:org2adminpw@localhost:8054 --caname ca-org2 -M /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/org2/tls-cert.pem
2020/11/11 10:12:10 [INFO] TLS Enabled
2020/11/11 10:12:10 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:10 [INFO] encoded CSR
2020/11/11 10:12:10 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/cert.pem
2020/11/11 10:12:10 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2020/11/11 10:12:10 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/IssuerPublicKey
2020/11/11 10:12:10 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/IssuerRevocationPublicKey
Create Orderer Org Identities
Enroll the CA admin
+ fabric-ca-client enroll -u https://admin:adminpw@localhost:9054 --caname ca-orderer --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/tls-cert.pem
2020/11/11 10:12:10 [INFO] Created a default configuration file at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:10 [INFO] TLS Enabled
2020/11/11 10:12:10 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:10 [INFO] encoded CSR
2020/11/11 10:12:10 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/signcerts/cert.pem
2020/11/11 10:12:10 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/cacerts/localhost-9054-ca-orderer.pem
2020/11/11 10:12:10 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/IssuerPublicKey
2020/11/11 10:12:10 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/IssuerRevocationPublicKey
Register orderer
+ fabric-ca-client register --caname ca-orderer --id.name orderer --id.secret ordererpw --id.type orderer --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/tls-cert.pem
2020/11/11 10:12:10 [INFO] Configuration file location: /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:10 [INFO] TLS Enabled
2020/11/11 10:12:10 [INFO] TLS Enabled
Password: ordererpw
Register the orderer admin
+ fabric-ca-client register --caname ca-orderer --id.name ordererAdmin --id.secret ordererAdminpw --id.type admin --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/tls-cert.pem
2020/11/11 10:12:10 [INFO] Configuration file location: /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2020/11/11 10:12:10 [INFO] TLS Enabled
2020/11/11 10:12:10 [INFO] TLS Enabled
Password: ordererAdminpw
Generate the orderer msp
+ fabric-ca-client enroll -u https://orderer:ordererpw@localhost:9054 --caname ca-orderer -M /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp --csr.hosts orderer.example.com --csr.hosts localhost --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/tls-cert.pem
2020/11/11 10:12:10 [INFO] TLS Enabled
2020/11/11 10:12:10 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:10 [INFO] encoded CSR
2020/11/11 10:12:10 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/signcerts/cert.pem
2020/11/11 10:12:10 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/localhost-9054-ca-orderer.pem
2020/11/11 10:12:10 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/IssuerPublicKey
2020/11/11 10:12:10 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/IssuerRevocationPublicKey
Generate the orderer-tls certificates
+ fabric-ca-client enroll -u https://orderer:ordererpw@localhost:9054 --caname ca-orderer -M /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls --enrollment.profile tls --csr.hosts orderer.example.com --csr.hosts localhost --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/tls-cert.pem
2020/11/11 10:12:10 [INFO] TLS Enabled
2020/11/11 10:12:10 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:10 [INFO] encoded CSR
2020/11/11 10:12:11 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/signcerts/cert.pem
2020/11/11 10:12:11 [INFO] Stored TLS root CA certificate at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/tlscacerts/tls-localhost-9054-ca-orderer.pem
2020/11/11 10:12:11 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/IssuerPublicKey
2020/11/11 10:12:11 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/IssuerRevocationPublicKey
Generate the admin msp
+ fabric-ca-client enroll -u https://ordererAdmin:ordererAdminpw@localhost:9054 --caname ca-orderer -M /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp --tls.certfiles /root/fabric/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/tls-cert.pem
2020/11/11 10:12:11 [INFO] TLS Enabled
2020/11/11 10:12:11 [INFO] generating key: &{A:ecdsa S:256}
2020/11/11 10:12:11 [INFO] encoded CSR
2020/11/11 10:12:11 [INFO] Stored client certificate at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/cert.pem
2020/11/11 10:12:11 [INFO] Stored root CA certificate at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/cacerts/localhost-9054-ca-orderer.pem
2020/11/11 10:12:11 [INFO] Stored Issuer public key at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/IssuerPublicKey
2020/11/11 10:12:11 [INFO] Stored Issuer revocation public key at /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/IssuerRevocationPublicKey
Generate CCP files for Org1 and Org2
/root/fabric/fabric-samples/test-network/../bin/configtxgen
Generating Orderer Genesis block
+ configtxgen -profile TwoOrgsOrdererGenesis -channelID system-channel -outputBlock ./system-genesis-block/genesis.block
2020-11-11 10:12:11.241 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-11-11 10:12:11.259 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002 orderer type: etcdraft
2020-11-11 10:12:11.259 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 Orderer.EtcdRaft.Options unset, setting to tick_interval:"500ms" election_tick:10 heartbeat_tick:1 max_inflight_blocks:5 snapshot_interval_size:16777216
2020-11-11 10:12:11.259 CST [common.tools.configtxgen.localconfig] Load -> INFO 004 Loaded configuration: /root/fabric/fabric-samples/test-network/configtx/configtx.yaml
2020-11-11 10:12:11.260 CST [common.tools.configtxgen] doOutputBlock -> INFO 005 Generating genesis block
2020-11-11 10:12:11.260 CST [common.tools.configtxgen] doOutputBlock -> INFO 006 Writing genesis block
+ res=0
Creating volume "net_orderer.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_peer0.org2.example.com" with default driver
WARNING: Found orphan containers (ca_org1, ca_org2, ca_orderer) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
Pulling couchdb1 (couchdb:3.1.1)...
3.1.1: Pulling from library/couchdb
bb79b6b2107f: Pull complete
64047dc3beae: Pull complete
1bf5219be029: Pull complete
fe133310c07e: Pull complete
605d5fefdc3d: Pull complete
38f02fa87302: Pull complete
f0d636474460: Pull complete
5bf601c5e86b: Pull complete
7540b73bba44: Pull complete
cb1d0befd4e1: Pull complete
9d632d0fb7a8: Pull complete
Digest: sha256:b5e6939af792a0e40218e405e8656d68c14a03220899f470b58611a00dc65a9a
Status: Downloaded newer image for couchdb:3.1.1
Creating couchdb0               ... done
Creating couchdb1            ... done
Creating orderer.example.com    ... done
Creating peer0.org2.example.com ... done
Creating peer0.org1.example.com ... done
CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS                  PORTS                                        NAMES
6c32023f5e0e        hyperledger/fabric-peer:latest      "peer node start"        1 second ago        Up Less than a second   0.0.0.0:7051->7051/tcp                       peer0.org1.example.com
feb6ebc411dd        hyperledger/fabric-peer:latest      "peer node start"        1 second ago        Up Less than a second   7051/tcp, 0.0.0.0:9051->9051/tcp             peer0.org2.example.com
bc3ea660a70a        couchdb:3.1.1                       "tini -- /docker-ent…"   2 seconds ago       Up Less than a second   4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp   couchdb1
c1ba79897122        hyperledger/fabric-orderer:latest   "orderer"                2 seconds ago       Up Less than a second   0.0.0.0:7050->7050/tcp                       orderer.example.com
9e979d26cbaa        couchdb:3.1.1                       "tini -- /docker-ent…"   2 seconds ago       Up Less than a second   4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp   couchdb0
0e99c67c3330        hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   16 seconds ago      Up 14 seconds           0.0.0.0:7054->7054/tcp                       ca_org1
96ca258f845b        hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   16 seconds ago      Up 14 seconds           7054/tcp, 0.0.0.0:8054->8054/tcp             ca_org2
24111893e038        hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   16 seconds ago      Up 14 seconds           7054/tcp, 0.0.0.0:9054->9054/tcp             ca_orderer
Generating channel create transaction 'mychannel.tx'
+ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/mychannel.tx -channelID mychannel
2020-11-11 10:12:22.111 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-11-11 10:12:22.128 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /root/fabric/fabric-samples/test-network/configtx/configtx.yaml
2020-11-11 10:12:22.128 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 003 Generating new channel configtx
2020-11-11 10:12:22.130 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 004 Writing new channel tx
+ res=0
Generating anchor peer update transactions
Generating anchor peer update transaction for Org1MSP
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
2020-11-11 10:12:22.151 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-11-11 10:12:22.169 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /root/fabric/fabric-samples/test-network/configtx/configtx.yaml
2020-11-11 10:12:22.169 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Generating anchor peer update
2020-11-11 10:12:22.170 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 004 Writing anchor peer update
+ res=0
Generating anchor peer update transaction for Org2MSP
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP
2020-11-11 10:12:22.190 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-11-11 10:12:22.209 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /root/fabric/fabric-samples/test-network/configtx/configtx.yaml
2020-11-11 10:12:22.209 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Generating anchor peer update
2020-11-11 10:12:22.210 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 004 Writing anchor peer update
+ res=0
Creating channel mychannel
Using organization 1
+ peer channel create -o localhost:7050 -c mychannel --ordererTLSHostnameOverride orderer.example.com -f ./channel-artifacts/mychannel.tx --outputBlock ./channel-artifacts/mychannel.block --tls --cafile /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
+ res=0
2020-11-11 10:12:25.248 CST [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2020-11-11 10:12:25.260 CST [cli.common] readBlock -> INFO 002 Expect block, but got status: &{NOT_FOUND}
2020-11-11 10:12:25.262 CST [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2020-11-11 10:12:25.463 CST [cli.common] readBlock -> INFO 004 Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-11-11 10:12:25.464 CST [channelCmd] InitCmdFactory -> INFO 005 Endorser and orderer connections initialized
2020-11-11 10:12:25.665 CST [cli.common] readBlock -> INFO 006 Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-11-11 10:12:25.667 CST [channelCmd] InitCmdFactory -> INFO 007 Endorser and orderer connections initialized
2020-11-11 10:12:25.868 CST [cli.common] readBlock -> INFO 008 Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-11-11 10:12:25.870 CST [channelCmd] InitCmdFactory -> INFO 009 Endorser and orderer connections initialized
2020-11-11 10:12:26.071 CST [cli.common] readBlock -> INFO 00a Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-11-11 10:12:26.073 CST [channelCmd] InitCmdFactory -> INFO 00b Endorser and orderer connections initialized
2020-11-11 10:12:26.274 CST [cli.common] readBlock -> INFO 00c Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-11-11 10:12:26.276 CST [channelCmd] InitCmdFactory -> INFO 00d Endorser and orderer connections initialized
2020-11-11 10:12:26.478 CST [cli.common] readBlock -> INFO 00e Received block: 0
Channel 'mychannel' created
Join Org1 peers to the channel...
Using organization 1
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2020-11-11 10:12:29.518 CST [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2020-11-11 10:12:29.639 CST [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
Join Org2 peers to the channel...
Using organization 2
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2020-11-11 10:12:32.679 CST [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2020-11-11 10:12:32.807 CST [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
Updating anchor peers for org1...
Using organization 1
+ peer channel update -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
+ res=0
2020-11-11 10:12:35.844 CST [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2020-11-11 10:12:35.853 CST [channelCmd] update -> INFO 002 Successfully submitted channel update
Anchor peers updated for org 'Org1MSP' on channel 'mychannel'
Updating anchor peers for org2...
Using organization 2
+ peer channel update -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
+ res=0
2020-11-11 10:12:41.894 CST [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2020-11-11 10:12:41.903 CST [channelCmd] update -> INFO 002 Successfully submitted channel update
Anchor peers updated for org 'Org2MSP' on channel 'mychannel'
Channel successfully joined