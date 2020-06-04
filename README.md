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
So executing git clone https://github.com/hyperledger/caliper-benchmarks.git you would be able to start to work with caliper
Once you have clone this repository you have to go inside the main directory of caliper-benchmarks and installa caliper:

```
   cd caliper-benchmarks
   npm init -y
   npm install --only=prod @hyperledger/caliper-cli@0.3.0
   npx caliper bind --caliper-bind-sut ethereum:latest
   npx caliper launch master --caliper-workspace . --caliper-benchconfig benchmarks/scenario/simple/config.yaml \
    --caliper-networkconfig networks/ethereum/1node-clique/ethereum.json
```
As well if you run the last command and you have errors, problably you will have to change your genesis.json code as follow:
```
{
  "config": {
    "chainId": 21194,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "ethash": {}
  },
  "nonce": "0x0",
  "timestamp": "0x5cd09c57",
  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "gasLimit": "0x2faf080",
  "difficulty": "0x80000",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "alloc": {
    "c0a8e4d217eb85b812aeb1226fab6f588943c2c2": {
      "balance": "0x200000000000000000000000000000000000000000000000000000000000000"
    }
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
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
* Contracts configuration.

NOTE: In case you want to run your tests out of the local node, you have to open and other kind of family of protocols, in this case you have to use Personal as well, so open it in your node.
In case you have problems as well with the simple.json, this is the json file you have to use:

```
{
    "contractName": "simple",
    "abi": [
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "string",
            "name": "acc_id",
            "type": "string"
          },
          {
            "internalType": "int256",
            "name": "amount",
            "type": "int256"
          }
        ],
        "name": "open",
        "outputs": [],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      },
      {
        "constant": true,
        "inputs": [
          {
            "internalType": "string",
            "name": "acc_id",
            "type": "string"
          }
        ],
        "name": "query",
        "outputs": [
          {
            "internalType": "int256",
            "name": "amount",
            "type": "int256"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      {
        "constant": false,
        "inputs": [
          {
            "internalType": "string",
            "name": "acc_from",
            "type": "string"
          },
          {
            "internalType": "string",
            "name": "acc_to",
            "type": "string"
          },
          {
            "internalType": "int256",
            "name": "amount",
            "type": "int256"
          }
        ],
        "name": "transfer",
        "outputs": [],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
      }
    ],
    "bytecode": "0x608060405234801561001057600080fd5b506104cf806100206000396000f3fe608060405234801561001057600080fd5b506004361061005d577c010000000000000000000000000000000000000000000000000000000060003504631de45b1081146100625780637c26192914610193578063906412931461024b575b600080fd5b6101916004803603606081101561007857600080fd5b81019060208101813564010000000081111561009357600080fd5b8201836020820111156100a557600080fd5b803590602001918460018302840111640100000000831117156100c757600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600092019190915250929594936020810193503591505064010000000081111561011a57600080fd5b82018360208201111561012c57600080fd5b8035906020019184600183028401116401000000008311171561014e57600080fd5b91908080601f01602080910402602001604051908101604052809392919081815260200183838082843760009201919091525092955050913592506102f3915050565b005b610239600480360360208110156101a957600080fd5b8101906020810181356401000000008111156101c457600080fd5b8201836020820111156101d657600080fd5b803590602001918460018302840111640100000000831117156101f857600080fd5b91908080601f0160208091040260200160405190810160405280939291908181526020018383808284376000920191909152509295506103cb945050505050565b60408051918252519081900360200190f35b6101916004803603604081101561026157600080fd5b81019060208101813564010000000081111561027c57600080fd5b82018360208201111561028e57600080fd5b803590602001918460018302840111640100000000831117156102b057600080fd5b91908080601f0160208091040260200160405190810160405280939291908181526020018383808284376000920191909152509295505091359250610432915050565b806000846040518082805190602001908083835b602083106103265780518252601f199092019160209182019101610307565b51815160209384036101000a60001901801990921691161790529201948552506040519384900381018420805495909503909455505083518392600092869290918291908401908083835b602083106103905780518252601f199092019160209182019101610371565b51815160209384036101000a600019018019909216911617905292019485525060405193849003019092208054939093019092555050505050565b600080826040518082805190602001908083835b602083106103fe5780518252601f1990920191602091820191016103df565b51815160209384036101000a6000190180199092169116179052920194855250604051938490030190922054949350505050565b806000836040518082805190602001908083835b602083106104655780518252601f199092019160209182019101610446565b51815160209384036101000a60001901801990921691161790529201948552506040519384900301909220929092555050505056fea265627a7a7231582085b12c7cb67f4952de46b525e7b167f6740ffb97814392b305af2d550f56472e64736f6c634300050f0032",
    "gas": 4700000,
    "deployedBytecode": "0x608060405234801561001057600080fd5b506004361061005d577c010000000000000000000000000000000000000000000000000000000060003504631de45b1081146100625780637c26192914610193578063906412931461024b575b600080fd5b6101916004803603606081101561007857600080fd5b81019060208101813564010000000081111561009357600080fd5b8201836020820111156100a557600080fd5b803590602001918460018302840111640100000000831117156100c757600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600092019190915250929594936020810193503591505064010000000081111561011a57600080fd5b82018360208201111561012c57600080fd5b8035906020019184600183028401116401000000008311171561014e57600080fd5b91908080601f01602080910402602001604051908101604052809392919081815260200183838082843760009201919091525092955050913592506102f3915050565b005b610239600480360360208110156101a957600080fd5b8101906020810181356401000000008111156101c457600080fd5b8201836020820111156101d657600080fd5b803590602001918460018302840111640100000000831117156101f857600080fd5b91908080601f0160208091040260200160405190810160405280939291908181526020018383808284376000920191909152509295506103cb945050505050565b60408051918252519081900360200190f35b6101916004803603604081101561026157600080fd5b81019060208101813564010000000081111561027c57600080fd5b82018360208201111561028e57600080fd5b803590602001918460018302840111640100000000831117156102b057600080fd5b91908080601f0160208091040260200160405190810160405280939291908181526020018383808284376000920191909152509295505091359250610432915050565b806000846040518082805190602001908083835b602083106103265780518252601f199092019160209182019101610307565b51815160209384036101000a60001901801990921691161790529201948552506040519384900381018420805495909503909455505083518392600092869290918291908401908083835b602083106103905780518252601f199092019160209182019101610371565b51815160209384036101000a600019018019909216911617905292019485525060405193849003019092208054939093019092555050505050565b600080826040518082805190602001908083835b602083106103fe5780518252601f1990920191602091820191016103df565b51815160209384036101000a6000190180199092169116179052920194855250604051938490030190922054949350505050565b806000836040518082805190602001908083835b602083106104655780518252601f199092019160209182019101610446565b51815160209384036101000a60001901801990921691161790529201948552506040519384900301909220929092555050505056fea265627a7a7231582085b12c7cb67f4952de46b525e7b167f6740ffb97814392b305af2d550f56472e64736f6c634300050f0032",
    "linkReferences": {},
    "deployedLinkReferences": {}
  }
```
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
# Report examples

# Some References
"Architecture -", Hyperledger Caliper, 2020. [Online]. Available: https://hyperledger.github.io/caliper/v0.3/architecture/. [Accessed: 14- April- 2020].

"Writing Adapters", Hyperledger Caliper, 2020. [Online]. Available: https://hyperledger.github.io/caliper/v0.3/writing-adaptors/. [Accessed: 14- April- 2020].

"Rate Controllers", Hyperledger Caliper, 2020. [Online]. Available: https://hyperledger.github.io/caliper/v0.3/rate-controllers/. [Accessed: 14- April- 2020].

"Etherum Configuration", Hyperledger Caliper, 2020. [Online]. Available: https://hyperledger.github.io/caliper/v0.3/ethereum-config/. [Accessed: 14- April- 2020].
