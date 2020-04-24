# alastria-load-testing
Documentation and scripts for load testing for Alastria Red-T Network
(ideally, the same type of load testing will be executed also in Alastria Red-B Network)

# Requirements
It is necessary to install in your system previous to work with Caliper the next components:
 * NodeJS 8 (LTS), 9, o 10 (LTS).
 * node-gyp
 * Docker
 * Docker-compose
 
 # Installation
Once you have requirements completed it is necessary to clone the caliper repository on your machine where you are going to run the caliper test.
So executing git clone https://github.com/hyperledger/caliper.git you would be able to start to work with caliper
Once you have clone this repository you have to go inside the main directory of caliper:

```
   cd caliper-benchmarks
   npm init -y
   npm install --only=prod @hyperledger/caliper-cli@0.3.0
   npx caliper bind --caliper-bind-sut ethereum:latest
   npx caliper launch master --caliper-workspace . --caliper-benchconfig benchmarks/scenario/simple/config.yaml \
    --caliper-networkconfig networks/ethereum/1node-clique/docker-compose.yml
```
# Etherum Configuration File
It is important to know you must configure the Etherum configuration file to let Caliper to know some aspects about your Nodes configuration, for this reason an example of configuration file would be the next one:
```
{
    "caliper": {
        "blockchain": "ethereum",
        "command" : {
            "start": "docker-compose -f network/ethereum/1node-clique/docker-compose.yml up -d && sleep 3",
            "end" : "docker-compose -f network/ethereum/1node-clique/docker-compose.yml down"
          }
    },
    "ethereum": {
        "url": "http://localhost:8545",
        "contractDeployerAddress": "0xc0A8e4D217eB85b812aeb1226fAb6F588943C2C2",
        "contractDeployerAddressPassword": "password",
        "fromAddress": "0xc0A8e4D217eB85b812aeb1226fAb6F588943C2C2",
        "fromAddressPassword": "password",
        "transactionConfirmationBlocks": 12,
        "contracts": {
            "simple": {
                "path": "src/contract/ethereum/simple/simple.json"
            }
        }
    }
}
```
