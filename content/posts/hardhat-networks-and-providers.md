+++
title = "Hardhat: networks and providers"
date = 2022-11-11
+++

_Originally published at [HackMD](https://hackmd.io/@fvictorio/hardhat-networks-and-providers)._

An Ethereum node can be used to read the state of the blockchain or to send transactions that modify that state. To let you do this, Ethereum clients implement a protocol called [JSON-RPC](https://en.wikipedia.org/wiki/JSON-RPC). This allows applications to rely on a (somewhat) standardized set of methods that should work with any client.

## Sending a JSON-RPC call

We'll use [Alchemy](https://www.alchemy.com/) in these examples and, if you want to reproduce them, you'll need to get an API key from them. Even better, if you have a locally synced node, you can configure it to expose the JSON-RPC interface and use that instead.

Let's say we want to get the latest block number of the Goerli network. To do that, we have to use the `eth_blockNumber` JSON-RPC method, which returns the current block number. We can send this method via curl, or any other HTTP client:

```
$ curl -X POST -H 'Content-Type: application/json' \
--data '{"jsonrpc":"2.0", "id": "1", "method":"eth_blockNumber", "params":[]}' \
https://eth-goerli.alchemyapi.io/v2/<API_KEY>

{"jsonrpc": "2.0", "id": "1", "result": "0x8a73e2"}
```

This is a mouthful, but there are only a few crucial parts; the rest is just boilerplate:

- We are sending an HTTP POST request to the Alchemy endpoint.
- The body of the request is a JSON object. This object includes the name of the method we are calling, `eth_blockNumber`. It also has a list of params, which is empty in this case.
- We receive a JSON object as the response, which includes a `result` field with the value we are interested in: the current block number, as a hexadecimal string.

Other actions use other JSON-RPC methods. To send a signed transaction you use `eth_sendRawTransaction`, to get a transaction receipt you use `eth_getTransactionReceipt`, and so on.

All of this is meant to illustrate what happens under the hood each time you interact with a node. But we don't want to do something like that for everything.

## JSON-RPC networks in Hardhat

You can interact with nodes via JSON-RPC more easily using [Hardhat](https://hardhat.org/). We'll see how to configure a network and then send calls to it.

Before starting, go to an empty directory and install Hardhat:

```
$ cd /path/to/some/empty/directory
$ npm install --save-dev hardhat
```

Now create a `hardhat.config.js` file with this content:

```js
module.exports = {
  networks: {
    goerli: {
      url: "https://eth-goerli.alchemyapi.io/v2/<API_KEY>"
    }
  }
}
```

This is adding a network to the Hardhat configuration. To configure a network you need at least two things: a name, which in this case is `goerli`, and a URL. We could've used a different name here, like `testnet`, if we wanted.

After setting this up, you can start a console connected to this network by running `npx hardhat console --network goerli`, or `hh console --network goerli` if you have installed the [Hardhat shorthand](https://hardhat.org/guides/shorthand.html). A node.js REPL will be started, where you can send JSON-RPC calls more easily:

```
> await network.provider.send("eth_blockNumber", [])
'0x8a733e'
```

Here, `network.provider` is an object that implements the [EIP-1193](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1193.md) standard. This is an abstraction with a minimal API used to interact with a node via JSON-RPC. When you use `window.ethereum` in a dapp, you are using an instance of a EIP-1193 provider exposed by the browser wallet.

The only part we care about here is that this object has a `send` function that receives the name of the method and a list of arguments.

_Aside: the correct way to make calls in EIP-1193 is with the `request` method, but here we'll use the old `send` method because it's more concise._

You can have more than one network in your configuration:

```js
module.exports = {
  networks: {
    mainnet: {
      url: "https://eth-mainnet.alchemyapi.io/v2/<API_KEY>"
    },
    goerli: {
      url: "https://eth-goerli.alchemyapi.io/v2/<API_KEY>"
    }
  }
}
```

But you can only connect to one of them at a time. If we exit the console and run `hh console --network mainnet`, we can make the same call as before but to the mainnet network instead:

```
> await network.provider.send("eth_blockNumber")
'0xc607e5'
```

## Local development node

Using a testnet for local development is slow, and you need to get test ether from a faucet. An easier and faster alternative is to use the local development node that comes with Hardhat. This node starts an instance of the Hardhat Network, which includes features like `console.log` and Solidity stack traces.

If you run `hh node` in a terminal, an HTTP server with this development node will start listening for requests in `http://localhost:8545`:

```
$ hh node
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========
Account #0: 0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

...
```

You can now connect to this network from another terminal running `hh console --network localhost` and make the same call as before:

```
> await network.provider.send("eth_blockNumber")
'0x0'
```

The response is `0x0` here because this is a new network that hasn't received any transactions.

But wait, what is this `localhost` network? We haven't added it to our config. The reason we can use it is that Hardhat comes with this pre-defined network, and it's pretty much equivalent to adding this:

```
localhost: {
  url: "http://localhost:8545"
}
```

Since connecting to this particular address and port is so common, Hardhat includes it by default. But you can explicitly add it to your config to customize it.

So now we have an easy way to develop locally. For example, if we want to run our tests, we can start a node in one terminal and then run `hh test --network localhost` in another. But this is very cumbersome. It means working with two different terminals and killing and starting the node again and again every time we want to start from scratch.

Luckily, you don't need to do that. Besides `localhost`, Hardhat comes with another pre-defined network: `hardhat`. This is the same network used by the development node, but instead of starting an HTTP server, it's an in-process network that is created when you run your task, and killed at the end of it. So if you run `hh test --network hardhat`, the result will be pretty much equivalent to the combination of starting a new node, running `hh test --network localhost`, and then killing the node, all in one command.

Besides, this is the default network, so you only need to do `hh test`.

## Hardhat Network

This `hardhat` network is different from the other networks we have configured so far, because no HTTP server is started at all. Instead, the provider that will be exposed is connected to an in-process, ephemeral instance of the Hardhat Network. This might surprise you if you thought that you could only interact with JSON-RPC via HTTP, but the protocol is actually transport-agnostic.

The [configuration options for the Hardhat Network](https://hardhat.org/hardhat-network/reference/#config) are also different from the options for the [external networks](https://hardhat.org/config/#json-rpc-based-networks). For example, external networks have a `url` configuration field, but the Hardhat network configuration doesn't. On the other hand, the Hardhat network has a `blockGasLimit` option that doesn't exist for other networks.

Keep in mind that the `hardhat` entry of your configuration is used to configure both the in-process network used by default, and the network started when you run `hh node`.

## Ethers.js

Technically, you can do anything you want just with a JSON-RPC provider, but this is still a very low-level interface. Libraries like [ethers.js](https://docs.ethers.io/v5/) offer a higher-level functionality to make your life easier.

One of these interfaces is a set of [providers](https://docs.ethers.io/v5/single-page/#/v5/api/providers/). These are like the EIP-1193 provider we've been used so far, but with many helper methods. For example, we previously called the `eth_blockNumber` method and got a hexadecimal string in response. With an ethers provider, you would do this instead:

```
> await anEthersProvider.getBlockNumber()
9073582
```

There are two significant differences here: we use a specific and more readable method to get the current block number, and we get a number in response, not a hexadecimal string.

How do we convert our provider into an ethers.js provider? The easiest way is to use ethers.js's [`Web3Provider`](https://docs.ethers.io/v5/single-page/#/v5/api/providers/other/-%23-Web3Provider) to wrap our existing provider. If you have ethers.js installed, you can start an `hh console` and execute this:

```
> const ethers = require("ethers")
undefined

> const anEthersProvider = new ethers.providers.Web3Provider(network.provider)
undefined

> await anEthersProvider.getBlockNumber()
9073591
```

Even easier, you can use the [`@nomiclabs/hardhat-ethers`](https://hardhat.org/plugins/nomiclabs-hardhat-ethers.html) plugin. This plugin will add an ethers property to the Hardhat runtime environment, which has all the functionality from the ethers package, but that also includes some extra things, like an already initialized provider. So if you install the plugin, import it in your config, and then start a Hardhat console again, you'll be able to do this:

```
> await ethers.provider.getBlockNumber()
9073599
```

## Learn more

- If you want to know more about the JSON-RPC interface of Ethereum nodes, you can check [the ethereum.org page](https://ethereum.org/en/developers/docs/apis/json-rpc/). There is work in progress to have an up to date specification in the [eth1.0-apis](https://github.com/ethereum/eth1.0-apis) repository, which is used to generate [this site](https://playground.open-rpc.org/?schemaUrl=https://raw.githubusercontent.com/ethereum/eth1.0-apis/assembled-spec/openrpc.json&uiSchema%5BappBar%5D%5Bui:splitView%5D=false&uiSchema%5BappBar%5D%5Bui:input%5D=false&uiSchema%5BappBar%5D%5Bui:examplesDropdown%5D=false). Finally, this information used to live in the [Ethereum Wiki](https://eth.wiki/json-rpc/API) and, while this page isn't maintained anymore, it still has useful information.
- To learn more about the Hardhat Network, check [its docs](https://hardhat.org/hardhat-network/).
