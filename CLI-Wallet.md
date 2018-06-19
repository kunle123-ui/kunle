**The wiki is deprecated and will be removed June 30th. Please check out the constantly improving [developers portal](http://developers.eos.io)**

----

- [Overview](#overview)
- [How to Run `keosd`](#how-to-run-keosd)
- [Command Reference](#command-reference)
- [Managing Wallets with `nodeos`](#managing-wallets-with-nodeos)

## Overview

The program `keosd`, located in the `eos/build/programs/keosd` folder within the EOSIO/eos repository, can be used to store private keys that cleos will use to sign transactions sent to the block chain. `keosd` runs on your local machine and stores your private keys locally.

## How to Run `keosd`

For most users the easiest way to use keosd is to have cleos launch it automatically. By default, `keosd` creates the folder `~/eosio-wallet` and populates it with a basic `config.ini` file. Wallet files (named `foo.wallet` for example) will also be created in this directory by default.

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

Unlocking a wallet will also open the wallet if it is not already open.

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

### Create a key pair within the wallet

Creates a key pair within the wallet so that you don't need to manually import it like you would with `cleos create key`. By default this will create a key with the type "favored" by the wallet, which is a K1 key. But this command also lets you create a key in R1 format.

```
$ cleos wallet create_key ${options}
```

Positionals:

  key_type K1/R1              Key type to create (K1/R1)

Options:

  -n,--name TEXT=default      The name of the wallet to create key into

### List

List opened wallets, * = unlocked

```
$ cleos wallet list
```

### Public Keys

List of public keys from all unlocked wallets. These are the keys that could be used to sign transactions.

```
$ cleos wallet keys
```

### Public/Private Key Pairs

It is possible to query for the public and private key pairs of an individual wallet. The wallet must already be unlocked _and you must give the password again_.

```
$ cleos wallet private_keys ${options}
```

Options:

  -n,--name TEXT=default      The name of the wallet to list keys from
  --password TEXT             The password returned by wallet create

Read the full [Cleos Command Reference](Command%20Reference)

## Auto locking

By default keosd is set to auto lock your wallets after 15 minutes of inactivity. This is configurable in the `config.ini`. Be aware if you need to disable this feature you will have to set an enormous number -- setting it to 0 will cause keosd to always lock your wallet.

## Launching keosd manually

It is possible to launch `keosd` manually simply with

```
$ keosd 
```

By default, `keosd` creates the folder `~/eosio-wallet` and populates it with a basic `config.ini` file.  The location of the config file can be specified on the command line using the `--config-dir` argument.  The configuration file contains the http server endpoint for incoming http connections and other parameters for cross origin resource sharing. Be aware that if you allowed cleos to auto launch keosd, a config.ini will be generated that will be sightly different than if you had launched keosd manually.

The location of the wallet data folder can be specified on the command line using the --data-dir argument.

## Managing Wallets with `nodeos`

keosd is just an application containing the wallet_plugin, wallet_api_plugin and http_plugin. As an alternative to using `keosd` to manage your wallets, you can use `nodeos` to manage them by configuring `nodeos` to use the `eosio::wallet_api_plugin` which will integrate the wallet within nodeos. This is generally not recommended.

To start `nodeos` with the wallet plugin, include `--plugin eosio::wallet_api_plugin` on the `nodeos` command line.

To include the wallet using the `nodeos` config file, add the following line to the `nodeos` `config.ini` file:
```
plugin = eosio::wallet_api_plugin 
```

Be aware that you will need to point `cleos` to use `nodeos` as your wallet now. Your `cleos` command may end up looking something like

```
$ cleos --wallet-url http://localhost:8888 wallet keys
```

_**Note:** If you use `nodeos` for wallet management, your wallets will be locked when `nodeos` is shut down.  You will need to unlock your wallets after restarting `nodeos`._

## Multiple keosd running

It is possible, when doing development and testing, that `keosd` is started manually (not by `cleos`) and you end up with multiple `keosd` processes running.  When multiple instances of `keosd` are running on the same server, you might find that your `cleos` command is not finding the right set of keys.  To check whether multiple instances of `keosd` are running, and what ports they are running on, you can try something like the following to isolate the `keosd` processes and ports in use:
```
$ pgrep keosd | xargs printf " -p %d" | xargs lsof -Pani
COMMAND   PID         USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
keosd   49590 tutorial        6u  IPv4 0x72cd8ccf8c2c2d03      0t0  TCP 127.0.0.1:8900 (LISTEN)
keosd   62812 tutorial        7u  IPv4 0x72cd8ccf90428783      0t0  TCP 127.0.0.1:8899 (LISTEN)