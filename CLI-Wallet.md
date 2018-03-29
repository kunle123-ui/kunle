- [1. Overview](#1-overview)
- [2. Purpose](#2-purpose)
- [3. How to run the Wallet](#3-how-to-run-the-wallet)
- [4. Available Commands](#4-available-commands)

## 1. Overview

Program: eosiowd
Path: eos/build/programs/eosiowd

## 2. Purpose

To store private keys that will be used to sign transactions sent to the block chain. Please note that this is a local process running on your local machine and that it also stores your private keys locally.

## 3. How to run the Wallet

Start eosiowd process on your local instance as follows:

```
$ eosiowd 
```

You will notice that it creates a folder called data-dir and contains a config.ini file. The configuration file contains the http server endpoint for incoming http connections and other parameters for cross origin resource sharing.

The default parameters should be sufficient for running a local instance of wallet process.

## 4. Available Commands

The command line tool to interact with eosiowd is called “cleos”. It is found in eos/build/programs/cleos folder.

It provides the following commands to interact with eosiowd:

### Create

```
$ cleos wallet create ${options}
```

Options:

  -n,--name TEXT=default      The name of the new wallet

If you don’t provide an optional name it creates a default wallet. 

### Open

Open an already created wallet. You need to open a wallet (if you are not on default wallet) to operate on it.

```
$ cleos wallet open ${options}
```

Options:

  -n,--name TEXT              The name of the wallet to open

### Lock

Locks a wallet.

```
$ cleos wallet lock ${options}
```

Options:

  -n,--name TEXT              The name of the wallet to lock

### Unlock wallet

```
$ cleos wallet unlock ${options}
```

Options:

  -n,--name TEXT              The name of the wallet to unlock
  --password TEXT             The password returned by wallet create

### Import private key into wallet

```
$ cleos wallet import ${options} key
```

Positionals:

  key TEXT                    Private key in WIF format to import

Options:

  -n,--name TEXT              The name of the wallet to import key into

### List

List opened wallets, * = unlocked

```
$ cleos wallet list
```

### Keys

List of private keys from all unlocked wallets in wif format.

```
$ cleos wallet keys
```

