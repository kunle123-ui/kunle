## Comprehensive Accounts & Wallets Tutorial

**Note** _This tutorial is geared towards use on a **private single node testnet**, but will work on a public testnet with minor modifications._ 

#### Tutorial Audience

This tutorial is for users who want to learn about wallet and account management, how to use `cleos` to manage wallets and accounts, and how the wallet and account management EOSIO components interact with each other.  Additional information can be found in the **[Command Reference](Command%20Reference)**.

#### What You'll Learn

You'll learn how to create and manage wallets, their keys and then use this wallet to interact with the blockchain through `cleos`.  You will then learn how to create accounts using `cleos`.  The tutorial will introduce you to some of the interaction between `cleos`, `keosd`, and `nodeos` to sign content published to the blockchain.

#### Prerequisites

- Built and running copy of `cleos` and `keosd` on your system.
- Built and _ready to run_ copy of `nodeos` on your system.
- Basic understanding of command line interfaces. 

**Note:** Instructions require slight modification when applied to a docker installation. 

### EOSIO Accounts and Wallets Conceptual Overview

The diagram below provides a simple conceptual view of accounts and wallets in EOSIO.  While there are other supported deployment configurations, this view matches the one that we will use for this tutorial.

![Accounts and Wallets Conceptual Overview](assets/Accounts-and-Wallets-Overview.png)

The wallet can be thought of as a repository of public-private key pairs.  These are needed to sign operations performed on the blockchain.  Wallets and their content are managed by `keosd`.  Wallets are accessed using `cleos`.

An account can be thought of as an on-chain identifier that has access permissions associated with it (i.e., a security principal).  `nodeos` manages the publishing of accounts and account-related actions on the blockchain.  The account management capabilities of `nodeos` are also accessed using `cleos`.

There is no inherent relationship between accounts and wallets.  Accounts do not know about wallets, and vice versa.  Correspondingly, there is no inherent relationship between `nodeos` and `keosd`.  Their basic functions are fundamentally different.  (_Having said that, there are deployment configurations that blur the distinction.  However, that topic is beyond the scope of this tutorial._)

Where overlap occurs is when signatures are required, e.g., to sign transactions.  The wallet facilitates obtaining signatures in a secure manner by storing keys locally in an encrypted store that can be locked.  `cleos` effectively serves as an intermediary between `keosd` key retrieval operations and `nodeos` account (and other) blockchain actions that require signatures generated using those keys.

### Creating and Managing Wallets

Open your terminal and change to the directory where EOSIO was built.  This will make it easier for us to interact with `cleos`, which is a command line interface for interacting with `nodeos` and `keosd`. 

```bash
$ cd /path_to_eos/build/programs/cleos
```

The first thing you'll need to do is create a wallet; use the `wallet create` command of `cleos`

```bash
$ cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"A MASTER PASSWORD"
```

A wallet called _default_ is now inside `keosd` and has returned the **master password** for this wallet. Be sure to save this password somewhere safe. This password is used to unlock (decrypt) your wallet file. 

The file for this wallet is named `default.wallet`.  By default, `keosd` stores wallets in the `~/eosio-wallet` folder.    The location of the wallet data folder can be specified on the command line using the `--data-dir` argument.

#### Managing Multiple Wallets and Wallet Names

`cleos` is capable of managing multiple wallets. Each individual wallet is protected by different wallet master passwords. The example below creates another wallet and demonstrates how to name it by using the `-n` argument.

```bash
$ cleos wallet create -n periwinkle
Creating wallet: periwinkle
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"A MASTER PASSWORD"
```

Now confirm that the wallet was created with your chosen name.

```bash
$ cleos wallet list
Wallets:
[
  "default *",
  "periwinkle *"
]
```

It's important to notice the asterisk (*) after each listed wallet, which means that the respective wallet is unlocked. When using `create wallet` the resulting wallet is unlocked by default for your convenience.

Lock that second wallet using `wallet lock`

```bash
$ cleos wallet lock -n periwinkle
Locked: 'periwinkle'
```

Upon running `wallet list` again, you will see that the asterisk is gone, meaning that the wallet is now locked.

```bash
$ cleos wallet list
Wallets:
[
  "default *",
  "periwinkle"
]
```

