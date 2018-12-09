# Create the PaperNet Network

## Create the Orderer Genesis Block & Channel Configuration Transaction
Go to the directory *global-config*
```
> mkdir channel-artifacts
> ../bin/configtxgen -profile SystemGenesis -outputBlock ./channel-artifacts/genesis.block
```

You should see output like this:
```
2018-12-09 11:42:35.102 HKT [common/tools/configtxgen] main -> WARN 001 Omitting the channel ID for configtxgen for output operations is deprecated.  Explicitly passing the channel ID will be required in the future, defaulting to 'testchainid'.
2018-12-09 11:42:35.102 HKT [common/tools/configtxgen] main -> INFO 002 Loading configuration
2018-12-09 11:42:35.108 HKT [common/tools/configtxgen/encoder] NewChannelGroup -> WARN 003 Default policy emission is deprecated, please include policy specifications for the channel group in configtx.yaml
2018-12-09 11:42:35.108 HKT [common/tools/configtxgen/encoder] NewOrdererGroup -> WARN 004 Default policy emission is deprecated, please include policy specifications for the orderer group in configtx.yaml
2018-12-09 11:42:35.109 HKT [common/tools/configtxgen/encoder] NewOrdererOrgGroup -> WARN 005 Default policy emission is deprecated, please include policy specifications for the orderer org group MiddleBank in configtx.yaml
2018-12-09 11:42:35.110 HKT [common/tools/configtxgen/encoder] NewOrdererOrgGroup -> WARN 006 Default policy emission is deprecated, please include policy specifications for the orderer org group MagnetoCorp in configtx.yaml
2018-12-09 11:42:35.110 HKT [common/tools/configtxgen/encoder] NewOrdererOrgGroup -> WARN 007 Default policy emission is deprecated, please include policy specifications for the orderer org group DigiBank in configtx.yaml
2018-12-09 11:42:35.110 HKT [common/tools/configtxgen/encoder] NewOrdererOrgGroup -> WARN 008 Default policy emission is deprecated, please include policy specifications for the orderer org group HedgeMatic in configtx.yaml
2018-12-09 11:42:35.111 HKT [common/tools/configtxgen] doOutputBlock -> INFO 009 Generating genesis block
2018-12-09 11:42:35.111 HKT [common/tools/configtxgen] doOutputBlock -> INFO 00a Writing genesis block
```

```
> export CHANNEL_NAME=papernet  && ../bin/configtxgen -profile PapernetChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```

Output would be similar to this:
```
2018-12-09 11:58:08.038 HKT [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-12-09 11:58:08.044 HKT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-12-09 11:58:08.044 HKT [common/tools/configtxgen/encoder] NewApplicationGroup -> WARN 003 Default policy emission is deprecated, please include policy specifications for the application group in configtx.yaml
2018-12-09 11:58:08.044 HKT [common/tools/configtxgen/encoder] NewApplicationOrgGroup -> WARN 004 Default policy emission is deprecated, please include policy specifications for the application org group MagnetoCorp in configtx.yaml
2018-12-09 11:58:08.045 HKT [common/tools/configtxgen/encoder] NewApplicationOrgGroup -> WARN 005 Default policy emission is deprecated, please include policy specifications for the application org group DigiBank in configtx.yaml
2018-12-09 11:58:08.045 HKT [common/tools/configtxgen/encoder] NewApplicationOrgGroup -> WARN 006 Default policy emission is deprecated, please include policy specifications for the application org group HedgeMatic in configtx.yaml
2018-12-09 11:58:08.047 HKT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 007 Writing new channel tx
```

# Define the Anchor Peer for MagnetoCorp
```
../bin/configtxgen -profile PapernetChannel -outputAnchorPeersUpdate ./channel-artifacts/MagnetoCorpMSPanchors.tx -channelID $CHANNEL_NAME -asOrg MagnetoCorp
```

# Define the Anchor Peer for DigiBank
```
../bin/configtxgen -profile PapernetChannel -outputAnchorPeersUpdate ./channel-artifacts/DigiBankMSPanchors.tx -channelID $CHANNEL_NAME -asOrg DigiBank
```

```
2018-12-09 12:08:27.727 HKT [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-12-09 12:08:27.733 HKT [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-12-09 12:08:27.733 HKT [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```

# Start the PaperNet Network

Let's take the role of the orderer first, go to the directory */extend-commercial-paper-example/organization/middlebank/configuration*, run the following command to start the orderer container. At this point, we are using the solo mode where only one orderer presents in the network. Later, we will change it to Kafka mode.

```
(middlebank) > docker-compose -f docker-compose.yml up
```

