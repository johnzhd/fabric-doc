# docker

|name|command|ports|images|
|:--------|:------------|:--------------------------------|:--------------------------------|
|peer0.org1.example.com|"peer node start"|0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp|hyperledger/fabric-peer:latest|
|peer1.org1.example.com|"peer node start"|0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp|hyperledger/fabric-peer:latest|
|peer0.org2.example.com|"peer node start"|0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp|hyperledger/fabric-peer:latest|
|peer1.org2.example.com|"peer node start"|0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp|hyperledger/fabric-peer:latest|
|orderer.example.com|"orderer"|0.0.0.0:7050->7050/tcp|hyperledger/fabric-orderer:latest|
|cli|"/bin/bash"||hyperledger/fabric-tools:latest|
|dev-peer0.org1.example.com-mycc-1.0|"chaincode -peer.add…"||dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302...|


# files


## ./channel-artifacts/

create by generate

权限控制文档？

1. block

genesis.block

2. channel

channel.tx

3. MSP anchor

Org1MSPanchors.tx
Org2MSPanchors.tx


## ./crypto-config/

create by generate

x509 certs & signing keys

* order Organizations

**/ca/
**/msp/
**/tlsca/
**/users/
**/orderers/


* peer Organizations

**/ca/
**/msp/
**/tlsca/
**/users/
**/peers/

## ./crypto-config.yaml

create by generate

record network topology

配置文件

1. name
2. template
3. users
4. domain


# logic

-- network ---

--- domain ---

-- MSP --

--- Genesis Block | Channel | Anchor Peer ---

Genesis Block: configuration blockchain network or channel, as first block on a chain.

