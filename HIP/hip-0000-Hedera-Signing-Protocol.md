- hip: XX
- title: Hedera Signing Protocol
- author: [0xJepsen](https://github.com/0xJepsen), [rocketmay](https://github.com/rocketmay)
- type: Standards Track
- category: Application
- status: Draft 
- created: <07/03/21>
- discussions-to: <https://github.com/hashgraph/hedera-improvement-proposal/discussions/98>
- updated: <07/09/21> <07/08/21> <07/03/21>
- requires: <HIP number(s)>
- replaces: <HIP number(s)>
- superseded-by: <HIP number(s)>

## Abstract

This specification proposes a protocol for web applications to send pregenerated unsigned transactions to client applications for users to sign, tentatively titled the Hedera Signing Protocol. 

This protocol describes the interactions of two actors:

The Requesting Website : The web application providing a service. This application is not controlled by the user, but is providing a service which interacts with the user's Hedera account in some fashion. An example would be Uniswap for the Ethereum blockchain.

The Client : The client application, henceforth referred to as the Client. This application is controlled by the user and has access to a Hedera Account Private Key, which is used to sign the transactions for the services provided by the Requesting Website. An example would be the Metamask wallet for the Ethereum blockchain.

This proposal does not require any new Hedera API endpoints. In addition, this protocol is designed to function without making any calls to the Hedera API. Any API calls are done outside the scope of this protocol.

This proposal introduces a javascript module used to send and receive and sign transaction data before they are submitted to the Hedera public network. The module will be open source and installable as a public node module for ease of inclusion into applications. 

## Motivation

Developers are currently creating web authentication mechanisms from scratch for Hedera-based web apps. This is limits consumer adoption and results in a poor user and developer experience. A standard protocol for decentralized applications to communicate with clients and allow clients to sign transactions would significantly improve the developer and user experience. The Hedera development space is still young, with many new projects developing promising ideas. These projects will need a standardized method of communication, much like what exists in the Ethereum space.

Taking the Metamask wallet as an example, Metamask offers a seamless way for users to interact with dApps, sending requests and signing transactions without compromising the security of their accounts. Another example is Wallet Connect, which is an open protocol for signing ETH transactions.

Without a communication standard, projects in the space are required to reinvent the wheel for every application and wallet. This adds overhead for the project team. This also represents a significant risk to the user who must reveal sensitive account information to web applications in order to interact with the Hedera network.

## Rationale

We propose establishing a standard protocol for Requesting Websites to present Clients with transactions to be signed by the User. The module allows the Requesting Website to present transaction data to the DOM environment where it can be viewed by the Client. There is no sensitive account information within the transaction data. The Client recieves the transaction data and signs it using their private key without exposing the key to the Requesting Website. The signed transaction is then submitted to the Hedera API network by the Client and the receipt is sent to the Requesting Website.

This protocol is designed to allow any Client to implement it and be accessed by Requesting Websites, thus providing maximum flexibility for users and developers. It is similar to Wallet Connect, which is an open protocol. Contrast this with Metamask's protocol which is designed specifically for Metamask.


## Specification

JavaScript is the primary language that can communicate with the browser's [DOM](https://www.w3.org/TR/REC-DOM-Level-1/introduction.html#). It is thus necessary for this module to be a JavaScript module. The transaction transmission and signing process is governed by this protocol.

This is the generalized flow:

1. The User sends a request to the Requesting Website for a transaction that they wish to make through the Requesting Website's interface.

_Start of Protocol Scope_

2. The Requesting Website queries the browser for extensions which implement the Hedera Signing Protocol
3. The Requesting Website generates a list of Clients and the Account IDs that the Clients manage
4. The User selects the desired Client/Account pair which they want to perform the transaction.
5. The Requesting Website creates a transaction, freezes it, and JSONifies it for use as txnObject in Step 6.
6. The Requesting Website sends a sendTransaction RPC to the Client, with txnObject as a param. 
7. The Client displays the transaction to the User and prompts the user to Sign the transaction or Cancel.
8. The Client sends the signed transaction back to the Requesting Website
_End of Protocol Scope_

9. The Requesting Website executes the transaction on the Hedera API, displays the result and continues the user experience.

**Notes:**

**Step 2**: Clients that implement the protocol will contain a 'HederaSP' identifier in their manifest.json. The Requesting Website can then query for extensions which have the identifier and populate a list. 

An empty list signifies that there are no compatible browser extensions. It is then up to the Requesting Website if they want to provide options/instructions to the user.

**Step 3**: The Requesting Website sends the RPC "requestAccounts" to each Client identified in step 2.

Each Client implements requestAccounts() which returns an array of Hedera Account IDs (type string).

**Step 4**: The Requesting Website should display the list of Accounts, sorted by Client. As part of this step the Requesting Website may query the network to obtain further information to assist the user in their choice (such as account balance), though this is not strictly part of the protocol.

The User will then select the Account which they want to perform the transaction with.
		
**Step 6**: The Requesting Website calls the Client RPC sendTransaction, which contains a number of parameters.
	
	const txnParams = {
	
	txnObject: 'xxxx' // the frozen transaction object JSONified
	memo: 'string' // An optional string that contains further information about the transaction. Note that this can be different than the memo in the transaction object itself
	}


**Step 7**: The Client receives txnParams through sendTransaction() and displays the data to the user for their approval. Note that displaying the information of the transaction is not part of this protocol, however it is recommended as best practice that the Client unpackages the transaction and displays all the relevant information.
	
If the account ID in txnObject does not match an account ID served by the Client, it sends back an InvalidAccountID error.
If the Client does not implement the specific transaction type that has been sent through, it can return a TransactionNotSupported error to the Requesting Website.
If the Client cancels the transaction a generic TransactionCancelled error is returned.

**Step 9**: The Requesting Website is responsible for executing the transaction, which allows for multi-sig type transactions to be coordinated by the Requesting Website. 

## Backwards Compatibility

This HIP is entirely opt-in and does not modify any existing functionality. It simply provides standards that client applications (such as wallets) and web applications can follow to interact with each other.

## Security Implications

Clients are responsible for locally signing transactions. At no point are private keys ever shared or revealed to the Requesting Website. This is the main purpose of this protocol. Because no sensitive account data is shared, account security through this protocol is maintained.

On the other hand, there are many considerations which developers should take into account when implementing this protocol into their applications:

Nothing can be done about a Requesting Website (intentionally or not) generating an incorrect transaction. A malicious Requesting Website can generate a transaction that is different than what the user is expecting. This protocol assumes that the Client properly unpackages the transactions that it receives and displays the information in a readable, clear manner to the user for their review, and that the User is given accurate information and a clear indication of what action they are approving by signing the transaction.

The permissions schema referenced below in the Open Issues section would provide more robust security for users.

## How to Teach This

Simple examples and guides will be incorporated into the existing Hedera documentation. Additionally, developer advocates can write educational content on the 'How to' of this feature. 

## Reference Implementation

To be developed.

## Rejected Ideas

N/A

## Open Issues

Open issues (not required for implementing the current HIP) allow users of web3 applications to provide partial information to the dApp—for example, implementing a wallet permissions schema and protocol. [EIP2255](https://eips.ethereum.org/EIPS/eip-2255) is a good resource for this.

## References

- [0] https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md
- [1] https://docs.hedera.com/guides/docs/hedera-api
- [3] https://eips.ethereum.org/EIPS/eip-1102#eth_requestaccounts
- [4] https://www.w3.org/TR/REC-DOM-Level-1/introduction.html#
- [5] https://github.com/NoahZinsmeister/web3-react
- [6] https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md
- [6] https://github.com/aragon/use-wallet
- [7] https://docs.metamask.io/guide/rpc-api.html#permissions

## Copyright/license

This document is licensed under the Apache License, Version 2.0 -- see [LICENSE](../LICENSE) or (https://www.apache.org/licenses/LICENSE-2.0)
