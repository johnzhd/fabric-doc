# Build prerequisites  

1. Set docker proxy  

> If network is not support connect docker servers directly  

* /usr/lib/systemd/system/docker.service  

```service  

# add
# change proxyIP and 1080 into real proxy IP and port

Environment='http_proxy=http://proxyIP:1080'
Environment='https_proxy=https://proxyIP:1080'
Environment="NO_PROXY=localhost,127.0.0.0/8,docker-registry.somecorporation.com"

```

# byfn.sh  

enter script

## define 

1. Environment variables for shell  

|name|type|value|  
|-------:|:-----:|:-------------------------------------|  
|PATH|change|${PWD}/../bin:${PWD}:$PATH|  
|FABRIC_CFG_PATH|new|${PWD}|  
|BLACKLISTED_VERSIONS|new|`^1\.0\. ^1\.1\.0-preview ^1\.1\.0-alpha`|  
|OS_ARCH|new|`$(echo "$(uname -s|tr '[:upper:]' '[:lower:]'|sed 's/mingw64_nt.*/windows/')-$(uname -m | sed 's/x86_64/amd64/g')" | awk '{print tolower($0)}')`|
|CLI_TIMEOUT|operator|10|
|CLI_DELAY|operator|3|
|CHANNEL_NAME|operator|"mychannel"|
|COMPOSE_FILE|operator|docker-compose-cli.yaml|
|COMPOSE_FILE_COUCH|new|docker-compose-couch.yaml|
|LANGUAGE|operator|golang|
|IMAGETAG|operator|"latest"|
|MODE|operator|$1;shift|
|IF_COUCHDB|operator| "couchdb" or "" |

2. Function

|name|type|describe|
|-------:|:-----:|:-------------------------------------|  
|printHelp|UI| print Help |
|askProceed|UI| ask continue |
|clearContainers|docker| clear docker|
|removeUnwantedImages|docker| clear images|
|checkPrereqs|sub work| check version match|
|networkUp|work| build all |
|upgradeNetwork|work| backup & update & testing | 
|networkDown|work| clear all |
|replacePrivateKey|sub work| replace string about CA |
|generateCerts|sub work| create cert files |
|generateChannelArtifacts|sub work| create block/channel/MSP files |

## Step

1. set values
2. set params
3. run work
    * up work
    * down work
    * generate work
    * restart work
    * upgrade work

### up work