Unlocking a named wallet entails calling `wallet unlock` with an `-n` parameter followed by the name of the wallet and then entering the wallet's master password in the password prompt (yes, you can paste the password). Go ahead and grab the master key for the second wallet you created, execute the command below and when the _password_ prompt appears, paste and press _enter_. You'll be presented with a confirmation.

```bash
$ cleos wallet unlock -n periwinkle
```

cleos will let you know that the wallet was unlocked

```bash
Unlocked: 'periwinkle'
```

_**Note:** You can also use a `--password` argument followed by the master password to skip the prompt, but this results in your master password being visible in the console history_

Now check your progress:

```bash
$ cleos wallet list
Wallets:
[
  "default *",
  "periwinkle *"
]
```

The _periwinkle_ wallet is followed by an asterisk, so it is now unlocked. 

_**Note:** Interacting with the 'default' wallet using the wallet command does not require the `-n` parameter_

Restart `keosd` now, and then go back to where you were calling `cleos` and run the following command

```bash
$ cleos wallet list
Wallets:
[]
```

Wallets first need to be opened before they can be operated on, including listing them.  The wallets were locked when you shut down `keosd`.  When `keosd` was restarted, the wallet wasn't open.  Run the following commands to open, then list the default wallet.

```bash
$ cleos wallet open
$ cleos wallet list
Wallets:
[
  "default"
]
```

_**Note:** If you wanted to open a named wallet, you would run `$ cleos wallet open -n periwinkle`._

You'll notice in the last response that the default wallet is locked by default.  Unlock the now; you'll need it in the subsequent steps. 

Run the `wallet unlock` command and paste your _default_ wallet's master password when the password prompt appears.

```bash
$ cleos wallet unlock
Unlocked: 'default'
```
Check that the wallet is unlocked:

```bash
$ cleos wallet list
Wallets:
[
  "default *"
]
```
The wallet is accompanied by an asterisk, so it's unlocked. 

You've learned how to create multiple wallets, and interact with them in `cleos`.  However, an empty wallet doesn't do you much good.  We will now learn to import keys into the wallet. 

### Generating and Importing EOSIO Keys

There are several ways to generate an EOSIO key pair, but this tutorial will focus on the `create key` command in `cleos`.

Generate two public/private key pairs.  Note the general format of the keys.

