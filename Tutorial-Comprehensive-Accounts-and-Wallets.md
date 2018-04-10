## Comprehensive Accounts & Wallets Tutorial

**Notice** This tutorial is geared towards use on a **private testnet**, but will work on public testnet with minor modifications. 

#### What you'll learn

You'll learn how to create and manage wallets, their keys and then use this wallet to interact with the blockchain through `cleos`.

#### Who this Tutorial is For

This tutorial is for users who want to learn about wallet and account management. It will attempt to teach you as much about `cleos` and how wallets and accounts on EOS interact with each other. Advanced users may be better suited to check out the **[Command Reference](Command%20Reference)**

#### Prerequisites

- Built and running copy of `cleos` and `keosd` on your system.
- Basic understanding of command line interface. 

**Note:** Instructions require slight modification when applied to a docker installation. 

### Creating and Managing Wallets

#### Open your terminal and change to the directory where EOSIO was built

This will make it easier for us to interact with `cleos`, which is a command line interface for interacting with `nodeos` and `keosd`. 

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

You'll notice in the last response that the default wallet is locked by default.  Unlock it now; you'll need it in the subsequent steps. 

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

**TODO:  the following steps don't work.  Need to align keosd 

Performing actions on the blockchain requires the use of accounts.  We use `cleos` to request `nodeos` to create accounts and publish them on the blockchain.

First *examine* the `create account` command and its positional arguments.

```bash
$ cleos create account inita ${desired_account_name} ${public_key_1} ${public_key_2}
```

**Breakdown of the positional arguments for `create account` command**

- `inita` is the name of the **account name** that will fund the account creation, and subsequently the new account.
- `desired_account_name` is the name of the account you would like to create
- `public_key_1` and `public_key_2` are public keys, the first one will be permissioned as the owner authority of your account, and the second one will be permissioned for the active authority of your account. 

You generated two keypairs previously, you can either scroll up in your console, as you executed this command only moments ago, or execute `wallet keys` if you disdain scrolling or have cleared your screen

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

As a reminder, your public keys start with `EOS...`. The keys above are arbitrary until you have added them to an authority, which one you decide to user for active and owner are inconsequential until you have created your account.

_Reminder,_ your owner key equates to full control of your account, whereas your active key equates to full access over funds in your account. 

With all you've just learned, replace the placeholders in the following command and press enter. 

```bash
$ cleos create account inita ${desired_account_name} ${public_key_1} ${public_key_2}
```
Did you recieve an error mentioning "authorities" somewhere? The reason you got an error, is that you do not have the **@inita**  account keys loaded.

**inita's** keys are included in `config.ini` but for your convenience I've went ahead and grabbed the key and provided it below. Just execute the provided command, maybe that makes up for forcing you through an error?

```bash
$ cleos wallet import 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```
it will respond with 

```bash
imported private key for: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
```

Now that the **@inita** account's keys are loaded, execute the `create account` command you tried earlier.

If all goes well, `cleos` will return a json object with a transaction ID similar to the following

```
{
  "transaction_id": "6acd2ece68c4b86c1fa209c3989235063384020781f2c67bbb80bc8d540ca120",
  "processed": {
    "refBlockNum": "25217",
    "refBlockPrefix": "2095475630",
    "expiration": "2017-07-25T17:54:55",
    "scope": [
      "eos"...
```

Great! You now have an account and it is on a blockchain. 

Go ahead and pat yourself on the back, you've created a wallet, learned a bit about how wallets work, generated keys, imported them into your wallet and created an account.