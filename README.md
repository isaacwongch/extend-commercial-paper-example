# Extending the Commercial Paper Network

![cp.network](./resource/develop.diagram.2.png)

The aim of this project is to walk you through every little step in creating a realistic fabric network, with reference to the existing commercial paper sample in fabric-sample.

## Project Structure

## An In-Depth Look on configtx.yaml

In the PaperNet network, we define a consortium called PapernetConsortium which includes three organizations -- DigiBank, MangetoCorp and HedgeMatic. On the other hand, we have another organization called MiddleBank to be the orderer organization. Note that MiddleBank is not part of the consortium, this serves act as an indepedant unit that will not participate in any transactions.

```yaml
Profiles:

    SystemGenesis:
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *MiddleBank
        Consortiums:
            PapernetConsortium:
                Organizations:
                    - *MagnetoCorp
                    - *DigiBank
                    - *HedgeMatic
    PapernetChannel:
        Consortium: PapernetConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *MagnetoCorp
                - *DigiBank
                - *HedgeMatic
```

A few points to note from the above YAML.
* You need a consortium to create a channel
* Organizations defined under PaperNetChannel must be part of the consortium linked

### Command

#### Create the orderer genesis block
```
../bin/configtxgen -profile SystemGenesis -outputBlock ./channel-artifacts/genesis.block
```

The created genesis block will be referenced by the docker-compose file while creating the orderer container (environemnt & volume)

## Create Cryptographic Materials for Network Identities

In a production environment, we probably will not use cryptogen tool to create the cryptographic material for those network entities. This tool is solely for demo purpose because it is not desirable to distribute the private key across the network. A good practice should be using Fabric CA or CA instead.

Network Entities presented in PaperNet:

* DigiBank (2 peers, 2 users and 1 admin with NodeOU enabled)
* MagnetoCorp (1 peer, 2 user and 1 admin)
* HedgeMatic (1 peer, 1 user and 1 admin with NodeOU enabled)

```
ordererOrganizations
│   └── papernet.com
│       ├── ca
│       ├── msp
│       │   ├── admincerts
│       │   ├── cacerts
│       │   └── tlscacerts
│       ├── orderers
│       │   └── middleBank.papernet.com
│       │       ├── msp
│       │       │   ├── admincerts
│       │       │   ├── cacerts
│       │       │   ├── keystore
│       │       │   ├── signcerts
│       │       │   └── tlscacerts
│       │       └── tls
│       ├── tlsca
│       └── users
│           └── Admin@papernet.com
│               ├── msp
│               │   ├── admincerts
│               │   ├── cacerts
│               │   ├── keystore
│               │   ├── signcerts
│               │   └── tlscacerts
│               └── tls
```
```
peerOrganizations
    ├── digibank.papernet.com
    │   ├── ca
    │   ├── msp
    │   │   ├── admincerts
    │   │   ├── cacerts
    │   │   └── tlscacerts
    │   ├── peers
    │   │   ├── peer0.digibank.papernet.com
    │   │   │   ├── msp
    │   │   │   │   ├── admincerts
    │   │   │   │   ├── cacerts
    │   │   │   │   ├── keystore
    │   │   │   │   ├── signcerts
    │   │   │   │   └── tlscacerts
    │   │   │   └── tls
    │   │   └── peer1.digibank.papernet.com
    │   │       ├── msp
    │   │       │   ├── admincerts
    │   │       │   ├── cacerts
    │   │       │   ├── keystore
    │   │       │   ├── signcerts
    │   │       │   └── tlscacerts
    │   │       └── tls
    │   ├── tlsca
    │   └── users
    │       ├── Admin@digibank.papernet.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   ├── cacerts
    │       │   │   ├── keystore
    │       │   │   ├── signcerts
    │       │   │   └── tlscacerts
    │       │   └── tls
    │       ├── User1@digibank.papernet.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   ├── cacerts
    │       │   │   ├── keystore
    │       │   │   ├── signcerts
    │       │   │   └── tlscacerts
    │       │   └── tls
    │       └── User2@digibank.papernet.com
    │           ├── msp
    │           │   ├── admincerts
    │           │   ├── cacerts
    │           │   ├── keystore
    │           │   ├── signcerts
    │           │   └── tlscacerts
    │           └── tls
    ├── hedgematic.papernet.com
    │   ├── ca
    │   ├── msp
    │   │   ├── admincerts
    │   │   ├── cacerts
    │   │   └── tlscacerts
    │   ├── peers
    │   │   └── peer0.hedgematic.papernet.com
    │   │       ├── msp
    │   │       │   ├── admincerts
    │   │       │   ├── cacerts
    │   │       │   ├── keystore
    │   │       │   ├── signcerts
    │   │       │   └── tlscacerts
    │   │       └── tls
    │   ├── tlsca
    │   └── users
    │       ├── Admin@hedgematic.papernet.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   ├── cacerts
    │       │   │   ├── keystore
    │       │   │   ├── signcerts
    │       │   │   └── tlscacerts
    │       │   └── tls
    │       ├── User1@hedgematic.papernet.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   ├── cacerts
    │       │   │   ├── keystore
    │       │   │   ├── signcerts
    │       │   │   └── tlscacerts
    │       │   └── tls
    │       └── User2@hedgematic.papernet.com
    │           ├── msp
    │           │   ├── admincerts
    │           │   ├── cacerts
    │           │   ├── keystore
    │           │   ├── signcerts
    │           │   └── tlscacerts
    │           └── tls
    └── magnetoCorp.papernet.com
        ├── ca
        ├── msp
        │   ├── admincerts
        │   ├── cacerts
        │   └── tlscacerts
        ├── peers
        │   └── peer0.magnetoCorp.papernet.com
        │       ├── msp
        │       │   ├── admincerts
        │       │   ├── cacerts
        │       │   ├── keystore
        │       │   ├── signcerts
        │       │   └── tlscacerts
        │       └── tls
        ├── tlsca
        └── users
            ├── Admin@magnetoCorp.papernet.com
            │   ├── msp
            │   │   ├── admincerts
            │   │   ├── cacerts
            │   │   ├── keystore
            │   │   ├── signcerts
            │   │   └── tlscacerts
            │   └── tls
            └── User1@magnetoCorp.papernet.com
                ├── msp
                │   ├── admincerts
                │   ├── cacerts
                │   ├── keystore
                │   ├── signcerts
                │   └── tlscacerts
                └── tls
```

## To-do List
* Storing off-chain data such as the certificate using IPFS
* A master-details data structure demonstration
* Using Fabric CA in the network
* Deploy the network on Kubernetes
* Use Kafka for the orderer
* Use of private collection
* Demonstrate service discovery capability
* Meaning of CouchDB namespace