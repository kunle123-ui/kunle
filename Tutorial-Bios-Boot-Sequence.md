# Bios Boot Sequence Tutorial 
The description below covers the bios boot sequence to start a blockchain from a single "genesis" node, transitioning to a voted on set of producers.

The script `bios-boot-tutorial.py` implements these steps in a single-server installation (all `nodeos` nodes running on the same server). The script can be run with no arguments directly from the `programs/bios-boot-tutorial` directory.

```
cd programs/bios-boot-tutorial
./bios-boot-tutorial.py
```

The script logs the commands it executes in `./test.log`. This enables you to see what work it has done.

## Manual Execution of the Boot Sequence
The remainder of this document covers the same set of steps as the `bios-boot-tutorial.py` script, but walks you through
the commands step by step, enabling you to directly experience using the commands. The manual steps below do not try to
bring up a full blockchain or do the same scope of work as the script. For example, the script generates almost 400 accounts,
30+ running producer nodes, etc. This would be too unwieldy to do manually!

> _Note: In a live network, the steps involved require considerable coordination among independent parties who participate
in the network. It is likely that the boot sequence will be done collectively by a small group within the blockchain
community who work together closely to get an initial distributed network up and running. The network will then be expanded
to incorporate more members._
>
> _The steps here can be readily expanded for the networked case. Some assumptions are made here
regarding how the parties involved will coordinate with each other. However, there are many
ways that the community can choose to coordinate. The technical aspects of the process are objective; assumptions
of how the coordination might occur are speculative. Several approaches have already been suggested by the community.
You are encouraged to review the various approaches and get involved in the discussions as appropriate._

### Configure the initial set of `nodeos` nodes
In this tutorial, we will start a number of `nodeos` nodes, point them to each other, and eventually
vote on a set of producers.  All of the `nodeos` nodes will run on the same server.  In the following sections, we take
various steps to prepare our candidate set of producers.  We will use the naming convention `accountnumXY`, with XY chosen
from the digits 1-5, e.g.,
```
accountnum11
accountnum12
..
accountnum15
accountnum21
...
accountnum55
```

#### Create config and data directories for each `nodeos`
Since all of the `nodeos` nodes will run on the same server, we need separate config and data directories for each.
The following example creates directories for each `nodeos` under the directory `~/eosio_test`. Use whatever method
works best for you to create a number of accounts, keeping in mind the naming restrictions that account names must
be exactly 12 characters, from the set of a-z and 1-5.
```
$ mkdir ~/eosio_test
$ for (( i = 1; i <= 5; i++ )); do for (( j = 1 ; j <=5 ; j++ )); do mkdir ~/eosio_test/accountnum$i$j; done; done
```
You will use these directories with the `--config-dir` and `--data-dir` parameters on the `nodeos` command line.

#### Configure addresses for peer-to-peer communication
When we set up our producers, we will want to configure them to point to each other in order to perform peer-to-peer
communication. If you set up a full mesh configuration, each node will point to all other nodes. A full mesh network
will quickly grow unwieldy. The description below tells how to point nodes to other nodes in order to create the
peer-to-peer communication. Use this for the nodes you select to be peers, but it is recommended NOT to use this for all
nodes, as your configuration will grow explosively.

Configure IP address and port numbers for peer-to-peer communication among each `nodeos` node. Each `nodeos` can be
configured by setting the `p2p-peer-address` configuration property, either on the command line when starting `nodeos`
(one argument per peer), or by setting the property in the `config.ini` file for `nodeos` (one line per peer).

For example, assuming we are using port numbers 9011-9055 for the producers (i.e., `accountnum11` - `accountnum55`,
respectively), include the following arguments on the `nodeos` command line for `accountnum12`:
```
--p2p-peer-address localhost:9011 --p2p-peer-address localhost:9013 --p2p-peer-address localhost:9014 ...
```
_**OR**_
In the `config.ini` file for `accountnum12`, add lines for each peer:
``` 
p2p-peer-address = localhost:9011
p2p-peer-address = localhost:9013
p2p-peer-address = localhost:9014
...
```
If using the configuration file approach, you will have many copies of the `config.ini` file, one in each
`~/eosio_test/accountnumXY` directory. Whether using the command line or config file approach, be careful to NOT include
the producer's own address in the list of peers.


### Start the "genesis" node
The "genesis" node is the first `nodeos` that we start, that will originate the blockchain. All other nodes will derive
from the genesis node. Do the following on the genesis node.

