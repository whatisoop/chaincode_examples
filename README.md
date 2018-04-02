# chaincode_examples
1、证书配置、channel配置、docker配置见tools文件夹
cryptogen generate --config crypto-config.yaml --output=crypto-config
mkdir channel-artifacts
export CHANNEL_NAME=onechannel
export FABRIC_CFG_PATH=$PWD     #(设置一个环境变量来告诉configtxgen哪里去寻找configtx.yaml)
configtxgen -outputBlock ./channel-artifacts/orderer.block -profile TwoOrgsOrdererGenesis 
configtxgen -outputCreateChannelTx ./channel-artifacts/onechannel.tx -profile TwoOrgsChannel -channelID onechannel
2、进入tools容器，执行chaincode的测试
export CHANNEL_NAME=onechannel
peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./onechannel.tx 
peer channel join -b onechannel.block

############################################# chaincode_example01
# install chaincode
peer chaincode install -n chaincode_example01 -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/go/chaincode_example01
# init chaincode
peer chaincode instantiate -o orderer.example.com:7050 -C onechannel -n chaincode_example01 -v 1.0 -c '{"Args":["init","a","100","b","200"]}'
# invoke chaincode
peer chaincode invoke -C onechannel -n chaincode_example01 -v 1.0 -c '{"Args":["invoke","20"]}'

# upgrade chaincode
peer chaincode install -n chaincode_example01 -v 2.0 -p github.com/hyperledger/fabric/peer/chaincode/go/chaincode_example01
peer chaincode upgrade -C onechannel -n chaincode_example01 -v 2.0 -c '{"Args":["init","a","1000","b","2000"]}'


############################################# chaincode_example02
# install chaincode
peer chaincode install -n chaincode_example02 -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/go/chaincode_example02
# init chaincode
peer chaincode instantiate -o orderer.example.com:7050 -C onechannel -n chaincode_example02 -v 1.0 -c '{"Args":["init","a","100","b","200"]}'
# invoke chaincode
peer chaincode invoke -C onechannel -n chaincode_example02 -v 1.0 -c '{"Args":["invoke","a","b","20"]}'
# query chaincode
peer chaincode query -C onechannel -n chaincode_example02 -v 1.0 -c '{"Args":["query","a"]}'
# stub.DelState
peer chaincode invoke -C onechannel -n chaincode_example02 -v 1.0 -c '{"Args":["delete","a"]}'


############################################# chaincode_example03 ？？？？
# install chaincode
peer chaincode install -n chaincode_example03 -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/go/chaincode_example03
# init chaincode
peer chaincode instantiate -o orderer.example.com:7050 -C onechannel -n chaincode_example03 -v 1.0 -c '{"Args":["init","a","100"]}'
# invoke chaincode
peer chaincode invoke -C onechannel -n chaincode_example03 -v 1.0 -c '{"Args":["query","a","20"]}'


############################################# chaincode_example04
# install chaincode
peer chaincode install -n chaincode_example04 -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/go/chaincode_example04
# init chaincode
peer chaincode instantiate -o orderer.example.com:7050 -C onechannel -n chaincode_example04 -v 1.0 -c '{"Args":["init","a","100"]}'
# stub.InvokeChaincode   如果两个chaincode位于同一个channel，那么最后一个参数可为空，即不需要指明channel名称
peer chaincode invoke -C onechannel -n chaincode_example04 -v 1.0 -c '{"Args":["invoke","chaincode_example02","a","1","onechannel"]}'
# query chaincode
peer chaincode query -C onechannel -n chaincode_example04 -v 1.0 -c '{"Args":["query","a"]}'
# query chaincode
peer chaincode query -C onechannel -n chaincode_example04 -v 1.0 -c '{"Args":["query","a","chaincode_example02","a","onechannel"]}'


