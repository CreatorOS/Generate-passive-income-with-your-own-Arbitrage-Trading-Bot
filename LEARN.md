# Generate passive income with your own Arbitrage Trading Bot

Would you like to make some money while you sleep? Of course yes! Why wouldn't you? In this quest we will create our own program which will monitor for opportunities to make passive income. But before that lets understand how these opportunities can be found in crypto?

## Arbitrage Trading

As you might already know, crypto currencies or ERC20 Tokens built on top of Ethereum can be exchanged/swapped at certain exchange rates. Every exchange transaction includes a transaction fee or gas fee. The platforms which enable users to swap crypto-currencies/tokens are called Decentralized Exchanges or DEX. Some examples are [Uniswap](https://app.uniswap.org/#/swap), [Sushiswap](https://app.sushi.com/swap) etc. We have centralized exchanges like [Coinbase](https://www.coinbase.com/) as well.

Whenever the exchange rates for a given pair of tokens differ considerably on any 2 DEXes, there is an opportunity to buy at a cheaper rate from one DEX and sell at a higher rate on another. This is where we generate profits or make automatic money and it's known as arbitrage. But it is humanly pretty challenging to keep monitoring the rate changes and evaluate different combinations of trades for profitability. This is exactly where we can leverage code :) . All we need to do is : create a program/bot which monitors changes in exchange rates of any pair of tokens across different exchanges and decide if we can buy cheaper and sell higher to generate profits, after including the transaction gas fee.

Lets understand this with an example. Go to uniswap, select any 2 tokens, in this case, MKR and DAI have been chosen. In another tab, go to sushiswap and select the same 2 tokens but in opposite exchange order. Now try to find an arbitrage opportunity by changing the quantity of 2nd Token, in this case DAI. Begin with 1 DAI then change to 2 DAI, then change to any other amount like 10 DAI in this example, until you see that you can buy DAI for lower amount of MKR on uniswap and sell same quantity of DAI on sushiswap to get a higher amount of MKR. Here is example with screenshots.:

1. On uniswap, we can buy 10 DAI tokens with 0.00263804 MKR tokens.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/0ad2f605-c775-42dc-bdc0-b845bd76b935.jpg)

1. Next, on sushiswap, we can sell 10 DAI tokens to get 0.00276025 MKR tokens. Thus we earn a profit of 0.00276025 - 0.00263804 = 0.00012221 MKR tokens. ![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/1433ef06-42b9-4b9a-894f-bf00b1d6c2d1.jpg)

However, if we keep swapping MKR for DAI on uniswap again and again, over a period of time, uniswap will have larger quantities of MKR and lower quantities of DAI. This will create a huge supply of MKR but poor supply of DAI. So DAI will keep getting costlier and the exchange rate will change and the same arbitrage opportunity will cease to exist. This automatic process of deciding exchange rates based on token availability (also known as liquidity) is known as automatic market making (AMM). But why care about AMM? Well, it's important to know that exchange rates for a given swap will change depending on the number of tokens available in the pool and also the number of tokens we want to swap. Want to see a live example? Go to uniswap([https://app.uniswap.org/\#/swap](https://app.uniswap.org/#/swap)) and observe the exchange rate for MKRDAI by changing MKR amount to 1, 10, 100, 1000:

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/509cc514-3008-4bae-818e-d13b5ae4ff07.jpg)![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/6e509065-9b8d-42ac-8659-300fe56dcb5c.jpg)

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/9fa285cf-d66b-41c0-9746-34845f5138bb.jpg)![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/5a24317b-b47c-4b8d-a7a1-f82aee3a59f7.jpg)

Observe how DAI keeps getting costlier as you increase the input MKR amount. Now imagine there are other traders too who have placed higher swap amounts for the MKRDAI pair. Due to their higher trade orders and availability or unavailability of tokens in the pool, the exchange price is going to fluctuate. Thus a trade order placed at a rate R1 might end up executing at a rate R2 depending on the circumstances. This difference is called slippage. Why do we care about slippage? When evaluating the possibility of a profitable arbitrage, we should consider slippage prices so that we can be sure that even if exchange rates fluctuate, our profits are intact.

Awesome! Now that you understand how exchange rates work, let's code the arbitrage bot!

## The tools involved

