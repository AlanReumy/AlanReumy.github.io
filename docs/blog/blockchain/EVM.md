# EVM

## **introduction**

### **account**

There are two practical type of account:

#### **contain**

From the viewport of contain the account:

- External Owned Account (EOA): contain nonce, balance (controlled by private key)
- Contract Account: contain nonce, balance EVM code, account storage (controlled by EVM code)

#### **address**

From the viewport of address the account:

- EOA: private key => public key => (hash) => 160 bits address
- Contract Account: Sender Address + Nonce => (RLP,REC) => 160 bits address

### **transaction**

a transaction is a single-signed instruction.

There are two practical type of transaction:

- Contract creation
- Message call

a field of transaction:

- nonce
- gasPrice
- gasLimit
- to (160 bits address or 0 if contract creation)
- value
- v,r,s
- init or data

### **Message**

Message is data and value passed between two accounts.

two ways to trigger message:

- transaction (EOA -> account)
- EVM code (Contract account -> account)

### **Atomicity and order**

a transaction is a atomic operation. Can't divide or interrupt, and cannot be overlapped.

a transaction is not guaranteed, because miner determine it.

The order between blocks is determined by a consensus algorithm such as PoW.

## **Ethereum Virtual Machine**

The Ethereum Virtual Machine is a runtime for smart contracts in Ethereum.