############################################# chaincode_example05  同一个channel内chaincode互调/跨channel的chaincode调用
# install chaincode
peer chaincode install -n chaincode_example05 -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/go/chaincode_example05
# init chaincode
peer chaincode instantiate -o orderer.example.com:7050 -C onechannel -n chaincode_example05 -v 1.0 -c '{"Args":["init","a","100"]}'
# stub.InvokeChaincode   如果两个chaincode位于同一个channel，那么最后一个参数可为空，即不需要指明channel名称
peer chaincode invoke -C onechannel -n chaincode_example05 -v 1.0 -c '{"Args":["invoke","chaincode_example02","sum","onechannel"]}'
# query chaincode
peer chaincode query -C onechannel -n chaincode_example05 -v 1.0 -c '{"Args":["query","chaincode_example02","sum",""]}'

# 跨channel调用chaincode，例子：在onechannel的chaincode_example05里面调用anotherchannel的chaincode_example02
# 生成第二个channel，anotherchannel
export FABRIC_CFG_PATH=$PWD
configtxgen -outputCreateChannelTx ./anotherchannel.tx -profile TwoOrgsChannel -channelID anotherchannel
# peer创建新的channel，加入新的channel
peer channel create -o orderer.example.com:7050 -c anotherchannel -f ./anotherchannel.tx 
peer channel join -b anotherchannel.block
# install chaincode
peer chaincode install -n chaincode_example02 -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/go/chaincode_example02
# init chaincode
peer chaincode instantiate -o orderer.example.com:7050 -C anotherchannel -n chaincode_example02 -v 1.0 -c '{"Args":["init","a","100","b","200"]}'
# 跨channel调用chaincode
peer chaincode invoke -C onechannel -n chaincode_example05 -v 1.0 -c '{"Args":["invoke","chaincode_example02","sum","anotherchannel"]}'


############################################# enccc_example
# prerequire chaincode最好放在GOPATH下
go get -u github.com/kardianos/govendor
govendor init
govendor add +external
# install chaincode
peer chaincode install -n enccc_example -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/go/enccc_example
# init chaincode
peer chaincode instantiate -o orderer.example.com:7050 -C onechannel -n enccc_example -v 1.0 -c '{"Args":["init"]}'
# invoke chaincode
ENCKEY=`openssl rand 32 -base64` && DECKEY=$ENCKEY
peer chaincode invoke -n enccc_example -C onechannel -c '{"Args":["ENCRYPT","key1","value1"]}' --transient "{\"ENCKEY\":\"$ENCKEY\"}" 
# query chaincode
peer chaincode query -n enccc_example -C onechannel -c '{"Args":["DECRYPT","key1"]}' --transient "{\"DECKEY\":\"$DECKEY\"}"
# invoke chaincode
SIGKEY=`openssl ecparam -name prime256v1 -genkey | tail -n5 | base64 -w0` && VERKEY=$SIGKEY
peer chaincode invoke -n enccc_example -C onechannel -c '{"Args":["ENCRYPTSIGN","key3","value3"]}' --transient "{\"ENCKEY\":\"$ENCKEY\",\"SIGKEY\":\"$SIGKEY\"}"
# query chaincode
peer chaincode query -n enccc_example -C onechannel -c '{"Args":["DECRYPTVERIFY","key3"]}' --transient "{\"DECKEY\":\"$DECKEY\",\"VERKEY\":\"$VERKEY\"}"



############################################# map
# install chaincode
peer chaincode install -n map -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/go/map
# init chaincode
peer chaincode instantiate -o orderer.example.com:7050 -C onechannel -n map -v 1.0 -c '{"Args":["init"]}'
# invoke chaincode PutState
peer chaincode invoke -C onechannel -n map -v 1.0 -c '{"Args":["put","a","200"]}'
peer chaincode invoke -C onechannel -n map -v 1.0 -c '{"Args":["put","b","600"]}'
# invoke chaincode DelState
peer chaincode invoke -C onechannel -n map -v 1.0 -c '{"Args":["remove","a"]}'
# invoke chaincode GetState
peer chaincode invoke -C onechannel -n map -v 1.0 -c '{"Args":["get","a"]}'
# invoke chaincode GetStateByRange
peer chaincode invoke -C onechannel -n map -v 1.0 -c '{"Args":["keys","",""]}'
# invoke chaincode GetQueryResult
peer chaincode invoke -C onechannel -n map -v 1.0 -c '{"Args":["query"]}'
# invoke chaincode GetHistoryForKey
peer chaincode invoke -C onechannel -n map -v 1.0 -c '{"Args":["history","b"]}'