#### Create a wallet
Create a wallet. By default, `keosd` is automatically started to manage the wallet.
```
$ cleos wallet create
```
Be sure to save the wallet password to enable unlocking the wallet in the future.

The same wallet will be used for all key management for all accounts in this tutorial, regardless of which `nodeos` is
being accessed. In a distributed deployment, wallet management should be a local-only activity.

#### Create the key for the `eosio` account
`nodeos` comes pre-configured with a key pair, but we don't want to use that key. Create a key pair to be used for the
`eosio` account when starting the genesis node. 
``` 
$ cleos create key
Private key: 5JGxnezvp3N4V1NxBo8LPBvCrdR85bZqZUFvBZ8ACrbRC3ZWNYv
Public key: EOS8VJybqtm41PMmXL1QUUDSfCrs9umYN4U1ZNa34JhPZ9mU5r2Cm
```
#### Start the first `nodeos` node
Start the genesis node using the generated key pair.
```
$ nodeos --enable-stale-production --producer-name eosio --private-key '[ "EOS8VJybqtm41PMmXL1QUUDSfCrs9umYN4U1ZNa34JhPZ9mU5r2Cm","5JGxnezvp3N4V1NxBo8LPBvCrdR85bZqZUFvBZ8ACrbRC3ZWNYv" ]' --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin 
```
#### Set the `eosio.token`contract
Create an account for the `eosio.token` contract.  We will generate a unique key pair for this account, but
we will use the same key for both the account owner and active keys.

``` 
$ cleos create key  # for eosio.token
Private key: 5KAVVPzPZnbAx8dHz6UWVPFDVFtU1P5ncUzwHGQFuTxnEbdHJL4
Public key: EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG

$ cleos wallet import 5KAVVPzPZnbAx8dHz6UWVPFDVFtU1P5ncUzwHGQFuTxnEbdHJL4
imported private key for: EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG

$ cleos create account eosio eosio.token EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
executed transaction: d8fc14255dd576359043882e03b9a813c7c13facd7ec33c2c34357b59ffa2746  200 bytes  215 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.token","owner":{"threshold":1,"keys":[{"key":"EOS84BLRbGbFahNJEpnnJ...
```
Set the `eosio.token` contract. Note that this assumes that you built `eos` in the `~/Documents/eos` folder.
```
$ cleos set contract eosio.token ~/Documents/eos/build/contracts/eosio.token
Reading WAST/WASM from /Users/tutorial/Documents/eos/build/contracts/eosio.token/eosio.token.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 17fa4e06ed0b2f52cadae2cd61dee8fb3d89d3e46d5b133333816a04d23ba991  8024 bytes  974 us
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d01000000017f1560037f7e7f0060057f7e...
#         eosio <= eosio::setabi                {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...
```

#### Set the `eosio.msig`contract
Create an account for the `eosio.msig` contract.  We will generate a unique key pair for this account, but
we will use the same key for both the account owner and active keys.

``` 
$ cleos create key  # for eosio.msig
Private key: 5J2zhmRG33uYKd2n5wEydxdiqp6ymVKL5o9qCqAqiK1J3iAaRtT
Public key: EOS73N6SCJ4nENVcv5iCFwu3uMY3o83Vd6qgq41CJhNK9LYhBiWzc

$ cleos wallet import 5J2zhmRG33uYKd2n5wEydxdiqp6ymVKL5o9qCqAqiK1J3iAaRtT
imported private key for: EOS73N6SCJ4nENVcv5iCFwu3uMY3o83Vd6qgq41CJhNK9LYhBiWzc

$ cleos create account eosio eosio.msig EOS73N6SCJ4nENVcv5iCFwu3uMY3o83Vd6qgq41CJhNK9LYhBiWzc EOS73N6SCJ4nENVcv5iCFwu3uMY3o83Vd6qgq41CJhNK9LYhBiWzc
executed transaction: f321e9e47f6b1b3d43975dc6938272da4eee6b39eb36c33a12c9887e45496cfe  200 bytes  142 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.msig","owner":{"threshold":1,"keys":[{"key":"EOS73N6SCJ4nENVcv5iCFw...
```
Set the `eosio.msig` contract. Note that this assumes that you built EOSIO in the `~/Documents/eos` folder.
```
$ cleos set contract eosio.msig ~/Documents/eos/build/contracts/eosio.msig
Reading WAST/WASM from /Users/tutorial/Documents/eos/build/contracts/eosio.msig/eosio.msig.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 007507ad01de884377009d7dcf409bc41634e38da2feb6a117ceced8554a75bc  8840 bytes  925 us
#         eosio <= eosio::setcode               {"account":"eosio.msig","vmtype":0,"vmversion":0,"code":"0061736d010000000198011760017f0060047f7e7e7...
#         eosio <= eosio::setabi                {"account":"eosio.msig","abi":{"types":[{"new_type_name":"account_name","type":"name"}],"structs":[{...
```

