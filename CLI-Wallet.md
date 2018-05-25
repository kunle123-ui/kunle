- [Overview](#overview)
- [How to Run `keosd`](#how-to-run-keosd)
- [Command Reference](#command-reference)
- [Managing Wallets with `nodeos`](#managing-wallets-with-nodeos)

## Overview

The program `keosd`, located in the `eos/build/programs/keosd` folder within the EOSIO/eos repository, can be used to store private keys that will be used to sign transactions sent to the block chain. `keosd` runs on your local machine and stores your private keys locally.

## How to Run `keosd`

Start `keosd` on as follows:

```
$ keosd 
```

By default, `keosd` creates the folder `~/eosio-wallet` and populates it with a basic `config.ini` file.  The location of the config file can be specified on the command line using the `--config-dir` argument.  The configuration file contains the http server endpoint for incoming http connections and other parameters for cross origin resource sharing.

By default, `keosd` stores wallets in the `~/eosio-wallet` folder.  Wallet files follow the naming convention `<wallet-name>.wallet`.  For example, the default wallet will be stored in a file named `default.wallet`.  As other wallets are created, similar files will be created for each.  For example, a wallet named "foo" will have a corresponding wallet file named `foo.wallet`.  The location of the wallet data folder can be specified on the command line using the --data-dir argument.

`keosd` is started automatically by `cleos`.  It is possible, when doing development and testing, that `keosd` is started manually (not by `cleos`) and you end up with multiple `keosd` processes running.  When multiple instances of `keosd` are running on the same server, you might find that your `cleos` command is not finding the right set of keys.  To check whether multiple instances of `keosd` are running, and what ports they are running on, you can try something like the following to isolate the `keosd` processes and ports in use:
```
$ pgrep keosd | xargs printf " -p %d" | xargs lsof -Pani
COMMAND   PID         USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
keosd   49590 tutorial        6u  IPv4 0x72cd8ccf8c2c2d03      0t0  TCP 127.0.0.1:8900 (LISTEN)
keosd   62812 tutorial        7u  IPv4 0x72cd8ccf90428783      0t0  TCP 127.0.0.1:8899 (LISTEN)
```

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

_**Note:** Your wallets will be locked when your wallet manager (`keosd` or `nodeos`) is shut down.  You will need to unlock your wallets after restarting the wallet manager._

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

Read the full [Cleos Command Reference](Command%20Reference)

## Managing Wallets with `nodeos`

As an alternative to using `keosd` to manage your wallets, you can use `nodeos` to manage them.  Using `nodeos` for wallet management is not the preferred configuration, but is suitable for development and test. _It is not recommended to simultaneously have `keosd` and `nodeos` doing wallet management; it won't break anything, but you will likely find it very confusing!_

To use `nodeos` for wallet management, you must configure `nodeos` to use the `eosio::wallet_api_plugin`. You can do this on the command line argument when starting `nodeos`, or by defining it in the `nodeos` config file.  You will still use `cleos` as your interface to perform wallet management tasks, the difference being that `cleos` will direct its requests to `nodeos`.

To start `nodeos` with the wallet plugin, include `--plugin eosio::wallet_api_plugin` on the `nodeos` command line.

To include the wallet using the `nodeos` config file, add the following line to the `nodeos` `config.ini` file:
```
plugin = eosio::wallet_api_plugin 
```

_**Note:** If you use `nodeos` for wallet management, your wallets will be locked when `nodeos` is shut down.  You will need to unlock your wallets after restarting `nodeos`._