```bash
$ cleos create key
Private key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Public key: EOSXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
$ cleos create key
Private key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Public key: EOSXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

You now have two EOSIO keypairs. At this point, these are just arbitrary keypairs and by themselves have no authority.

If you followed all of the previous steps, your _default_ wallet should be open and unlocked.

In this next step, we will import the private keys into the wallet.  To do this, execute `wallet import` twice, once for each *private key* that was generated earlier.

```bash
$ cleos wallet import ${private_key_1}
```
And then again with the second private key

```bash
$ cleos wallet import ${private_key_2}
```

If successful, each time `wallet import` command responds with the public key corresponding to your private key, your console should look something like..

```bash
$ cleos wallet import 5Hvgh37WMCWAc4GuyRBiFrk4SArCnSQVMhNtEQVoszhBh6RgdWr
imported private key for: EOS84jJqXj5XBz3RqLreXZCMxXRKspUadXg3AVy8eb5J2axj8cywc
```

We can check which keys are loaded by calling `wallet keys`

```bash
$ cleos wallet keys
[[
    "EOS6....",
    "5KQwr..."
  ],
  [
    "EOS3....",
    "5Ks0e..."
  ]
]
```

the wallet file itself is encrypted, so the wallet will protect these keys when it's locked.  Accessing the keys in a locked wallet requires the master password that was provided to you during wallet creation. 

### Backing up your wallet

Now that your wallet contains keys, it's good to get into the habit of backing up the wallet, e.g., to a flash drive or other media, to protect against the loss of the wallet file.  Without the password, the wallet file is encrypted with high entropy, and the keys inside are incredibly difficult (improbable by all reasonable measures) to access.

You can find your wallet files in the `data-dir`. If you did not specify a `--data-dir` parameter when launching eos, your wallet files are stored in the `~/eosio-wallet` folder.

```bash
$ cd ~/eosio-wallet && ls
blockchain   blocks   config.ini   default.wallet   periwinkle.wallet
```

If you've been following the steps of this tutorial, you will the two files, `default.wallet` and `periwinkle.wallet`.  Save these files in a safe location.

### Creating an Account

Performing actions on the blockchain requires the use of accounts.  We use `cleos` to request `nodeos` to create accounts and publish them on the blockchain.  At this point in our tutorial, we need to start `nodeos`.  The following command will start a single node testnet.  See [Creating and Launching a Single Node Testnet](Local-Environment#4-creating-and-launching-a-single-node-testnet) for more on setting up a local environment.

>  For this part of the tutorial, we need keosd and nodeos to run simultaneously.  Presently, the default port for
>  `keosd` and `nodeos` is the same (port 8888).  To simplify running nodeos for this part of the tutorial, we will
>  change the port for `keosd` to 8899.  There are two ways that we can do this:
> 
>  1.  Edit the `keosd` config file (`~/eosio-wallet/config.ini`) and change the http-server-address property to:
>    
>       http-server-address = 127.0.0.1:8899 
>   
>  2.  Start keosd using the command line argument --http-server-address=localhost:8899
>
>  Restart keosd using the comand line argument:
>     
>       $ pkill keosd
>       $ keosd --http-server-address=localhost:8899
>    
>  Unlock your default wallet (it was locked when keosd was restarted).  Since keosd was started listening on
>  port 8899, you will need to use the --wallet-port command line argument to cleos.
>    
>       cleos --wallet-port=8899 wallet unlock
>   
>  When prompted for the password, enter the wallet password generated in the previous section when the wallet
>  was created.

To start `nodeos`, open a new terminal window, go to the folder that contains your `nodeos` executable, and run the following:

```bash
$ cd eos/build/programs/nodeos
$ nodeos -e -p eosio --plugin eosio::chain_api_plugin --plugin eosio::account_history_api_plugin
```

Now we can create the account.  Here is the structure of the `cleos create account` command.

```bash
$ cleos create account ${authorizing_account} ${new_account} ${owner_key} ${active_key}
```

- `authorizing_account` is the name of the account that will fund the account creation, and subsequently the new account.
- `new_account` is the name of the account you would like to create
- `owner_key` is a public key to be assigned to the **owner** authority of the account.  (See [Accounts & Permissions](Accounts%20%26%20Permissions))
- `active_key` is a public key to be assigned to the **active** authority of the account. of your account, and the second one will be permissioned for the active authority of your account. 

In this tutorial, `eosio` is the authorizing account.  The actions performed on the blockchain must be signed using the keys associated with the `eosio` account.  The `eosio` account is a special account used to bootstrap EOSIO nodes.  The keys for this account can be found in the `nodeos` config file, located at `~/.local/shared/eosio/config/config.ini` on Linux platforms, and `~/Libraries/Application Support/eosio/nodeos/config/config.ini` on MacOS.

We need a name for our new account.  Account names must conform to the following guidelines:

- Must be less than 13 characters
- Can only contain the following symbols:   .12345abcdefghijklmnopqrstuvwxyz 

We will use the name "myaccount" for the new account.

We will use the public keys that you generated above and imported into your wallet (recall that the public keys begin with `EOS`).  The keys are arbitrary until you have assigned them to an authority.  However, once assigned, it is important to remember the assignments.  Your owner key equates to full control of your account, whereas your active key equates to full access over funds in your account. 

Use `cleos create account` to create your account:

```bash
$ cleos --wallet-port=8899 create account eosio myaccount ${public_key_1} ${public_key_2}
```

If successful, you will see output similar to the following.

```bash
executed transaction: 7f1c6b87cd6573365a7bb3c6aa12f8162c3373d57d148f63f2a2d3373ad2fd54  352 bytes  102400 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"myaccount","owner":{"threshold":1,"keys":[{"key":"EOS5kkAs8HZ88m2N7iZWy4J...
```

#### Account Related Operations
There are a number of `cleos` commands specific to accounts.

`cleos` command | Description
----- | -----
`create account` | Create a new account on the blockchain
`get account` | Retrieve an account from the blockchain
`get code` | Retrieve the code and ABI for an account
`get accounts` | Retrieve accounts associated with a public key
`get servants` | Retrieve accounts which are servants of a given account 
`get transactions` | Retrieve all transactions with specific account name referenced in their scope
`set contract` | Create or update the contract on an account
`set account` | set or update blockchain account state
`transfer` | Transfer EOS from account to account