#### Create and allocate the `SYS` currency
Create the `SYS` currency with a maximum value of 10 billion tokens. Then issue one billion tokens. You
can replace `SYS` with your specific currency designation.
```
$ cleos push action eosio.token create '[ "eosio", "10000000000.0000 SYS", 0, 0, 0]' -p eosio.token
executed transaction: 0440461e0d8816b4a8fd9d47c1a6a53536d3c7af54abf53eace884f008429697  120 bytes  326 us
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"10000000000.0000 SYS"}

$ cleos push action eosio.token issue '[ "eosio", "1000000000.0000 SYS", "memo" ]' -p eosio
executed transaction: a53961a566c1faa95531efb422cd952611b17d728edac833c9a55582425f98ed  128 bytes  432 us
#   eosio.token <= eosio.token::issue           {"to":"eosio","quantity":"1000000000.0000 SYS","memo":"memo"}
```

#### Set the `eosio.system` contract
Set the `eosio.system` contract. After this we will be able to stake our accounts.
```
$ cleos set contract eosio ~/Documents/eos/build/contracts/eosio.system
Reading WAST/WASM from /Users/tutorial/Documents/eos/build/contracts/eosio.system/eosio.system.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 2150ed87e4564cd3fe98ccdea841dc9ff67351f9315b6384084e8572a35887cc  39968 bytes  4395 us
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001be023060027f7e0060067f7e7e7f7f...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"buyrambytes","base":"","fields":[{"name":"p...
```

### Stake tokens and expand the network
If you've followed the tutorial steps above to this point, you now have a single node configuration configured with the
key contracts:

- eosio.token
- eosio.msig
- eosio.system

We are now ready to being staking accounts and expanding the network.

#### Create staked accounts
During the boot sequence, accounts are staked with their tokens. In the initial staking process, tokens are staked 50/50
CPU and network. Additionally, by default, `cleos` stakes 8 KB of RAM on account creation, paid by the account creator.
In the initial staking, the `eosio` account is the account creator doing the staking. Tokens staked during the initial
token staking process cannot be unstaked and made liquid until after the minimum voting requirements have been met.

For this tutorial, we distribute 1 billion tokens among our member accounts and then stake those. To make the tutorial
more realistic, we distribute the tokens using a Pareto distribution. The Pareto distribution models an 80-20
rule, e.g., in this case, 80% of the tokens are held by 20% of the population. The examples here do not show how to
generate the distribution, focusing instead on the commands to do the staking. The script <INSERT_NAME_HERE> that
accompanies this tutorial uses the Python NumPy (numpy) library to generate a Pareto distribution.

##### Create a staked account
Use the following steps to stake tokens for each account. These steps must be done individually for each account.

> _The key pair is created here for this tutorial. In a "live" scenario, the key and token share for an account
should already be established._
> ```
>    $ cleos create key  # for accountnum11
>    Private key: 5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o
>    Public key: EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt
>    
>    $ cleos wallet import 5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o
>    imported private key for: EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt
>```

Create a staked account with initial resources and public key.
```
$ cleos system newaccount eosio --transfer accountnum11 EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt --stake-net "100000.0000 SYS" --stake-cpu "100000.0000 SYS"
775292ms thread-0   main.cpp:419                  create_action        ] result: {"binargs":"0000000000ea30551082d4334f4d113200200000"} arg: {"code":"eosio","action":"buyrambytes","args":{"payer":"eosio","receiver":"accountnum11","bytes":8192}} 
775295ms thread-0   main.cpp:419                  create_action        ] result: {"binargs":"0000000000ea30551082d4334f4d113200ca9a3b00000000045359530000000000ca9a3b00000000045359530000000001"} arg: {"code":"eosio","action":"delegatebw","args":{"from":"eosio","receiver":"accountnum11","stake_net_quantity":"100000.0000 SYS","stake_cpu_quantity":"100000.0000 SYS","transfer":true}} 
executed transaction: fb47254c316e736a26873cce1290cdafff07718f04335ea4faa4cb2e58c9982a  336 bytes  1799 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"accountnum11","owner":{"threshold":1,"keys":[{"key":"EOS8mUftJXepGzdQ2TaC...
#         eosio <= eosio::buyrambytes           {"payer":"eosio","receiver":"accountnum11","bytes":8192}
#         eosio <= eosio::delegatebw            {"from":"eosio","receiver":"accountnum11","stake_net_quantity":"100000.0000 SYS","stake_cpu_quantity...
```

