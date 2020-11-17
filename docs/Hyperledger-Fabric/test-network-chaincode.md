deploying chaincode on channel 'mychannel'
executing with the following
- CHANNEL_NAME: mychannel
- CC_NAME: fabcar
- CC_SRC_PATH: ../chaincode/fabcar/go/
- CC_SRC_LANGUAGE: go
- CC_VERSION: 1
- CC_SEQUENCE: 1
- CC_END_POLICY: NA
- CC_COLL_CONFIG: NA
- CC_INIT_FCN: initLedger
- DELAY: 3
- MAX_RETRY: 5
- VERBOSE: false
Vendoring Go dependencies at ../chaincode/fabcar/go/
~/fabric/fabric-samples/chaincode/fabcar/go ~/fabric/fabric-samples/test-network
go: downloading github.com/hyperledger/fabric-contract-api-go v1.1.0
go: extracting github.com/hyperledger/fabric-contract-api-go v1.1.0
go: downloading github.com/hyperledger/fabric-protos-go v0.0.0-20200424173316-dd554ba3746e
go: downloading github.com/hyperledger/fabric-chaincode-go v0.0.0-20200424173110-d7076418f212
go: downloading github.com/gobuffalo/packr v1.30.1
go: downloading github.com/go-openapi/spec v0.19.4
go: downloading github.com/xeipuuv/gojsonschema v1.2.0
go: extracting github.com/go-openapi/spec v0.19.4
go: extracting github.com/hyperledger/fabric-chaincode-go v0.0.0-20200424173110-d7076418f212
go: downloading github.com/golang/protobuf v1.3.2
go: downloading google.golang.org/grpc v1.23.0
go: extracting github.com/gobuffalo/packr v1.30.1
go: downloading github.com/gobuffalo/packd v0.3.0
go: downloading github.com/go-openapi/jsonreference v0.19.2
go: extracting github.com/xeipuuv/gojsonschema v1.2.0
go: extracting github.com/hyperledger/fabric-protos-go v0.0.0-20200424173316-dd554ba3746e
go: downloading github.com/go-openapi/swag v0.19.5
go: extracting github.com/go-openapi/swag v0.19.5
go: downloading github.com/gobuffalo/envy v1.7.0
go: downloading github.com/xeipuuv/gojsonreference v0.0.0-20180127040603-bd5ef7bd5415
go: downloading github.com/mailru/easyjson v0.0.0-20190626092158-b2ccc519800e
go: extracting github.com/golang/protobuf v1.3.2
go: extracting github.com/xeipuuv/gojsonreference v0.0.0-20180127040603-bd5ef7bd5415
go: extracting github.com/gobuffalo/envy v1.7.0
go: extracting github.com/gobuffalo/packd v0.3.0
go: downloading gopkg.in/yaml.v2 v2.2.8
go: extracting github.com/go-openapi/jsonreference v0.19.2
go: downloading github.com/go-openapi/jsonpointer v0.19.3
go: extracting github.com/go-openapi/jsonpointer v0.19.3
go: downloading github.com/xeipuuv/gojsonpointer v0.0.0-20180127040702-4e3ac2762d5f
go: downloading github.com/joho/godotenv v1.3.0
go: extracting github.com/xeipuuv/gojsonpointer v0.0.0-20180127040702-4e3ac2762d5f
go: downloading github.com/rogpeppe/go-internal v1.3.0
go: extracting github.com/joho/godotenv v1.3.0
go: extracting gopkg.in/yaml.v2 v2.2.8
go: downloading github.com/PuerkitoBio/purell v1.1.1
go: extracting github.com/mailru/easyjson v0.0.0-20190626092158-b2ccc519800e
go: extracting github.com/PuerkitoBio/purell v1.1.1
go: downloading golang.org/x/net v0.0.0-20190827160401-ba9fcec4b297
go: downloading golang.org/x/text v0.3.2
go: downloading github.com/PuerkitoBio/urlesc v0.0.0-20170810143723-de5bf2ad4578
go: extracting github.com/rogpeppe/go-internal v1.3.0
go: extracting github.com/PuerkitoBio/urlesc v0.0.0-20170810143723-de5bf2ad4578
go: extracting google.golang.org/grpc v1.23.0
go: downloading google.golang.org/genproto v0.0.0-20180831171423-11092d34479b
go: downloading golang.org/x/sys v0.0.0-20190710143415-6ec70d6a5542
go: extracting golang.org/x/net v0.0.0-20190827160401-ba9fcec4b297
go: extracting golang.org/x/sys v0.0.0-20190710143415-6ec70d6a5542
go: extracting google.golang.org/genproto v0.0.0-20180831171423-11092d34479b
go: extracting golang.org/x/text v0.3.2
~/fabric/fabric-samples/test-network
Finished vendoring Go dependencies
Using organization 1
+ peer lifecycle chaincode package fabcar.tar.gz --path ../chaincode/fabcar/go/ --lang golang --label fabcar_1
+ res=0
Chaincode is packaged on peer0.org1
Installing chaincode on peer0.org1...
Using organization 1
+ peer lifecycle chaincode install fabcar.tar.gz
+ res=0
2020-11-16 10:24:57.498 CST [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nIfabcar_1:762e0fe3dbeee0f7b08fb6200adeb4a3a20f649a00f168c0b3c2257e53b6e506\022\010fabcar_1" >
2020-11-16 10:24:57.498 CST [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: fabcar_1:762e0fe3dbeee0f7b08fb6200adeb4a3a20f649a00f168c0b3c2257e53b6e506
Chaincode is installed on peer0.org1
Install chaincode on peer0.org2...
Using organization 2
+ peer lifecycle chaincode install fabcar.tar.gz
+ res=0
2020-11-16 10:25:05.104 CST [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nIfabcar_1:762e0fe3dbeee0f7b08fb6200adeb4a3a20f649a00f168c0b3c2257e53b6e506\022\010fabcar_1" >
2020-11-16 10:25:05.104 CST [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: fabcar_1:762e0fe3dbeee0f7b08fb6200adeb4a3a20f649a00f168c0b3c2257e53b6e506
Chaincode is installed on peer0.org2
Using organization 1
+ peer lifecycle chaincode queryinstalled
+ res=0
Installed chaincodes on peer:
Package ID: fabcar_1:762e0fe3dbeee0f7b08fb6200adeb4a3a20f649a00f168c0b3c2257e53b6e506, Label: fabcar_1
Query installed successful on peer0.org1 on channel
Using organization 1
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name fabcar --version 1 --package-id fabcar_1:762e0fe3dbeee0f7b08fb6200adeb4a3a20f649a00f168c0b3c2257e53b6e506 --sequence 1 --init-required
+ res=0
2020-11-16 10:25:07.321 CST [chaincodeCmd] ClientWait -> INFO 001 txid [3ef3868772118813c276652ca50b80d6e7bad2113c16fa1c5e28370d77f1077c] committed with status (VALID) at localhost:7051
Chaincode definition approved on peer0.org1 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name fabcar --version 1 --sequence 1 --init-required --output json
+ res=0
{
	"approvals": {
		"Org1MSP": true,
		"Org2MSP": false
	}
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name fabcar --version 1 --sequence 1 --init-required --output json
+ res=0
{
	"approvals": {
		"Org1MSP": true,
		"Org2MSP": false
	}
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 2
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name fabcar --version 1 --package-id fabcar_1:762e0fe3dbeee0f7b08fb6200adeb4a3a20f649a00f168c0b3c2257e53b6e506 --sequence 1 --init-required
+ res=0
2020-11-16 10:25:15.619 CST [chaincodeCmd] ClientWait -> INFO 001 txid [32c02d39597660f29174173afc515744211d0ea58e99335344aca7d22c3e6cac] committed with status (VALID) at localhost:9051
Chaincode definition approved on peer0.org2 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name fabcar --version 1 --sequence 1 --init-required --output json
+ res=0
{
	"approvals": {
		"Org1MSP": true,
		"Org2MSP": true
	}
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name fabcar --version 1 --sequence 1 --init-required --output json
+ res=0
{
	"approvals": {
		"Org1MSP": true,
		"Org2MSP": true
	}
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 1
Using organization 2
+ peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name fabcar --peerAddresses localhost:7051 --tlsRootCertFiles /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt --version 1 --sequence 1 --init-required
+ res=0
2020-11-16 10:25:23.953 CST [chaincodeCmd] ClientWait -> INFO 001 txid [62b31575f04d8f7c1416102dbbace178635565603fa5d303c2045c48fcd5f2e5] committed with status (VALID) at localhost:7051
2020-11-16 10:25:23.962 CST [chaincodeCmd] ClientWait -> INFO 002 txid [62b31575f04d8f7c1416102dbbace178635565603fa5d303c2045c48fcd5f2e5] committed with status (VALID) at localhost:9051
Chaincode definition committed on channel 'mychannel'
Using organization 1
Querying chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to Query committed status on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name fabcar
+ res=0
Committed chaincode definition for chaincode 'fabcar' on channel 'mychannel':
Version: 1, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Querying chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to Query committed status on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name fabcar
+ res=0
Committed chaincode definition for chaincode 'fabcar' on channel 'mychannel':
Version: 1, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 1
Using organization 2
+ fcn_call='{"function":"initLedger","Args":[]}'
+ infoln 'invoke fcn call:{"function":"initLedger","Args":[]}'
+ println '\033[0;34minvoke fcn call:{"function":"initLedger","Args":[]}\033[0m'
+ echo -e '\033[0;34minvoke fcn call:{"function":"initLedger","Args":[]}\033[0m'
invoke fcn call:{"function":"initLedger","Args":[]}
+ peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /root/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n fabcar --peerAddresses localhost:7051 --tlsRootCertFiles /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles /root/fabric/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt --isInit -c '{"function":"initLedger","Args":[]}'
+ res=0
2020-11-16 10:25:30.177 CST [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
Invoke transaction successful on peer0.org1 peer0.org2 on channel 'mychannel'
~/fabric/fabric-samples/fabcar

Total setup execution time : 93 secs ...

Next, use the FabCar applications to interact with the deployed FabCar contract.
The FabCar applications are available in multiple programming languages.
Follow the instructions for the programming language of your choice:

JavaScript:

  Start by changing into the "javascript" directory:
    cd javascript

  Next, install all required packages:
    npm install

  Then run the following applications to enroll the admin user, and register a new user
  called appUser which will be used by the other applications to interact with the deployed
  FabCar contract:
    node enrollAdmin
    node registerUser

  You can run the invoke application as follows. By default, the invoke application will
  create a new car, but you can update the application to submit other transactions:
    node invoke

  You can run the query application as follows. By default, the query application will
  return all cars, but you can update the application to evaluate other transactions:
    node query

TypeScript:

  Start by changing into the "typescript" directory:
    cd typescript

  Next, install all required packages:
    npm install

  Next, compile the TypeScript code into JavaScript:
    npm run build

  Then run the following applications to enroll the admin user, and register a new user
  called appUser which will be used by the other applications to interact with the deployed
  FabCar contract:
    node dist/enrollAdmin
    node dist/registerUser

  You can run the invoke application as follows. By default, the invoke application will
  create a new car, but you can update the application to submit other transactions:
    node dist/invoke

  You can run the query application as follows. By default, the query application will
  return all cars, but you can update the application to evaluate other transactions:
    node dist/query

Java:

  Start by changing into the "java" directory:
    cd java

  Then, install dependencies and run the test using:
    mvn test

  The test will invoke the sample client app which perform the following:
    - Enroll admin and appUser and import them into the wallet (if they don't already exist there)
    - Submit a transaction to create a new car
    - Evaluate a transaction (query) to return details of this car
    - Submit a transaction to change the owner of this car
    - Evaluate a transaction (query) to return the updated details of this car

Go:

  Start by changing into the "go" directory:
    cd go

  Then, install dependencies and run the test using:
    go run fabcar.go

  The test will invoke the sample client app which perform the following:
    - Import user credentials into the wallet (if they don't already exist there)
    - Submit a transaction to create a new car
    - Evaluate a transaction (query) to return details of this car
    - Submit a transaction to change the owner of this car
    - Evaluate a transaction (query) to return the updated details of this car