## Hyperledger Fabric Samples

Please visit the [installation instructions](http://hyperledger-fabric.readthedocs.io/en/latest/samples.html).

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>

## How to use this on Mac OS X
1. /tmpディレクトリにfabric-samplesをクローンして、バイナリファイルやdockerイメージを導入します。

~~~~
cd /tmp
git clone https://github.com/hyperledger/fabric-samples.git``
cd fabric-samples/
curl -sSL https://goo.gl/PabWJX | bash
export PATH=$PWD/bin:$PATH # ダウンロードしたbinフォルダ下のpeerコマンドなどにパスを通します。
~~~~

2. first-networkディレクトリでbyfn.shのテストスクリプトを実行

~~~~
cd first-network/ 
./byfn.sh -m generate
./byfn.sh -m up  # ENDと出たらCtrl+Cで脱出。
~~~~

3) crypto-configフォルダ（証明書）とchannel-artifactsを作成
./byfn.sh -m down
./byfn.sh -m generate

4) cliコンテナを作らないdocker-composeを実行
 
cp docker-compose-cli.yaml docker-compose.yaml
 
vi docker-compose.yaml
(cli:以下の行を丸ごと削除する
 
  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    ...
 
)
 
docker-compose -f docker-compose.yaml up -d

5) peerコマンド実行のための初期設定
以下の環境変数を~/.bash_profileとかに設定
 
 
export GOPATH=/opt/gopath
export CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
export CORE_LOGGING_LEVEL=DEBUG
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_CERT_FILE=/tmp/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/tmp/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/tmp/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/tmp/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

/etc/hostsを修正して、ordererとpeerを127.0.0.1に設定
 
127.0.0.1 orderer.example.com
127.0.0.1 peer0.org1.example.com
127.0.0.1 peer1.org1.example.com
127.0.0.1 peer0.org2.example.com
127.0.0.1 peer1.org2.example.com

 
6) channelを作成してjoin
 
peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls true --cafile /tmp/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

peer channel join -b mychannel.block
 

 
7) chaincodeをインストール（$GOPATH下にgithub.com/.../chaincode_example02を配置すること）
peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
8) chaincodeを初期化
 
peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile /tmp/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
 

9) chaincodeをinvoke
 
peer chaincode invoke -o orderer.example.com:7050  --tls true --cafile /tmp/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'
 

10) chaincodeをquery
 
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'

 
11) 初期化してやり直したい時は3)からやり直し
