# Cross-Chain-Messaging-Between-Testnets-Using-WormHole-and-Remix
Getting Started with Wormhole and Remix: A Quick Guide to Cross-Chain Messaging


## Preparation
For this example, we will use Sepolia Ethereum and BNB testnets

### Ethereum Wallet
First, make sure you have a non-custodial Ethereum-compatible wallet like MetaMask, Trust Wallet, or a similar option.

### Add Testnet Networks to Wallet
For this example, we will use the Celo and BNB testnets. Let's prepare your wallet to use them. Go to each one of the below links, connect your wallet, and add the chains:

[Sepolia Testnet](https://chainlist.org/chain/11155111)

[BNB Testnet](https://chainlist.org/chain/97)

### Get Tokens from Faucets
Finally, get tokens from the testnet faucets to fund your transactions:

[Sepolia Testnet Faucet](https://www.alchemy.com/faucets/ethereum-sepolia)

[BNB Testnet Faucet](https://www.bnbchain.org/en/testnet-faucet)


## Deploying Smart Contracts

### Step 1: Open Remix IDE
Navigate to Remix IDE in your browser.
### Step 2: Create a New File
In Remix, click on the "File Explorers" tab.

Click on the "📄" button to create a new file.

Name the file HelloWormhole.sol.

Create file HelloWormhole.sol.

<img width="498" alt="create file picture" src="https://github.com/Osayeme/Cross-Chain-Messaging-Between-Testnets-Using-WormHole-and-Remix/blob/main/assets/newfile.png">




### Step 3: Paste the Smart Contract Code
Open the newly created file and paste your smart contract code into it.
``` solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "wormhole-solidity-sdk/interfaces/IWormholeRelayer.sol";
import "wormhole-solidity-sdk/interfaces/IWormholeReceiver.sol";

contract HelloWormhole is IWormholeReceiver {
    event GreetingReceived(string greeting, uint16 senderChain, address sender, address senderContract);

    uint256 constant GAS_LIMIT = 50_000;

    IWormholeRelayer public immutable wormholeRelayer;

    string public latestGreeting;

    constructor(address _wormholeRelayer) {
        wormholeRelayer = IWormholeRelayer(_wormholeRelayer);
    }

    function quoteCrossChainGreeting(
        uint16 targetChain
    ) public view returns (uint256 cost) {
        (cost, ) = wormholeRelayer.quoteEVMDeliveryPrice(
            targetChain,
            0,
            GAS_LIMIT
        );
    }

    function sendCrossChainGreeting(
        uint16 targetChain,
        address targetAddress,
        string memory greeting
    ) public payable {
        uint256 cost = quoteCrossChainGreeting(targetChain);
        require(msg.value == cost);
        wormholeRelayer.sendPayloadToEvm{value: cost}(
            targetChain,
            targetAddress,
            abi.encode(greeting, msg.sender), // payload
            0, // no receiver value needed since we're just passing a message
            GAS_LIMIT
        );
    }

    function receiveWormholeMessages(
        bytes memory payload,
        bytes[] memory, // additionalVaas
        bytes32 senderContractBytes, // address that called 'sendPayloadToEvm' (HelloWormhole contract address)
        uint16 sourceChain,
        bytes32 // unique identifier of delivery
    ) public payable override {
        require(msg.sender == address(wormholeRelayer), "Only relayer allowed");

        // Parse the payload and do the corresponding actions!
        (string memory greeting, address sender) = abi.decode(
            payload,
            (string, address)
        );
        latestGreeting = greeting;
        address senderContract = address(uint160(uint256(senderContractBytes)));
        emit GreetingReceived(latestGreeting, sourceChain, sender, senderContract);
    }
}```

Step 4: Set Up Compiler and Optimization
Go to the "Solidity Compiler" tab on the left.

Select the compiler version to 0.8.13

Select “Advanced Configurations” and Enable "Optimization".

Click on the "Compile HelloWormhole.sol" button.

Compile file HelloWormhole.sol