* Run docker ps to list all the running container
```
CONTAINER ID        IMAGE                        COMMAND                  CREATED              STATUS              PORTS                                                                       NAMES
77cb0715f3ed        hyperledger/fabric-tools     "/bin/bash"              About a minute ago   Up 58 seconds                                                                                   cli.magnetocorp.papernet.com
220abffb24c3        gliderlabs/logspout          "/bin/logspout"          About a minute ago   Up About a minute   127.0.0.1:8000->80/tcp                                                      logspout
5b318f009f62        hyperledger/fabric-tools     "/bin/bash"              2 minutes ago        Up 2 minutes                                                                                    cli.digibank.papernet.com
659e16cb18f8        hyperledger/fabric-peer      "peer node start --p…"   7 minutes ago        Up 7 minutes        0.0.0.0:18051->7051/tcp, 0.0.0.0:18052->7052/tcp, 0.0.0.0:18053->7053/tcp   peer1.magnetocorp.papernet.com
3d1e6ee9ee6f        hyperledger/fabric-peer      "peer node start --p…"   7 minutes ago        Up 7 minutes        0.0.0.0:17051->7051/tcp, 0.0.0.0:17052->7052/tcp, 0.0.0.0:17053->7053/tcp   peer0.magnetocorp.papernet.com
1e3568cc6ebe        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   7 minutes ago        Up 7 minutes        4369/tcp, 9100/tcp, 0.0.0.0:16984->5984/tcp                                 peer1.magnetocorp.couchdb
f1ac7740fdc8        hyperledger/fabric-ca        "sh -c 'fabric-ca-se…"   7 minutes ago        Up 7 minutes        0.0.0.0:17054->7054/tcp                                                     ca.magnetocorp.papernet.com
3bd33a77ca51        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   7 minutes ago        Up 7 minutes        4369/tcp, 9100/tcp, 0.0.0.0:15984->5984/tcp                                 peer0.magnetocorp.couchdb
730bec784811        hyperledger/fabric-peer      "peer node start --p…"   10 minutes ago       Up 10 minutes       0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.digibank.papernet.com
c6f25915ddcb        hyperledger/fabric-peer      "peer node start --p…"   10 minutes ago       Up 10 minutes       0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer2.digibank.papernet.com
26bd2d61c73f        hyperledger/fabric-peer      "peer node start --p…"   10 minutes ago       Up 10 minutes       0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.digibank.papernet.com
8ec95086366e        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   10 minutes ago       Up 10 minutes       4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  peer2.digibank.couchdb
8a359c52a59e        hyperledger/fabric-ca        "sh -c 'fabric-ca-se…"   10 minutes ago       Up 10 minutes       0.0.0.0:7054->7054/tcp                                                      ca.digibank.papernet.com
3dd6411c452b        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   10 minutes ago       Up 10 minutes       4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  peer1.digibank.couchdb
289e9c490dfa        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   10 minutes ago       Up 10 minutes       4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  peer0.digibank.couchdb
53c14644eb6a        hyperledger/fabric-orderer   "orderer"                2 hours ago          Up 14 minutes       0.0.0.0:7050->7050/tcp                                                      middlebank.papernet.com
```

```
> docker exec -it cli.digibank.papernet.com bash
root@dbc04a2d9b46:/opt/gopath/src/github.com/hyperledger/fabric/peer# echo $CORE_PEER_ADDRESS
peer0.digibank.papernet.com:7051
root@dbc04a2d9b46:/opt/gopath/src/github.com/hyperledger/fabric/peer# echo $CORE_PEER_MSPCONFIGPATH
/opt/gopath/src/github.com/identity/users/Admin@digibank.papernet.com/msp
root@dbc04a2d9b46:/opt/gopath/src/github.com/hyperledger/fabric/peer# echo $CORE_PEER_LOCALMSPID
DigiBankMSP
```

```
export CHANNEL_NAME=papernet
peer channel create -o middlebank.papernet.com:7050 -c $CHANNEL_NAME -f ./configuration/channel-artifacts/channel.tx
peer channel join -b papernet.block
```

By the time one issue this channel configuration transaction to the orderer, you should observe the following log in the middlebank orderer.
```
middlebank.papernet.com    | 2018-12-09 07:37:41.638 UTC [fsblkstorage] newBlockfileMgr -> INFO 008 Getting block information from block storage
middlebank.papernet.com    | 2018-12-09 07:37:41.658 UTC [orderer/commmon/multichannel] newChain -> INFO 009 Created and starting new chain papernet
```