##### Register as a producer
Some set of the accounts created will be registered as producers. Choose some set of the staked accounts to be producers.
```
$ cleos system regproducer accountnum11 EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt https://accountnum11.com/EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt
1487984ms thread-0   main.cpp:419                  create_action        ] result: {"binargs":"1082d4334f4d11320003fedd01e019c7e91cb07c724c614bbf644a36eff83a861b36723f29ec81dc9bdb4e68747470733a2f2f6163636f756e746e756d31312e636f6d2f454f53386d5566744a586570477a64513254614364754e7553504166584a48663232756578347534316162314556763945416857740000"} arg: {"code":"eosio","action":"regproducer","args":{"producer":"accountnum11","producer_key":"EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","url":"https://accountnum11.com/EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","location":0}} 
executed transaction: 4ebe9258bdf1d9ac8ad3821f6fcdc730823810a345c18509ac41f7ef9b278e0c  216 bytes  896 us
#         eosio <= eosio::regproducer           {"producer":"accountnum11","producer_key":"EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","u...
```

#### List the producers
To facilitate the voting process, list the available producers.

```
$ cleos system listproducers
Producer      Producer key                                           Url                                                         Scaled votes
accountnum11  EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt  https://accountnum11.com/EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22 0.0000
accountnum22  EOS5kgeCLuQo8MMLnkZfqcBA3GRFgQsPyDddHWmXceRLjRX8LJRaH  https://accountnum22.com/EOS5kgeCLuQo8MMLnkZfqcBA3GRFgQsPyD 0.0000
accountnum33  EOS63CnoyfeEQDjXXxwywN5PPKW7RYHC9tbtmvb8vFBGZooktz7kG  https://accountnum33.com/EOS63CnoyfeEQDjXXxwywN5PPKW7RYHC9t 0.0000
accountnum44  EOS6kBaCHrvz7VdUfFBLrLdhNjXYaKBmRkpDXU9PhbEUiHbspr7rz  https://accountnum44.com/EOS6kBaCHrvz7VdUfFBLrLdhNjXYaKBmRk 0.0000
```

#### Start the producers
Use the following command to start the producers.  Recall that in this tutorial, all of the producers are running on a
single server, so command line arguments are used to ensure each producer is using its own directory. This also expects
that the `genesis.json` file, e.g, see below, has been copied to the respective account directories.

```json
{
  "initial_timestamp": "2018-03-02T12:00:00.000",
  "initial_key": "EOS8Znrtgwt8TfpmbVpTKvA2oB8Nqey625CLN8bCN3TEbgx86Dsvr",
  "initial_configuration": {
    "max_block_net_usage": 1048576,
    "target_block_net_usage_pct": 1000,
    "max_transaction_net_usage": 524288,
    "base_per_transaction_net_usage": 12,
    "net_usage_leeway": 500,
    "context_free_discount_net_usage_num": 20,
    "context_free_discount_net_usage_den": 100,
    "max_block_cpu_usage": 100000,
    "target_block_cpu_usage_pct": 500,
    "max_transaction_cpu_usage": 50000,
    "min_transaction_cpu_usage": 100,
    "max_transaction_lifetime": 3600,
    "deferred_trx_expiration_window": 600,
    "max_transaction_delay": 3888000,
    "max_inline_action_size": 4096,
    "max_inline_action_depth": 4,
    "max_authority_depth": 6,
    "max_generated_transaction_count": 16
  },
  "initial_chain_id": "0000000000000000000000000000000000000000000000000000000000000000"
}
```
In a separate window for each producer, run the following `nodeos` command, adjusting the command line arguments for
each producer.
```
$ nodeos --genesis-json ~/eosio_test/accountnum11/genesis.json --block-log-dir ~/eosio_test/accountnum11/blocks --config-dir ~/eosio_test/accountnum11/ --data-dir ~/eosio_test/accountnum11/ --http-server-address 127.0.0.1:8011 --p2p-listen-endpoint 127.0.0.1:9011 --enable-stale-production --producer-name accountnum11 --private-key '[ "EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o" ]' --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin --p2p-peer-address localhost:9022  --p2p-peer-address localhost:9033  --p2p-peer-address localhost:9044 
```
Note that until all producers are up, connection error messages such as the following will be generated.
```
1826099ms thread-0   net_plugin.cpp:2927           plugin_startup       ] starting listener, max clients is 25
1826099ms thread-0   net_plugin.cpp:676            connection           ] created connection to localhost:9022
1826099ms thread-0   net_plugin.cpp:1948           connect              ] host: localhost port: 9022 
1826099ms thread-0   net_plugin.cpp:676            connection           ] created connection to localhost:9033
1826099ms thread-0   net_plugin.cpp:1948           connect              ] host: localhost port: 9033 
1826099ms thread-0   net_plugin.cpp:676            connection           ] created connection to localhost:9044
1826099ms thread-0   net_plugin.cpp:1948           connect              ] host: localhost port: 9044 
1826100ms thread-0   net_plugin.cpp:1989           operator()           ] connection failed to localhost:9022: Connection refused
1826100ms thread-0   net_plugin.cpp:1989           operator()           ] connection failed to localhost:9033: Connection refused
1826100ms thread-0   net_plugin.cpp:1989           operator()           ] connection failed to localhost:9044: Connection refused
```

