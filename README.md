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
# Benchmark Configuration File
An example of Benchmark Configuration file is this one. This file is important when you are going to run your tests
```
test:
  name: simple
  description: This is an example benchmark for caliper, to test the backend DLT's
    performance with simple account opening & querying transactions
  clients:
    type: local
    number: 1
  rounds:
  - label: open
    description: Test description for the opening of an account through the deployed chaincode
    txNumber:
    - 100
    rateControl:
    - type: fixed-rate
      opts:
        tps: 50
    arguments:
      money: 10000
    callback: benchmark/simple/open.js
  - label: query
    description: Test description for the query performance of the deployed chaincode
    txNumber:
    - 100
    rateControl:
    - type: fixed-rate
      opts:
        tps: 100
    callback: benchmark/simple/query.js
  - label: transfer
    description: Test description for transfering money between accounts
    txNumber:
        - 100
    rateControl:
        - type: fixed-rate
          opts:
              tps: 50
    arguments:
        money: 100
    callback: benchmark/simple/transfer.js
monitor:
  type:
  - docker
  - process
  docker:
    name:
    - all
  process:
  - command: node
    arguments: local-client.js
    multiOutput: avg
  interval: 1
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
These are the keys to provide inside the configuration file under the ethereum one:

* URL of the node to connect to. Only http is currently supported.
* Deployer address with which deploy required contracts
* Deployer address private key the private key of the deployer address
* Deployer address password to unlock the deployer address
* Address from which invoke methods of the benchmark
* Private Key the private key of the benchmark address
* Password to unlock the benchmark address
* Number of confirmation blocks to wait to consider a transaction as successfully accepted in the chain
* Contracts configuration

# Running a test
To run a test with Caliper you just have to run the following command:
```
 caliper benchmark run -w <path to workspace> -c <benchmark config> -n <blockchain config>
```
# Types of metrics you can see and use
Depending on what kind of metrics you want to test you can use the next ones:
* Transaction/read throughput
* Transaction/read latency (minimum, maximum, average, percentile)
* Resource consumption (CPU, Memory, Network IO, …)

# Some References
"Architecture -", Hyperledger Caliper, 2020. [Online]. Available: https://hyperledger.github.io/caliper/v0.3/architecture/. [Accessed: 14- April- 2020].

"Writing Adapters", Hyperledger Caliper, 2020. [Online]. Available: https://hyperledger.github.io/caliper/v0.3/writing-adaptors/. [Accessed: 14- April- 2020].

"Rate Controllers", Hyperledger Caliper, 2020. [Online]. Available: https://hyperledger.github.io/caliper/v0.3/rate-controllers/. [Accessed: 14- April- 2020].

"Etherum Configuration", Hyperledger Caliper, 2020. [Online]. Available: https://hyperledger.github.io/caliper/v0.3/ethereum-config/. [Accessed: 14- April- 2020].