```
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.582 UTC [endorser] callChaincode -> INFO 029 [][218993ac] Entry chaincode: name:"cscc"
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.586 UTC [ledgermgmt] CreateLedger -> INFO 02a Creating ledger [papernet] with genesis block
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.590 UTC [fsblkstorage] newBlockfileMgr -> INFO 02b Getting block information from block storage
peer0.digibank.couchdb         | [notice] 2018-12-09T07:40:50.598872Z nonode@nohost <0.13283.2> bc392495cb peer0.digibank.couchdb:5984 172.19.0.7 undefined GET /papernet_/ 404 ok 1
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.649 UTC [couchdb] CreateDatabaseIfNotExist -> INFO 02c Created state database papernet_
peer0.digibank.couchdb         | [notice] 2018-12-09T07:40:50.650092Z nonode@nohost <0.13283.2> 10d249e283 peer0.digibank.couchdb:5984 172.19.0.7 undefined PUT /papernet_/ 201 ok 42
peer0.digibank.couchdb         | [notice] 2018-12-09T07:40:50.695730Z nonode@nohost <0.13283.2> 6d5999321f peer0.digibank.couchdb:5984 172.19.0.7 undefined GET /papernet_/ 200 ok 26
peer0.digibank.couchdb         | [notice] 2018-12-09T07:40:50.738788Z nonode@nohost <0.13283.2> 96041f8425 peer0.digibank.couchdb:5984 172.19.0.7 undefined POST /papernet_/_all_docs?include_docs=true 200 ok 40
peer0.digibank.couchdb         | [notice] 2018-12-09T07:40:50.779990Z nonode@nohost <0.13283.2> f8f8f0b43d peer0.digibank.couchdb:5984 172.19.0.7 undefined POST /papernet_/_bulk_docs 201 ok 41
peer0.digibank.couchdb         | [notice] 2018-12-09T07:40:50.786524Z nonode@nohost <0.13283.2> b9c9b49777 peer0.digibank.couchdb:5984 172.19.0.7 undefined POST /papernet_/_ensure_full_commit 201 ok 3
peer0.digibank.couchdb         | [notice] 2018-12-09T07:40:50.794362Z nonode@nohost <0.13283.2> 9d6f46997e peer0.digibank.couchdb:5984 172.19.0.7 undefined GET /papernet_/statedb_savepoint?attachments=true 404 ok 7
peer0.digibank.couchdb         | [notice] 2018-12-09T07:40:50.798048Z nonode@nohost <0.13283.2> ae0a56fc24 peer0.digibank.couchdb:5984 172.19.0.7 undefined PUT /papernet_/statedb_savepoint 201 ok 3
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.806 UTC [kvledger] CommitWithPvtData -> INFO 02d [papernet] Committed block [0] with 1 transaction(s) in 154ms (state_validation=3ms block_commit=12ms state_commit=129ms)
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.814 UTC [ledgermgmt] CreateLedger -> INFO 02e Created ledger [papernet] with genesis block
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.856 UTC [gossip/gossip] JoinChan -> INFO 02f Joining gossip network of channel papernet with 3 organizations
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.857 UTC [gossip/gossip] learnAnchorPeers -> INFO 030 No configured anchor peers of DigiBankMSP for channel papernet to learn about
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.858 UTC [gossip/gossip] learnAnchorPeers -> INFO 031 No configured anchor peers of HedgeMaticMSP for channel papernet to learn about
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.858 UTC [gossip/gossip] learnAnchorPeers -> INFO 032 No configured anchor peers of MagnetoCorpMSP for channel papernet to learn about
peer0.digibank.couchdb         | [notice] 2018-12-09T07:40:50.854192Z nonode@nohost <0.13283.2> c171c308a2 peer0.digibank.couchdb:5984 172.19.0.7 undefined GET /papernet_/resourcesconfigtx.CHANNEL_CONFIG_KEY?attachments=true 200 ok 35
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.878 UTC [gossip/state] NewGossipStateProvider -> INFO 033 Updating metadata information, current ledger sequence is at = 0, next expected block is = 1
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.880 UTC [sccapi] deploySysCC -> INFO 034 system chaincode lscc/papernet(github.com/hyperledger/fabric/core/scc/lscc) deployed
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.881 UTC [cscc] Init -> INFO 035 Init CSCC
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.882 UTC [sccapi] deploySysCC -> INFO 036 system chaincode cscc/papernet(github.com/hyperledger/fabric/core/scc/cscc) deployed
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.883 UTC [qscc] Init -> INFO 037 Init QSCC
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.884 UTC [sccapi] deploySysCC -> INFO 038 system chaincode qscc/papernet(github.com/hyperledger/fabric/core/scc/qscc) deployed
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.885 UTC [sccapi] deploySysCC -> INFO 039 system chaincode +lifecycle/papernet(github.com/hyperledger/fabric/core/chaincode/lifecycle) deployed
peer0.digibank.papernet.com    | 2018-12-09 07:40:50.887 UTC [endorser] callChaincode -> INFO 03a [][218993ac] Exit chaincode: name:"cscc"  (304ms)
peer0.digibank.couchdb         | [notice] 2018-12-09T07:40:50.927128Z nonode@nohost <0.13313.2> 5d9d721c5c peer0.digibank.couchdb:5984 172.19.0.7 undefined GET /papernet_/_index 200 ok 140
peer0.digibank.papernet.com    | 2018-12-09 07:40:56.882 UTC [gossip/election] beLeader -> INFO 03b [247 159 218 155 21 44 243 200 138 194 251 186 180 139 25 24 65 101 130 100 130 2 39 148 36 161 43 35 180 186 130 107] : Becoming a leader
peer0.digibank.papernet.com    | 2018-12-09 07:40:56.882 UTC [gossip/service] func1 -> INFO 03c Elected as a leader, starting delivery service for channel papernet
```


