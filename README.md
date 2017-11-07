# Ethereum voting DApp

- set up your dev environment, 
- coded a simple contract, 
- compiled and deployed the contract on the blockchain (private test network)
- interacted with it via nodejs console 
- interacted with it through a webpage. 

## ethereumjs-testrpc

- Node Ethereum client (testing + devlepment)
- in-memory (test) blockchain
- includes all popular RPC features
- works as web3.js provider
- creates 10 test accounts to play with automatically. 
- These accounts come preloaded with 100 (fake) ethers.
- Gas Price default: 2000000000
- Gas Limit default: 9000 (limit < price)
- Tx price default: 4700000

        npm run testrpc

- runs in background and prints any new transactions, even if turn app off.

## DApp

### Via CLI

- even if stop Node cli process, testrpc server will run in BG in memory

        node
        Web3 = require('web3')
        web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
- initialize the solc and web3 objects

        web3.eth.accounts
- make sure web3 object is initialized and can communicate with the blockchain
- ^ queries all the accounts in the blockchain
- by default all are unlocked

        code = fs.readFileSync('Voting.sol').toString()
        solc = require('solc')
        compiledCode = solc.compile(code)
- compile the contract

        compiledCode
        compiledCode.contracts[':Voting'].bytecode
- compiled source code. code deployed to blockchain

        compiledCode.contracts[':Voting'].interface
- interface/template of contract. called ABI (application binary interface)
- which methods are available to contract user

        abiDefinition = JSON.parse(compiledCode.contracts[':Voting'].interface)
        VotingContract = web3.eth.contract(abiDefinition)
- create contract object, used to depoy + initiate contracts in the blockchain

        byteCode = compiledCode.contracts[':Voting'].bytecode
        deployedContract = VotingContract.new(['Rama','Nick','Jose'],{data: byteCode, from: web3.eth.accounts[0], gas: 11})

- ^ deploys contract to blockchain
- intitialised with candidates for election (via contracutor)
- data - compiled bytecode we will deploy to blockchain
- from - blockchain keeps track of who deployed contract (to blockchain). Here its our first account (now the owner). In live blockchain u must own account (locked via passphrase during creation, unlocked before transaction)
- gas - cost of interacting with blockcian, goes to miners who do work to include code in blockchain. specify how much willing to pay to get code in the blockchain, by setting value of gas. uses ether balance to buy. price set by network 
  
         deployedContract.address

- used to identify contract in that blockchain, via address

        contractInstance = VotingContract.at(deployedContract.address)
- instance of contract for us to interact with contract
- 100k's deployed on blockchain

        contractInstance.totalVotesFor.call('Rama')
- use instance and ABI definition to interact with contract

        contractInstance.voteForCandidate('Rama', {from: web3.eth.accounts[0]})
- get new transaction ID each time. can refer for it anytime. Immutable.

        contractInstance.voteForCandidate('Rama', {from: web3.eth.accounts[0]})
        contractInstance.totalVotesFor.call('Rama').toLocaleString()

### Interact with Blockchain

- load of RPC methods implemented in `ethereumjs-testrpc`.

        web3.eth.gasPrice.c

        web3.eth.blockNumber
- each time create transaction creates new block. starts at 0

        web3.eth.getBlock(1)
- all details of a block

        web3.eth.getBlockTransactionCount(3)
- all blocks seem to have 1 transaction ??. perhaps change gas price/limit.

        web3.eth.getTransaction("0x7673ee6ba39f7f1835cd3bff618678e79968f020e7ff7a85f0be8c92f73a1f7d")
- get a transactions details

        web3.eth.getBalance("0xceb9b1a3528bd83972859cf44f5240f9cd6d133e")
- account balance

### Via web UI

- need ABI and address to interact with contract