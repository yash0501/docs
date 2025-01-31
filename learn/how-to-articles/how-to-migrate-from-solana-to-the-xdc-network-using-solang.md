# Table of content
  - [Overview](#-overview)
  - [Install and setup Dev Environment](#-install-and-setup-dev-enviroment) 
  - [Create a project](#-create-a-project)
  - [Create an XDC Network Wallet](#-create-an-xdc-network-wallet)
  - [Write a smart contract](#-write-a-smart-contract)
  - [Compile and migrate the smart contract from Solana to the XDC Network](#-Compile-and-migrate-the-smart-contract-from-solana-to-the-xdc-network)

# Overview

Solana is a new blockchain focusing on performance. It supports smart contracts like Ethereum which they call Programs. You can develop those in Rust, but there's also a new project now to compile Solidity to Solana. In other words you can deploy your contracts written in Solidity now to Solana!.

### What you will learn

This guide aims at teaching on how a smart contract with solidity for solana using solang. 

### Solang

With [Solang](https://solang.readthedocs.io/en/latest/) you can compile smart contracts written in Solidity for Solana and Parity Substrate. It uses the llvm compiler framework to produce WebAssembly (wasm) or BPF contract code. As result, the output is highly optimized, which saves you in gas costs.

The best way to install Solang is using the [VS Code extension](https://solang.readthedocs.io/en/latest/extension.html). It will automatically install the correct solang binary along with the dependencies. 

Let's do some changes to our solang extension, we need to put the target in the settings of it: 

![solang-target](https://user-images.githubusercontent.com/34518489/195971806-c043bd4e-ec6a-41fc-83ac-62c32a37178a.png)

If you have solidity extension in VSCode Disable the workspace: 

![disable-solidity](https://user-images.githubusercontent.com/34518489/195971874-c49e3a0e-0a41-4cd6-8cd1-ac76d13cba62.png)

# Install and setup Dev Environment

For this guide I will be usuing `Windows`

There are a few technical requirements before we start. Please install the following:

- [Node.js v8+ LTS and npm](https://nodejs.org/en/) (comes with Node)
- [Git](https://git-scm.com/)
- [Yarn](https://yarnpkg.com/) (optional but I will use this one)
- [Solang](https://solang.readthedocs.io/en/latest/)

Once we have those installed, we only need one command to install Truffle:

```bash
npm install -g truffle
```

To verify that Truffle is installed properly, type **`truffle version`** on a terminal. You should see something like:

```bash
Truffle v5.5.27 (core: 5.5.27)
Ganache v7.4.0
Solidity v0.5.16 (solc-js)
Node v16.16.0
Web3.js v1.7.4
```

If you see an error instead, make sure that your npm modules are added to your path.

### For Windows

We need to install:

[Solana Tool Suite](https://docs.solana.com/cli/install-solana-cli-tools#use-solanas-install-tool)

```bash
curl https://release.solana.com/v1.14.3/solana-install-init-x86_64-pc-windows-msvc.exe --output C:\solana-install-tmp\solana-install-init.exe --create-dirs
```
And [Prebuilt Binaries](https://docs.solana.com/cli/install-solana-cli-tools#download-prebuilt-binaries)

Download the binaries by navigating to https://github.com/solana-labs/solana/releases/latest, download solana-release-x86_64-pc-windows-msvc.tar.bz2, then extract the archive using WinZip or similar. Open a Command Prompt and run: 

```bash
cd solana-release/
set PATH=%cd%/bin;%PATH%
```

If you run this command and all it's ok, you sucessfully installed solana: 

```bash
solana --version
```

![Screenshot_6](https://user-images.githubusercontent.com/34518489/195973523-baac3725-7875-4ebb-bdf5-b8d4b4eb86ea.png)


# Create a project

Lets start by setting up our folder, we are creating a project called `solana-migration-xdc`, create a new folder by running on terminal

```bash
mkdir solana-migration-xdc && cd solana-migration-xdc
```

And running `truffle init`. If truffle is correctly installed on your local environment, you should see the following message:

```bash
Starting init...
================
> Copying project files to /home/your/path/to/solana-migration-xdc
Init successful, sweet!
Try our scaffold commands to get started:
  $ truffle create contract YourContractName # scaffold a contract
  $ truffle create test YourTestName         # scaffold a test
http://trufflesuite.com/docs
```
Now we going to install some dependencies for solana: 

```bash
yarn add @solana/solidity @solana/web3.js dotenv @truffle/hdwallet-provider
```

I recommend for windows to download the solang.exe. Sometimes the Vs Extension doesn't work properly. Navigate to [hyperledger-solang-releases](https://github.com/hyperledger/solang/releases/) and search for the solang.exe, you can download it directly to your root project: 

![Screenshot_4](https://user-images.githubusercontent.com/34518489/195971737-b3392d35-7758-4604-b8e2-167847cf70bd.png)


# Create an XDC Network Wallet

- Easy way to create a wallet is installing the XDC Pay wallet extension download it from [here](https://chrome.google.com/webstore/detail/xdcpay/bocpokimicclpaiekenaeelehdjllofo)
- Receive 1000 free testnet XDC tokens using [Faucet](https://faucet.apothem.network/) 

# Write a smart contract

Lets create a simple incrementer.sol in the contracts folder , the incrementer contract should have:

- a constructor that initializes the `int32` value to the given `init_value`,
- a `inc` method to increment the state value,
- a `get` method to know what is the state value

Note: `Solang aims to be compatible with Solidity 0.7`

Write the following code to incrementer.sol:

```solidity
pragma solidity ^0.7.0;

contract incrementer {
	uint32 private value;

	/// Constructor that initializes the `int32` value to the given `init_value`.
	constructor(uint32 initvalue) {
		value = initvalue;
	}

	/// This increments the value by `by`. 
	function inc(uint32 by) public {
		value += by;
	}

	/// Simply returns the current value of our `uint32`.
	function get() public view returns (uint32) {
		return value;
	}
}
```

The `Solang extension` should compile the code automatically and a folder `Build` with 2 files: `bundle.so` and `incrementer.abi`. 

# Compile and migrate the smart contract from Solana to the XDC Network

If the extension doesn't work and you cannot see a build folder, try running this command: 

```bash
solang.exe compile contracts/incrementer.sol --target solana --output build
```

If this works properly you will see the folder with the 2 described files.  

![Screenshot_5](https://user-images.githubusercontent.com/34518489/195973424-997ac17e-7463-4fbe-a09b-9cccb8dcf2b0.png)

Now its compiled with `Solang` we going to deploy it to solana devnet, first lets create a `deployContract.js` file in the project root: 

```jsx
const { Connection, LAMPORTS_PER_SOL, Keypair } = require('@solana/web3.js');
const { Contract, Program } = require('@solana/solidity');
const { readFileSync } = require('fs');

const INCREMENTER_ABI = JSON.parse(readFileSync('./build/incrementer.abi', 'utf8'));
const PROGRAM_SO = readFileSync('./build/bundle.so');

(async function () {
    console.log('Connecting to your local Solana node ...');
    const connection = new Connection(
        //'https://api.mainnet-beta.solana.com',
        'https://api.devnet.solana.com',
        //'http://localhost:8899',
         'confirmed');

    const payer = Keypair.generate();

    console.log('Airdropping SOL to a new wallet ...');
    const signature = await connection.requestAirdrop(payer.publicKey, LAMPORTS_PER_SOL);
    await connection.confirmTransaction(signature, 'confirmed');

    const program = Keypair.generate();
    const storage = Keypair.generate();

    const contract = new Contract(connection, program.publicKey, storage.publicKey, INCREMENTER_ABI, payer);

    await contract.load(program, PROGRAM_SO);

    console.log('Program deployment finished, deploying the incrementer contract ...');

    await contract.deploy('incrementer', [1], storage, 1000);

    const res = await contract.functions.inc(2);
    console.log('Adding number 2 to the value', res)

    const res2 = await contract.functions.get();
    console.log('state: ' + res2.result);
})();
```
Run this command to config the devnet url for solana: 

```bash
solana config set --url devnet
```
Now the RPC and WSS are pointing to the devnet

![Screenshot_7](https://user-images.githubusercontent.com/34518489/195974147-464d6afb-6b01-437b-ab32-51e45be01ac1.png)

Run the command: 

```bash
node deployContract.js
```

If everything is fine and your deployContract script is right you should see this: 

![Screenshot_8](https://user-images.githubusercontent.com/34518489/195974211-dd984a17-33c7-4843-8081-64e53cd75f53.png)

### Migrate to XDC

Here we must first to copy our MNEMONIC phrase from our wallet already installed, so go to the extension and log in with your password and click here: 

![Screenshot_9](https://user-images.githubusercontent.com/34518489/195974467-ecaf5727-b6a4-49fa-8518-00751cee4217.png)

![Screenshot_10](https://user-images.githubusercontent.com/34518489/195974469-0d78635d-0306-4082-b914-5a2aa0c2fb88.png)

![Screenshot_11](https://user-images.githubusercontent.com/34518489/195974480-89eac49d-ed64-4072-b724-d28bbd12e87c.png)

![Screenshot_12](https://user-images.githubusercontent.com/34518489/195974481-795ef5bf-48b7-4f44-b414-cd2aaa6f7fcb.png)

![Screenshot_13](https://user-images.githubusercontent.com/34518489/195974484-c2cb33c9-008c-4e88-a57a-dc1acee8d572.png)

Once you have your phrase create an .env file and paste it into a variable called MNEMONIC.

Go to your truffle-config.js and paste this code: 

```jsx
require("dotenv").config();
const { MNEMONIC } = process.env;
const HDWalletProvider = require("@truffle/hdwallet-provider");

module.exports = {
  networks: {
    xinfin: {
      provider: () =>
        new HDWalletProvider(MNEMONIC, "https://erpc.xinfin.network"),
      network_id: 50,
      gasLimit: 6721975,
      confirmation: 2,
      networkCheckTimeout: 1000000,
      timeoutBlocks: 2000,
    },

    apothem: {
      provider: () =>
        new HDWalletProvider(MNEMONIC, "https://erpc.apothem.network"),
      network_id: 51,
      gasLimit: 6721975,
      confirmation: 2,
      networkCheckTimeout: 1000000,
      timeoutBlocks: 2000,
    },
  },
  mocha: {},
  compilers: {
    solc: {
      version: "0.7.0",
    },
  },
};
```
Now go to Migrations folder and create a file `1_incrementer.js` and paste this code: 

```jsx
const incrementer = artifacts.require("incrementer");

const VALUE = 1

module.exports = function(deployer) {
  deployer.deploy(incrementer, [VALUE]);
};
```


Once your have your truffle-config.js file ready it's time to migrate this contract to XDC network. Run this command: 

```bash
truffle migrate --network apothem
or 
truffle migrate --network xinfin
```
this should be the output: 

![Screenshot_14](https://user-images.githubusercontent.com/34518489/195974665-a6a4638f-5d06-4016-9fec-40fa97d80552.png)

Copy the transaction hash to check it in the [block explorer](https://explorer.apothem.network/) 

![Screenshot_15](https://user-images.githubusercontent.com/34518489/195974754-2c5a38aa-503b-47fd-87e1-100ea0d8aa0c.png)

Congrats! you have migrated a solana contract compiled with `Solang` to the `XDC network`