# Appendix
```
flag needs an argument: -outputAnchorPeersUpdate
Usage of ../bin/configtxgen:
  -asOrg string
    	Performs the config generation as a particular organization (by name), only including values in the write set that org (likely) has privilege to set
  -channelID string
    	The channel ID to use in the configtx
  -configPath string
    	The path containing the configuration to use (if set)
  -inspectBlock string
    	Prints the configuration contained in the block at the specified path
  -inspectChannelCreateTx string
    	Prints the configuration contained in the transaction at the specified path
  -outputAnchorPeersUpdate string
    	Creates an config update to update an anchor peer (works only with the default channel creation, and only for the first update)
  -outputBlock string
    	The path to write the genesis block to (if set)
  -outputCreateChannelTx string
    	The path to write a channel creation configtx to (if set)
  -printOrg string
    	Prints the definition of an organization as JSON. (useful for adding an org to a channel manually)
  -profile string
    	The profile from configtx.yaml to use for generation. (default "SampleInsecureSolo")
  -version
    	Show version information
```

## System Log when starting the orderer container
```
middlebank.papernet.com    | 2018-12-09 04:40:22.677 UTC [localconfig] completeInitialization -> INFO 001 Kafka.Version unset, setting to 0.10.2.0
middlebank.papernet.com    | 2018-12-09 04:40:22.718 UTC [orderer/common/server] prettyPrintStruct -> INFO 002 Orderer config values:
middlebank.papernet.com    | 	General.LedgerType = "file"
middlebank.papernet.com    | 	General.ListenAddress = "0.0.0.0"
middlebank.papernet.com    | 	General.ListenPort = 7050
middlebank.papernet.com    | 	General.TLS.Enabled = false
middlebank.papernet.com    | 	General.TLS.PrivateKey = "/etc/hyperledger/fabric/tls/server.key"
middlebank.papernet.com    | 	General.TLS.Certificate = "/etc/hyperledger/fabric/tls/server.crt"
middlebank.papernet.com    | 	General.TLS.RootCAs = [/etc/hyperledger/fabric/tls/ca.crt]
middlebank.papernet.com    | 	General.TLS.ClientAuthRequired = false
middlebank.papernet.com    | 	General.TLS.ClientRootCAs = []
middlebank.papernet.com    | 	General.Keepalive.ServerMinInterval = 1m0s
middlebank.papernet.com    | 	General.Keepalive.ServerInterval = 2h0m0s
middlebank.papernet.com    | 	General.Keepalive.ServerTimeout = 20s
middlebank.papernet.com    | 	General.GenesisMethod = "file"
middlebank.papernet.com    | 	General.GenesisProfile = "SampleInsecureSolo"
middlebank.papernet.com    | 	General.SystemChannel = "test-system-channel-name"
middlebank.papernet.com    | 	General.GenesisFile = "/etc/hyperledger/configtx/genesis.block"
middlebank.papernet.com    | 	General.Profile.Enabled = false
middlebank.papernet.com    | 	General.Profile.Address = "0.0.0.0:6060"
middlebank.papernet.com    | 	General.LogLevel = "info"
middlebank.papernet.com    | 	General.LogFormat = "%{color}%{time:2006-01-02 15:04:05.000 MST} [%{module}] %{shortfunc} -> %{level:.4s} %{id:03x}%{color:reset} %{message}"
middlebank.papernet.com    | 	General.LocalMSPDir = "/etc/hyperledger/msp/orderer/msp"
middlebank.papernet.com    | 	General.LocalMSPID = "MiddleBankMSP"
middlebank.papernet.com    | 	General.BCCSP.ProviderName = "SW"
middlebank.papernet.com    | 	General.BCCSP.SwOpts.SecLevel = 256
middlebank.papernet.com    | 	General.BCCSP.SwOpts.HashFamily = "SHA2"
middlebank.papernet.com    | 	General.BCCSP.SwOpts.Ephemeral = false
middlebank.papernet.com    | 	General.BCCSP.SwOpts.FileKeystore.KeyStorePath = "/etc/hyperledger/msp/orderer/msp/keystore"
middlebank.papernet.com    | 	General.BCCSP.SwOpts.DummyKeystore =
middlebank.papernet.com    | 	General.BCCSP.PluginOpts =
middlebank.papernet.com    | 	General.Authentication.TimeWindow = 15m0s
middlebank.papernet.com    | 	FileLedger.Location = "/var/hyperledger/production/orderer"
middlebank.papernet.com    | 	FileLedger.Prefix = "hyperledger-fabric-ordererledger"
middlebank.papernet.com    | 	RAMLedger.HistorySize = 1000
middlebank.papernet.com    | 	Kafka.Retry.ShortInterval = 5s
middlebank.papernet.com    | 	Kafka.Retry.ShortTotal = 10m0s
middlebank.papernet.com    | 	Kafka.Retry.LongInterval = 5m0s
middlebank.papernet.com    | 	Kafka.Retry.LongTotal = 12h0m0s
middlebank.papernet.com    | 	Kafka.Retry.NetworkTimeouts.DialTimeout = 10s
middlebank.papernet.com    | 	Kafka.Retry.NetworkTimeouts.ReadTimeout = 10s
middlebank.papernet.com    | 	Kafka.Retry.NetworkTimeouts.WriteTimeout = 10s
middlebank.papernet.com    | 	Kafka.Retry.Metadata.RetryMax = 3
middlebank.papernet.com    | 	Kafka.Retry.Metadata.RetryBackoff = 250ms
middlebank.papernet.com    | 	Kafka.Retry.Producer.RetryMax = 3
middlebank.papernet.com    | 	Kafka.Retry.Producer.RetryBackoff = 100ms
middlebank.papernet.com    | 	Kafka.Retry.Consumer.RetryBackoff = 2s
middlebank.papernet.com    | 	Kafka.Verbose = false
middlebank.papernet.com    | 	Kafka.Version = 0.10.2.0
middlebank.papernet.com    | 	Kafka.TLS.Enabled = false
middlebank.papernet.com    | 	Kafka.TLS.PrivateKey = ""
middlebank.papernet.com    | 	Kafka.TLS.Certificate = ""
middlebank.papernet.com    | 	Kafka.TLS.RootCAs = []
middlebank.papernet.com    | 	Kafka.TLS.ClientAuthRequired = false
middlebank.papernet.com    | 	Kafka.TLS.ClientRootCAs = []
middlebank.papernet.com    | 	Kafka.SASLPlain.Enabled = false
middlebank.papernet.com    | 	Kafka.SASLPlain.User = ""
middlebank.papernet.com    | 	Kafka.SASLPlain.Password = ""
middlebank.papernet.com    | 	Kafka.Topic.ReplicationFactor = 3
middlebank.papernet.com    | 	Debug.BroadcastTraceDir = ""
middlebank.papernet.com    | 	Debug.DeliverTraceDir = ""
middlebank.papernet.com    | 2018-12-09 04:40:22.731 UTC [fsblkstorage] newBlockfileMgr -> INFO 003 Getting block information from block storage
middlebank.papernet.com    | 2018-12-09 04:40:22.741 UTC [orderer/commmon/multichannel] NewRegistrar -> INFO 004 Starting system channel 'testchainid' with genesis block hash cdf581f242adf6f269b0fe31a080fafe6aa765e969f821b66087e4bfffec23cb and orderer type solo
middlebank.papernet.com    | 2018-12-09 04:40:22.741 UTC [orderer/common/server] Start -> INFO 005 Starting orderer:
middlebank.papernet.com    |  Version: 1.3.0
middlebank.papernet.com    |  Commit SHA: ab0a67a
middlebank.papernet.com    |  Go version: go1.10.4
middlebank.papernet.com    |  OS/Arch: linux/amd64
middlebank.papernet.com    |  Experimental features: false
middlebank.papernet.com    | 2018-12-09 04:40:22.742 UTC [orderer/common/server] Start -> INFO 006 Beginning to serve requests
```