Since we are just going to monitor prices, we can use javascript to execute the price fetching logic in a loop. In order to get exchange rates and swap tokens, we will need to interact with the smart contracts of uniswap and sushiswap via javascript code. Introducing ethers.js ([https://docs.ethers.io/v5/](https://docs.ethers.io/v5/)) which will enable us to invoke functions on the smart contracts in js code and help us fetch the exchange rates and also execute the token swaps.

But wait, what if our bot has bugs? How can we be sure that we are actually making the profits? Is there a way we can test the bot without making real money transactions? And the answer is yes! We can achieve this using a tool called HardHat ([https://hardhat.org/getting-started/](https://hardhat.org/getting-started/)) and an archival node called Infura.

You must be wondering what is an archival node and why do we need it ? Any blockchain is a network of nodes. Ethereum's main network has [3 types of nodes](https://ethereum.org/en/developers/docs/nodes-and-clients/#node-types): full nodes, light nodes and archive nodes.A full node stores the state of the most recent 128 blocks (or only 64 if you’re using fast/snap sync) and about one week of trace data(this would be in GB.) updated as soon as the new block comes in. This is so because full nodes help verify transaction and validity of the blockchain state. A light node contains the header chain and requests everything else.So querying for data which needs larger historical context is not readily possible for a full node or light node. Here is an example of one such query: Getting eth balance of one of the earliest accounts needs historical records of all transactions that happened on it till now.

This is where Archive nodes help. They are full nodes running with a special option known as "archive mode". Archive nodes have all the historical data of the blockchain since the 1st transactions.This would be in TBs. These nodes can be used to test our smart contracts with any historical state of the network quickly and reliably. But since the amount of data in these nodes is humongous we will not be downloading the network in our local machines, instead we will be using online services which provide us access to archival nodes via cloud. Infura is one such archival node service.

Using Hardhat and Infura, we can deploy and test our code on a local copy of any blockchain network. Hardhat also provides us with test accounts (with large ether balance) and passwords so that we can use them to test our transactions. You can take up the identity of these accounts or in fact any account on your local fork of the blockchain network using HardHat. This is called impersonation. What this means is, while I dont have any money in my real account, I can take up the identity of any account and execute my transactions as if I were them! You could become Vitalik too! See his account details: [https://etherscan.io/address/0xab5801a7d398351b8be11c439e05c5b3259aec9b](https://etherscan.io/address/0xab5801a7d398351b8be11c439e05c5b3259aec9b)

## Connecting to Infura

Go to [https://infura.io/](https://infura.io/) and get started to create your free account. Create an Ethereum project and go to its Settings tab. Copy the endpoint url of the network you want to test on (in our case, mainnet), by selecting it from the drop down.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/4bee83a7-32eb-40a5-bace-b53747f2c988.jpg)

## Fork the mainnet

You can create a local blockchain of your own by copying the latest state of any blockchain node using Hardhat; this is called forking. This blockchain node (also called the fork of the original node) will run as a server on your localhost. In a new terminal, you can create the mainnet fork using following command:

```

npx hardhat node --fork https://mainnet.infura.io/v3/<key>

```

Observe how it creates a server on the local host at 8545 port on top of the fork created from mainnet. It also displays list of accounts,their balances and private keys so that we can impersonate them:

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/64f8c19c-a161-455c-8fe0-93bd7bde9769.jpg)

Please note that when using a free account on any archival node service, there is an upper limit after which the node will throw an error unless reinstated. Example of such error you might face when the limit is reached:

ProviderError: project ID does not have access to archive state

Solution is to restart the local server by re-executing this:

```

npx hardhat node --fork https://mainnet.infura.io/v3/<key>

```

## Initiate the project

1.Create a folder for this project. Navigate to the project path in the terminal and run the following command to initialize the project with node package manager(npm). Npm helps us manage all the third party libraries/dependencies we need to use in our code. Initializing the project this way will create a package.json file which will store the information needed by npm to install dependencies of our project:

```

npm init -y


```

2.Add hardhat as development dependency:

```

sudo npm install -D hardhat

```

3.npx is a utility from node.js which helps create local project installations. Create empty config using:

```

npx hardhat

```

Press down to select “create an empty hardhat.config.js”

This config is used by Hardhat to create connection to the blockchain server using the values we populate in the ‘networks’ key. Populate it with a url for the local blockchain node fork we created from mainnet in previous subquest.

```

const LOCAL_URL = "http://127.0.0.1:8545/";

const PRIVATE_KEY = '';

module.exports = {

  solidity: "0.7.3",

  networks: {

    local: {

      url: LOCAL_URL,

      accounts: \[PRIVATE_KEY\]

    }

  }

};

```

4.Observe new files created in your project. An auto generated file (package.json) is used to keep track of all the external libraries we would be using, Populate package.json with this, just adding 2 dependencies: ethers and uniswap-sdk:

```

{

  "name": "arbitrageQuest",

  "version": "1.0.0",

  "description": "",

  "main": "arbitrageQuest.js",

  "scripts": {

    "test": "echo \\"Error: no test specified\\" && exit 1"

  },

  "keywords": \[\],

  "author": "",

  "license": "ISC",

  "dependencies": {

    "@uniswap/sdk": "^3.0.3-beta.1",

    "ethers": "^5.0.8"

  },

  "devDependencies": {

    "hardhat": "^2.6.1"

  }

}

```

5.Install the dependencies with following command in terminal:

```

npm install

```

## Whats Next

We will code the bot step by step and execute to earn profits!
