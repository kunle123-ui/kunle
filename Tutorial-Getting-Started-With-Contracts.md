## Getting Started with Contracts 

The purpose of this tutorial is to demonstrate how to setup a local blockchain 
that can be used to experiment with smart contracts. The first part of this
tutorial will focus on:

- Starting a Private Blockchain
- Creating a Wallet
- Loading the Bios Contract
- Creating Accounts

The second part of this tutorial will walk you through creating and deploying
your own contracts.

- eosio.token Contract
- Exchange Contract
- Hello World Contract

This tutorial assumes that you have installed EOSIO and that `nodeos` and 
`cleos` are in your path. 


## Starting a Private Blockchain

You can start your own single-node blockchain with this single command:

```
$ nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::account_history_api_plugin 
...
eosio generated block 046b9984... #101527 @ 2018-04-01T14:24:58.000 with 0 trxs
eosio generated block 5e527ee2... #101528 @ 2018-04-01T14:24:58.500 with 0 trxs

```

This command sets many flags and loads some optional plugins which we will need for the rest of this tutorial. Assuming everything worked properly, you should see a block generation message every 0.5 seconds.  

```
eosio generated block 046b9984... #101527 @ 2018-04-01T14:24:58.000 with 0 trxs
```

This means your local blockchain is live, producing blocks, and ready to be used.

For more information about the arguments to `nodeos` you can use:

```
nodeos --help
```

## Creating a Wallet

A wallet is a repository of private keys necessary to authorize actions on the blockchain.  These keys are stored on disk encrypted using a password generated for you.  This password should be stored in a secure password manager.

```
$ cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JuBXoXJ8JHiCTXfXcYuJabjF9f9UNNqHJjqDVY7igVffe3pXub"
```

For the purpose of this simple development environment, your wallet is being managed by your local `nodeos` via the `eosio::wallet_api_plugin` we enabled when we started `nodeos`.  Any time you restart `nodeos` you will have to unlock your wallet before you can use the keys within.

```
$ cleos wallet unlock --password PW5JuBXoXJ8JHiCTXfXcYuJabjF9f9UNNqHJjqDVY7igVffe3pXub
Unlocked: default
```

It is generally not secure to use your password directly on the commandline where it gets logged to your bash history, so you can also unlock in interactive mode:

```
$ cleos wallet unlock
password:
```

For security purposes it is generally best to leave your wallet locked when you are not using it.  To lock your wallet without shutting down `nodeos` you can do:

```
$ cleos wallet lock
Locked: default
```

You will need your wallet unlocked for the rest of this tutorial.

## Loading the Bios Contract

Now that we have a wallet with the key for the `eosio` account loaded, we can set a default system contract.  For the purposes of development, the default `eosio.bios` contract can be used.  This contract enables you to have direct control over the resource allocation of other accounts and to access other privileged API calls. In a public blockchain, this contract will manage the staking and unstaking of tokens to reserve bandwidth for CPU and network activity, and memory for contracts. 

The `eosio.bios` contract can be found in the `contracts/eosio.bios` folder of your EOSIO source code.  The command sequence below assumes it is being executed from the root of the EOSIO source, but you can execute it from anywhere by specifying the full path to `${EOSIO_SOURCE}/build/contracts/eosio.bios`.

```
$ cleos set contract eosio build/contracts/eosio.bios -p eosio
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
```

The result of this command sequence is that `cleos` generated a transaction with two actions, `eosio::setcode` and `eosio::setabi`.  

The code defines how the contract runs and the abi describes how to convert between binary and json representations of the arguments.  While an abi is technically optional, all of the EOSIO tooling depends upon it for ease of use.  

Any time you execute a transaction you will see output like:
```
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
```

This can be read as: The action `setcode` as defined by `eosio` was executed by `eosio` contract with `{args...}`.

```
#         ${executor} <= ${contract}:${action} ${args...}
> console output from this execution, if any
```

As we will see in a bit, actions can be processed by more than one contract.

The last argument to this call was `-p eosio`.  This tells `cleos` to sign this action with the active authority of the `eosio` account, i.e., to sign the action using the private key for the `eosio` account that we imported earlier. 

## Creating Accounts

Now that we have setup the basic system contract, we can start to create our own accounts.  We will create two accounts, `user` and `tester`, and we will need to associate a key with each account.  In this example, the same key will be used for both accounts.

To do this we first generate a key for the accounts. 

```
$ cleos create key
Private key: 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
Public key: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
```

Then we import this key into our wallet:
```
$ cleos wallet import 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
imported private key for: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
```
**NOTE:** Be sure to use the actual key value generated by the `cleos` command and not the one shown in the example above!

Keys are not automatically added to a wallet, so skipping this step could result in losing control of your account.

### Create Two User Accounts

Next we will create two accounts, `user` and `tester`, using the key we created and imported above.

```
$ cleos create account eosio user EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 8aedb926cc1ca31642ada8daf4350833c95cbe98b869230f44da76d70f6d6242  364 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"user","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...

$ cleos create account eosio tester EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083 366 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"tester","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...
```
**NOTE:** The `create account` subcommand requires two keys, one for the OwnerKey (which in a production environment should be kept highly secure) and one for the ActiveKey.  In this tutorial example, the same key is used for both.

Because we are using the `eosio::account_history_api_plugin` we can query all accounts that are controlled by our key:

```
$ cleos get accounts EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
{
  "account_names": [
    "tester",
    "user"
  ]
}
```

## Next Step - Eosio.token, Exchange, and Eosio.msig Contracts
You are now ready to proceed to the next tutorial, [Eosio.token, Exchange, and Eosio.msig Contracts](Tutorial-eosio-token-Contract)