For convenience, the following commands can be run (in separate shell windows) for accounts `accountnum22`,
`accountnum33`, and `accountnum44`. If you run these, you can see how multiple nodes respond
when running in a peer-to-peer configuration. This assumes that accounts have been staked using the relevant key pairs
(the key pairs can be seen in each of the command lines below).
```
$ nodeos --genesis-json ~/eosio_test/accountnum22/genesis.json --block-log-dir ~/eosio_test/accountnum22/blocks --config-dir ~/eosio_test/accountnum22/ --data-dir ~/eosio_test/accountnum22/ --http-server-address 127.0.0.1:8022 --p2p-listen-endpoint 127.0.0.1:9022 --enable-stale-production --producer-name accountnum22 --private-key '[ "EOS5kgeCLuQo8MMLnkZfqcBA3GRFgQsPyDddHWmXceRLjRX8LJRaH","5Jh4rseyguLx5Y7KE2oLL81PRmDcyyzbyyyJd3GvdHijKqENbRk" ]' --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin --p2p-peer-address localhost:9011  --p2p-peer-address localhost:9033  --p2p-peer-address localhost:9044
$ nodeos --genesis-json ~/eosio_test/accountnum33/genesis.json --block-log-dir ~/eosio_test/accountnum33/blocks --config-dir ~/eosio_test/accountnum33/ --data-dir ~/eosio_test/accountnum33/ --http-server-address 127.0.0.1:8033 --p2p-listen-endpoint 127.0.0.1:9033 --enable-stale-production --producer-name accountnum33 --private-key '[ "EOS63CnoyfeEQDjXXxwywN5PPKW7RYHC9tbtmvb8vFBGZooktz7kG","5JWp5K24x5dbAFBNP7hwzSRS7XjD7wjHL4nrAwSLdRuJnERjgqB" ]' --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin --p2p-peer-address localhost:9011  --p2p-peer-address localhost:9022  --p2p-peer-address localhost:9044
$ nodeos --genesis-json ~/eosio_test/accountnum44/genesis.json --block-log-dir ~/eosio_test/accountnum44/blocks --config-dir ~/eosio_test/accountnum44/ --data-dir ~/eosio_test/accountnum44/ --http-server-address 127.0.0.1:8044 --p2p-listen-endpoint 127.0.0.1:9044 --enable-stale-production --producer-name accountnum44 --private-key '[ "EOS6kBaCHrvz7VdUfFBLrLdhNjXYaKBmRkpDXU9PhbEUiHbspr7rz","5KeGoDkbhEdkZTpYqQg2rPZvtxqfWAtGgixuCLUt1Dmoq4NmXCj" ]' --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin --p2p-peer-address localhost:9011  --p2p-peer-address localhost:9022  --p2p-peer-address localhost:9033
```

#### Vote for producers
Voting for producers can begin as accounts are staked and producers are registered.

The following command enables votes to be cast.
```
$ cleos system voteproducer prods accountnum23 accountnum11 accountnum33
```
In this example, account `accountnum23` has voted for `accountnum11` and `accountnum33`. Block production does not begin
until 15% of the available votes have been voted.

