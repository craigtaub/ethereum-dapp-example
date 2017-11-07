# Ethereum voting DApp

- set up your dev environment, 
- coded a simple contract, 
- compiled and deployed the contract on the blockchain (private test network)
- interacted with it via nodejs console 
- interacted with it through a webpage. 

- JS API: https://github.com/ethereum/wiki/wiki/JavaScript-API

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

### Create Smart Contract

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


### Basic transaction

        gasPrice=1000000000000000000
        gasLimit=900000
- start: 

        web3.fromWei(web3.eth.gasPrice)
- 1 ETH.

        web3.fromWei(web3.eth.getBalance(web3.eth.accounts[0]))
        web3.fromWei(web3.eth.getBalance(web3.eth.accounts[1]))
- 100 ETH each.

        web3.eth.sendTransaction({
                from: web3.eth.accounts[0],
                to: web3.eth.accounts[1],
                value: web3.toWei(1), // expects in wei
        });
- send 1 ETH

        web3.fromWei(web3.eth.getBalance(web3.eth.accounts[0]))
- 99 ETH, repeat, 98 ETH

        web3.fromWei(web3.eth.getBalance(web3.eth.accounts[1]))
- 101 ETH, repeat, 102 ETH

        web3.eth.getTransaction("X")
- gas: 110000. provided by sender (me) in sendTransaction OR defaults to gasLimit

        web3.eth.getTransactionReceipt("X")
- costs 2100 to send TX (gasUsed+cumulativeGasUsed)

        web3.fromWei(web3.eth.getTransaction("X").value)
- Confirm it sent 1 ETH

#### Many TX on 1 block

        --blocktime=10
- sets when it will mine TX's. Turns auto-mining on which creates new block (even if no TX). 
- and limit obv.

### Smart Contract

        gasPrice=1000000000000000000 
        gasLimit=900000
- start

        VotingContract.new, gas: 500000 * 1000000000000000000

- costs 500000 for each transaction. contract gas price. used for calc of tx price.

        web3.fromWei(web3.eth.getBalance(web3.eth.accounts[0]))
- 100 ETH, cost couple Wei to launch contract 

        contractInstance.voteForCandidate('Rama', {from: web3.eth.accounts[0]})
- voted 

        web3.eth.getBalance(web3.eth.accounts[0])
- cost 43000 wei. 

        web3.eth.getTransaction("0x07e5ccd1bfb6c4b77a822c9239455f4154384901308215bc8fb25d74e6524c46")
- value: { [String: '0'] s: 1, e: 0, c: [ 0 ] }, -> always same
- gas: 90000, -> limit, always the same
- gasPrice: { [String: '1'] s: 1, e: 0, c: [ 1 ] }, -> always same
  
        contractInstance.voteForCandidate('Rama', {from: web3.eth.accounts[0]})
- voted again

        web3.eth.getTransactionReceipt("0x07e5ccd1bfb6c4b77a822c9239455f4154384901308215bc8fb25d74e6524c46")
- gasUsed+cumulativeGasUsed: 43402, -> confirms cost

        web3.eth.getTransaction("0x4776f1b595508be856b43cfaf41e3efe2f3a599d50cc827b64f0abf3368afce4")
        web3.eth.getTransactionReceipt("0x4776f1b595508be856b43cfaf41e3efe2f3a599d50cc827b64f0abf3368afce4")
- gasUsed+cumulativeGasUsed: 28402,

### Via web UI

- need ABI and address to interact with contract