############################################# marbles02
# install chaincode
peer chaincode install -n marbles02 -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/go/marbles02
# init chaincode
peer chaincode instantiate -o orderer.example.com:7050 -C onechannel -n marbles02 -v 1.0 -c '{"Args":["init"]}'
# invoke chaincode CreateCompositeKey
peer chaincode invoke -C onechannel -n marbles02 -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
peer chaincode invoke -C onechannel -n marbles02 -c '{"Args":["initMarble","marble2","red","50","tom"]}'
peer chaincode invoke -C onechannel -n marbles02 -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
peer chaincode invoke -C onechannel -n marbles02 -c '{"Args":["transferMarble","marble2","jerry"]}'
# invoke chaincode GetStateByPartialCompositeKey SplitCompositeKey
peer chaincode invoke -C onechannel -n marbles02 -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
peer chaincode invoke -C onechannel -n marbles02 -c '{"Args":["delete","marble1"]}'
# invoke chaincode GetStateByRange 
peer chaincode query -C onechannel -n marbles02 -c '{"Args":["readMarble","marble1"]}'
peer chaincode query -C onechannel -n marbles02 -c '{"Args":["getMarblesByRange","marble1","marble3"]}'
peer chaincode query -C onechannel -n marbles02 -c '{"Args":["getHistoryForMarble","marble1"]}'
# invoke chaincode rich query 
peer chaincode query -C onechannel -n marbles02 -c '{"Args":["queryMarblesByOwner","tom"]}'
peer chaincode query -C onechannel -n marbles02 -c '{"Args":["queryMarbles","{\"selector\":{\"owner\":\"tom\"}}"]}'
# 创建index
curl -i -X POST -H "Content-Type: application/json" -d "{\"index\":{\"fields\":[\"docType\",\"owner\"]},\"name\":\"indexOwner\",\"ddoc\":\"indexOwnerDoc\",\"type\":\"json\"}" http://192.168.1.143:5984/onechannel_marbles02/_index
curl -i -X POST -H "Content-Type: application/json" -d "{\"index\":{\"fields\":[{\"size\":\"desc\"},{\"docType\":\"desc\"},{\"owner\":\"desc\"}]},\"ddoc\":\"indexSizeSortDoc\", \"name\":\"indexSizeSortDesc\",\"type\":\"json\"}" http://192.168.1.143:5984/onechannel_marbles02/_index
# query chaincode
peer chaincode query -C onechannel -n marbles02 -c '{"Args":["queryMarbles","{\"selector\":{\"docType\":\"marble\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}"]}'
peer chaincode query -C onechannel -n marbles02 -c '{"Args":["queryMarbles","{\"selector\":{\"docType\":{\"$eq\":\"marble\"},\"owner\":{\"$eq\":\"tom\"},\"size\":{\"$gt\":0}},\"fields\":[\"docType\",\"owner\",\"size\"],\"sort\":[{\"size\":\"desc\"}],\"use_index\":\"_design/indexSizeSortDoc\"}"]}'

############################################# passthru 一个chaincode调用另一个chaincode，参考chaincode_example04和chaincode_example05



############################################# sleeper chaincode sleep for a bit
# install chaincode
peer chaincode install -n sleeper -v 1.0 -p github.com/hyperledger/fabric/peer/chaincode/go/sleeper
# init chaincode
peer chaincode instantiate -o orderer.example.com:7050 -C onechannel -n sleeper -v 1.0 -c '{"Args":["2000"]}'
# invoke chaincode
peer chaincode invoke -C onechannel -n sleeper -v 1.0 -c '{"Args":["put","a","avalue","2000"]}'
peer chaincode invoke -C onechannel -n sleeper -v 1.0 -c '{"Args":["get","a","2000"]}'
