# solc-plugin-verify-proxy
[![NPM Version](https://img.shields.io/npm/v/solc-plugin-verify-proxy.svg)](https://www.npmjs.com/package/solc-plugin-verify-proxy)
[![NPM Monthly Downloads](https://img.shields.io/npm/dm/solc-plugin-verify-proxy.svg)](https://www.npmjs.com/package/solc-plugin-verify-proxy)

This plugin allows you to automatically verify your smart contracts' source code (compiled with solc) on Etherscan, straight from nodejs.

**Note:** This version of the plugin uses **multi-file verification**. If you want to use source code flattening instead for any reason, please use the [legacy version (v0.4.x)](https://github.com/rkalis/truffle-plugin-verify/tree/legacy) of the plugin.

## Installation / Preparation
1. Install the plugin with npm or yarn
    ```sh
    npm install solc-plugin-verify-proxy
    yarn add solc-plugin-verify-proxy
    ```

2. Generate an API Key on your Etherscan account (see the [Etherscan website](https://etherscan.io/apis))
3. Add your Etherscan API key and other configuration to your config (make sure to use something like `dotenv` so you don't commit the api key)

    ```js
    let config = {
        api_keys: {
            etherscan: 'MY_API_KEY'
        }
    }
    ```

## Usage
Before running verification, make sure that you have successfully deployed your contracts to a public network with Web3 and solc. The contract deployment must have completely finished without errors.

After deployment, connect the contract that you wish to verify and corresponding deployment address with `@`, and set the following `config` data:

```js
let config = {
    api_keys: {
        etherscan: 'MY_API_KEY'
    },
    networks: {
        rinkeby: {
            provider: () => new Web3.providers.HttpProvider('END_POINT'),
            network_id: 4,
            skipDryRun: true
        },
        bsc: {
            provider: () => new Web3.providers.HttpProvider('END_POINT'),
            network_id: 56,
            skipDryRun: true
        }
    },
    network: 'rinkeby',
    working_directory: 'ABSOLUTE_PROJECT_ROOT_PATH',
    contracts_build_directory: 'ABSOLUTE_PROJECT_ROOT_PATH/build',
    contract_name_address_pairs: ['CONTRACT_NAME@DEPLOYMENT_ADDRESS'],
    debug: true
}
```

The network parameter should correspond to a network defined in the `config` data, with the correct network id set. The Ethereum mainnet and all main public testnets are supported.

For example, if we defined `rinkeby` as network in `config` data, and we wish to verify the `SimpleStorage` contract:

```
const verify_plugin = require('solc-plugin-verify-proxy/verify')

let config = {
    api_keys: {
        etherscan: 'MY_API_KEY'
    },
    networks: {
        rinkeby: {
            provider: () => new Web3.providers.HttpProvider('END_POINT'),
            network_id: 4,
            skipDryRun: true
        }
    },
    network: 'rinkeby',
    working_directory: 'ABSOLUTE_PROJECT_ROOT_PATH',
    contracts_build_directory: 'ABSOLUTE_PROJECT_ROOT_PATH/build',
    contract_name_address_pairs: ['SimpleStorage@DEPLOYMENT_ADDRESS']
}

await verify_plugin(config)
```

This can take some time, and will eventually either return `Pass - Verified` or `Fail - Unable to verify` for each contract. Since the information we get from the Etherscan API is quite limited, it is currently impossible to retrieve any more information on verification failure. There should be no reason though why the verification should fail if the usage is followed correctly.

If you do receive a `Fail - Unable to verify` and you are sure that you followed the instructions correctly, please [open an issue](/issues/new) and I will look into it. Optionally, a `--debug` flag can also be passed into the CLI to output additional debug messages. It is helpful if you run this once before opening an issue and provide the output in your bug report.


### Proxy Contracts
This plugin supports [EIP1967](https://eips.ethereum.org/EIPS/eip-1967) proxies out of the box. If you try to verify a proxy contract (e.g. contracts deployed with OpenZeppelin's `deployProxy`), it will correctly verify the proxy contract and call Etherscan's "proxy verification" so that the proxy contract gets marked as a proxy on Etherscan (enabling Read/Write as Proxy). Note that EIP1967 *Beacon* contracts are not yet supported, and other types of non-standard proxies are also not supported.


### Usage with supported chains
These instructions were written for verification on Etherscan for Ethereum mainnet and testnets, but it also works for verification on other platforms for other chains. To verify your contracts on these chains make sure that your `truffle-config.js` file contains a network config for your preferred network. Also make sure that you request an API key from the platform that you're using and add it to your `truffle-config.js` file. If you want to verify your contracts on multiple chains, please provide separate API keys.

```js
module.exports = {
  /* ... rest of truffle-config */

  api_keys: {
    etherscan: 'MY_API_KEY',
    optimistic_etherscan: 'MY_API_KEY',
    arbiscan: 'MY_API_KEY',
    bscscan: 'MY_API_KEY',
    snowtrace: 'MY_API_KEY',
    polygonscan: 'MY_API_KEY',
    ftmscan: 'MY_API_KEY',
    hecoinfo: 'MY_API_KEY',
    moonscan: 'MY_API_KEY',
    bttcscan: 'MY_API_KEY',
    aurorascan: 'MY_API_KEY',
    cronoscan: 'MY_API_KEY'
  }
}
```

#### All supported platforms & networks
- [Etherscan](https://etherscan.io/) (Ethereum Mainnet & Ropsten, Kovan, Rinkeby, Goerli Testnets)
- [Optimistic Etherscan](https://optimistic.etherscan.io/) (Optimistic Ethereum Mainnet & Kovan Testnet)
- [Arbiscan](https://arbiscan.io) (Arbitrum Mainnet & Rinkeby Testnet)
- [BscScan](https://bscscan.com) (BSC Mainnet & Testnet)
- [Snowtrace](https://snowtrace.io/) (Avalanche Mainnet & Fuji Testnet)
- [PolygonScan](https://polygonscan.com) (Polygon Mainnet & Mumbai Testnet)
- [FtmScan](https://ftmscan.com) (Fantom Mainnet & Testnet)
- [HecoInfo](https://hecoinfo.com) (HECO Mainnet & Testnet)
- [Moonscan](https://moonscan.io/) (Moonbeam, Moonriver Mainnets & Moonbase Alpha TestNet)
- [BTTCScan](https://bttcscan.com/) (BitTorrent Mainnet & Donau Testnet)
- [Aurorascan](https://aurorascan.dev/) (Aurora Mainnet & Testnet)
- [Cronoscan](https://cronoscan.com) (Cronos Mainnet)