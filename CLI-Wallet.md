- [Overview](#overview)
- [How to Run keosd](#how-to-run-keosd)
- [Command Reference](#command-reference)


## Overview

The program `keosd`, located in the `eos/build/programs/keosd` folder within the EOSIO/eos repository, can be used to store private keys that will be used to sign transactions sent to the block chain. `keosd` runs on your local machine and stores your private keys locally.

## How to run `keosd`

Start `keosd` on as follows:

```
$ keosd 
```

By default, `keosd` creates the folder `~/eosio-wallet` and populates it with a basic `config.ini` file.  The location of the config file can be specified on the command line using the `--config-dir` argument.  The configuration file contains the http server endpoint for incoming http connections and other parameters for cross origin resource sharing.

By default, `keosd` stores wallets in the `~/eosio-wallet` folder.  Wallet files follow the naming convention `<wallet-name>.wallet`.  For example, the default wallet will be stored in a file named `default.wallet`.  As other wallets are created, similar files will be created for each.  For example, a wallet named "foo" will have a corresponding wallet file named `foo.wallet`.  The location of the wallet data folder can be specified on the command line using the --data-dir argument.


## Command Reference

The command line tool to interact with keosd is called “cleos”. It is found in eos/build/programs/cleos folder.

It provides the following commands to interact with keosd:

### Create

```
$ cleos wallet create ${options}
```

Options:

  -n,--name TEXT=default      The name of the new wallet

If you don’t provide an optional name it creates a default wallet. 

### Open

Open an already created wallet. You need to open a wallet to operate on it.

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

List of private keys from all unlocked wallets in WIF format.

```
$ cleos wallet keys
```