1. [checkPrereqs](#checkPrereqs)
2. call `generate` work if files not exist
3. call [scripts/script.sh](#scripts/script.sh) in docker `cli`

### down work

1. shutdown dockers
2. clear backups
3. [clearContainers](#clearContainers)
4. [removeUnwantedImages](#removeUnwantedImages)
5. clear local files


### generate work

1. [generateCerts](#generateCerts)
2. [replacePrivateKey](#replacePrivateKey)
3. [generateChannelArtifacts](#generateChannelArtifacts)

### restart work

1. `down` work
2. `up` work

### upgrade work

1. check volumes
2. backup productions in docker images
3. call [scripts/upgrade_to_v11.sh](#scripts/upgrade_to_v11.sh)

## Function

### printHelp

__skip__

### askProceed

__skip__

### clearContainers

call `docker ps` then call `docker rm`

### removeUnwantedImages

list images then call `docker rmi`

### checkPrereqs

1. get version

```sh
configtxlator version

docker run --rm hyperledger/fabric-tools:$IMAGETAG peer version
```

2. check version by regexp

### networkUp

`up` work

### upgradeNetwork

`upgrade` work

### networkDown

`down` work

### replacePrivateKey

1. cp template config file to config file
2. get keyName by key files
3. replace keyName in config file
4. remove temp file if needed

### generateCerts

1. check tools
2. create key files with config file

### generateChannelArtifacts

1. check tools
2. create block files with config file
3. create channel files with config file
4. update one Org in channel
5. update two Org in channel

# scripts/script.sh

in cli docker  

build first network

## define 

1. variables


|name|type|value|  
|-------:|:-----:|:-------------------------------------|  
|CHANNEL_NAME|inherit|mychannel|
|DELAY|inherit|3|
|LANGUAGE|inherit|golang|
|TIMEOUT|inherit|10|
|COUNTER|new|1|
|MAX_RETRY|new|5|
|ORDERER_CA|new|/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem|
|CC_SRC_PATH|new|github.com/chaincode/chaincode_example02/go/|


2. Function


|name|type|value|  
|-------:|:-----:|:-------------------------------------|  
|createChannel|peer||
|joinChannel|peer||

3. Script

import [scripts/utils.sh](#scripts/utils.sh)

## step

1. set values
2. import [scripts/utils.sh](#scripts/utils.sh)
3. call [createChannel](#createChannel)
4. call [joinChannel](#joinChannel)
5. anchor org1 org2 on channel
    * call [updateAnchorPeers](#updateAnchorPeers)
6. install Chaincode on peer0 on eche org
    * call [installChaincode](#installChaincode)
7. instantiate datas on peer0 org2
    * call [instantiateChaincode](#instantiateChaincode)
8. check peer0 org1 result = 100
    * call [chaincodeQuery](#chaincodeQuery)
9. invoke transaction on peer0 org1
    * call [chaincodeInvoke](#chaincodeInvoke)
10. install Chaincode on peer1 org2. install new node
    * call [installChaincode](#installChaincode)
11. check peer1 org2 (new node) result = 90 (after invoke)
    * call [chaincodeQuery](#chaincodeQuery)


## Function

### createChannel

1. set globals values with peer0 org1
    * call [setGlobals](#setGlobals)

2. peer channel create


### joinChannel

1. loop call for peer be joined
    * call [joinChannelWithRetry](#joinChannelWithRetry)

# scripts/upgrade_to_v11.sh

## define

1. variables

|name|type|value|  
|-------:|:-----:|:-------------------------------------|  
|CHANNEL_NAME|inherit|mychannel|
|DELAY|inherit|3|
|LANGUAGE|inherit|golang|
|TIMEOUT|inherit|10|
|COUNTER|new|1|
|MAX_RETRY|new|5|
|ORDERER_CA|new|/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem|
|CC_SRC_PATH|new|github.com/chaincode/chaincode_example02/go/|

2. Function

|name|type|value|  
|-------:|:-----:|:-------------------------------------|  
|addCapabilityToChannel|||

3. import [scripts/utils.sh](#scripts/utils.sh)

## Step

1. update system
2. install jq
3. config update for orderer testchainid
4. config update for channel testchainid
5. config update for orderer by name
6. config update for application CHANNEL_NAME
7. config update for channel CHANNEL_NAME
8. test query
9. test invoke & query on different nodes


## Function

### addCapabilityToChannel

1. set globals values with channelName & Group
    * call [setOrdererGlobals](#setOrdererGlobals)
2. write config.json by channelName
    * call [fetchChannelConfig](#fetchChannelConfig)
3. change config.json by needed
4. combo config files & write channelName into db files
    * call [createConfigUpdate](#createConfigUpdate)
5. prepare global settings by work type
6. peer channel to update

# scripts/utils.sh

## define

1. Function

|name|type|value|  
|-------:|:-----:|:-------------------------------------|  
|verifyResult|base||
|setOrdererGlobals|base||
|setGlobals|base||
|updateAnchorPeers|peer||
|joinChannelWithRetry|peer||
|installChaincode|peer||
|instantiateChaincode|peer||
|upgradeChaincode|peer||
|chaincodeQuery|peer||
|fetchChannelConfig|peer||
|signConfigtxAsPeerOrg|peer||
|createConfigUpdate|file||
|chaincodeInvoke|peer||

## Function

### verifyResult

1. check last call ret != 0
2. report failed

### setOrdererGlobals

1. set global values for order by static datas

### setGlobals

1. set global values for peer by peer org

### updateAnchorPeers

1. setGlobals
2. peer channel update by anchors.tx

### joinChannelWithRetry

1. setGlobals
2. peer channel join

### installChaincode  

1. setGlobals
2. peer chaincode install

### instantiateChaincode

1. setGlobals
2. peer chaincode instantiate

### upgradeChaincode

1. setGlobals
2. peer chaincode upgrade

### chaincodeQuery

1. setGlobals
2. peer chaincode query

### fetchChannelConfig

1. setOrdererGlobals
2. peer channel fetch config

### signConfigtxAsPeerOrg

1. setGlobals
2. peer channel signconfigtx

### createConfigUpdate

1. configtxlator proto_encode
    1. original -> original_config.pb
    2. modified -> modified_config.pb
2. configtxlator compute_update
    * channel_id, original_config.pb, modified_config.pb -> config_update.pb
3. configtxlator proto_decode
    * config_update.pb -> config_update.json
4. write
    * channel_id, config_update.json -> config_update_in_envelope.json
5. configtxlator proto_encode
    * config_update_in_envelope.json -> `result` files

### chaincodeInvoke

1. setGlobals
2. peer chaincode invoke
