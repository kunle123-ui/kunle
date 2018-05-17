
This tutorial describes how to set up a multi-node blockchain configuration running on a single host.  This is referred
to as a _**single host, multi-node testnet**_.  We will set up two nodes on your local computer and have them communicate
with each other.  The examples in this section rely on three command-line applications, `nodeos`, `keosd`, and `cleos`. 
The following diagram depicts the desired testnet configuration.
![Single Host, Multi-Node Testnet](assets/Single-Host-Multi-Node-Testnet.png)

It is assumed that `keosd`, `cleos`, and `nodeos` have been installed in your path, or that you know how to start these
applications from the location in the file system.  (See [Setting Up A Local Environment](Local-Environment))

Open three "terminal" windows to perform the steps in this tutorial.

### Create a Default Wallet
In the next terminal window, use `cleos`, the command-line utility, to create the default wallet.
```
cleos wallet create
```

`cleos` will indicate that it created the "default" wallet, and will provide a password for future wallet access. As the
message says, be sure to preserve this password for future use. Here is an example of this output:
```
cleos wallet create 
"/usr/local/bin/keosd" launched
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5K8UL1WFpRuMnuSe7VpB78CoNHu3PidALNrJsmhbcbafwvycWSS"
```

We will continue to use this window for subsequent `cleos` commands.

_Note that `keosd` was automatically launched. This instance is running on the default wallet port (8900) and writing to
the default wallet directory (~/eosio-wallet)._

### Start the First Producer Node
We can now start the first producer node. In the second terminal window run:
```
nodeos --enable-stale-production --producer-name eosio --plugin eosio::chain_api_plugin --plugin eosio::net_api_plugin
```

This creates a special producer, known as the "bios" producer. Assuming everything has executed correctly to this point,
you should see output from the `nodeos` process reporting block creation.

### Start the Second Producer Node
The following commands assume that you are running this tutorial from your `${EOSIO_SOURCE}` directory, from which you
ran `./eosio_build.sh` to build.  See [Getting the Code](Local-Environment#1-getting-the-code) for more information if
this is not clear.

To start additional nodes, you must first load the `eosio.bios` contract. This contract enables you to have direct control
over the resource allocation of other accounts and to access other privileged API calls. Return to the second terminal
window and run the following command to load the contract:
```
cleos set contract eosio build/contracts/eosio.bios
```
You should see output similar to the following:
``` bash
Reading WAST/WASM from build/contracts/eosio.bios/eosio.bios.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: d08a017c20398d3d9396ada3eb6a5efbc9f0491c1b5c95d01627bea1b5e2b7e0  3256 bytes  802 us
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d01000000015c1160037f7e7f0060057f7e7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
```

We will create an account to become a producer, using the account name `producernde1`.  To create the account, we need to
generate keys to associate with the account, and import those into our wallet.

Run the create key command:
```
cleos create key
```
This will report newly generated public and private keypairs that will look similar to the following.

**IMPORTANT:  The command line instructions that follow use the keys shown below.  In order to be able to cut-and-paste
the command line instructions directly from this tutorial, do NOT use the keys that you just generated.  If you want to
use your newly generated keys, you will need to replace the key values with yours in the commands.**
```
Private key: 5J2Hihw5VhnD3qsQYvegRsQixy55iB424PyqCLau4RrSxZNjrWp
Public key: EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai
```

Now import the private key portion into your wallet. If successful, the matching public key will be reported. This should
match the previously generated public key:

``` bash
cleos wallet import 5J2Hihw5VhnD3qsQYvegRsQixy55iB424PyqCLau4RrSxZNjrWp
imported private key for: EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai
```

Create the `producernde1` account that we will use to become a producer. The `create account` command requires two public
keys, one for the account's owner key and one for its active key.  In this example, the newly created public key is used
twice, as both the owner key and the active key. Example output from the create command is shown:

``` bash
cleos create account eosio producernde1 EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai
executed transaction: 3bfe642201152082a56cea776f049315fe224f662c5a5a1dff66c31292c4fac7  200 bytes  215 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"producernde1","owner":{"threshold":1,"keys":[{"key":"EOS5wXTc92qD3DgSgdfi...
```
We now have an account that is available to have a contract assigned to it, enabling it to do meaningful work. In other
tutorials, the account has been used to establish simple contracts. In this case, the account will be designated as a
block producer.

In the third terminal window, start a second `nodeos` instance. Notice that this command line is substantially longer
than the one we used above to create the first producer. This is necessary to avoid collisions with the first `nodeos`
instance. Fortunately, you can just cut and paste this command line and adjust the keys:

```
nodeos --producer-name producernde1 --plugin eosio::chain_api_plugin --plugin eosio::net_api_plugin --http-server-address 127.0.0.1:8889 --p2p-listen-endpoint 127.0.0.1:9877 --p2p-peer-address 127.0.0.1:9876 --config-dir node2 --data-dir node2 --private-key [\"EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai\",\"5J2Hihw5VhnD3qsQYvegRsQixy55iB424PyqCLau4RrSxZNjrWp\"]
```

The output from this new node will show a little activity but will stop reporting until the last step in this tutorial,
when the `producernde1` account is registered as a producer account and activated. Here is some example output from a newly
started node. Your output might look a little different, depending on how much time you took entering each of these commands.
Furthermore, this example is only a few lines of output:

```
2521356ms thread-0   net_plugin.cpp:2921           plugin_startup       ] starting listener, max clients is 25
2521356ms thread-0   net_plugin.cpp:676            connection           ] created connection to 127.0.0.1:9876
2521356ms thread-0   net_plugin.cpp:1948           connect              ] host: 127.0.0.1 port: 9876 
2521356ms thread-0   net_api_plugin.cpp:69         plugin_startup       ] starting net_api_plugin
2521356ms thread-0   http_plugin.cpp:325           add_handler          ] add api url: /v1/net/connect
2521356ms thread-0   http_plugin.cpp:325           add_handler          ] add api url: /v1/net/connections
2521356ms thread-0   http_plugin.cpp:325           add_handler          ] add api url: /v1/net/disconnect
2521356ms thread-0   http_plugin.cpp:325           add_handler          ] add api url: /v1/net/status
2521356ms thread-0   producer_plugin.cpp:381       plugin_startup       ] producer plugin:  plugin_startup() begin
2521356ms thread-0   producer_plugin.cpp:388       plugin_startup       ] Launching block production for 1 producers.
2521356ms thread-0   producer_plugin.cpp:401       plugin_startup       ] producer plugin:  plugin_startup() end
2521719ms thread-0   producer_plugin.cpp:213       on_incoming_block    ] Received block 000003e8f45db37b... #1000 @ 2018-05-17T00:40:05.500 signed by eosio [trxs: 0, lib: 999, confirmed: 0]
2521800ms thread-0   producer_plugin.cpp:213       on_incoming_block    ] Received block 000004c797082035... #1223 @ 2018-05-17T00:41:57.000 signed by eosio [trxs: 0, lib: 1222, confirmed: 0]
2521800ms thread-0   producer_plugin.cpp:213       on_incoming_block    ] Received block 000004c89f62eef1... #1224 @ 2018-05-17T00:41:57.500 signed by eosio [trxs: 0, lib: 1223, confirmed: 0]
```

At this point, the second `nodeos` is an idle producer. To turn it into an active producer, `producernde1` needs to be
registered as a producer with the bios node, and the bios node needs to perform an action to update the producer schedule.

``` bash
cleos push action eosio setprods "{ \"schedule\": [{\"producer_name\": \"producernde1\",\"block_signing_key\": \"EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai\"}]}" -p eosio@active
executed transaction: 46f7a8333c557392f78eefd193abd8c33778e7a57d8bf869c01ad1a353cd44b1  136 bytes  269 us
#         eosio <= eosio::setprods              {"schedule":[{"producer_name":"producernde1","block_signing_key":"EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCu...
```
On the original node, you should see output similar to the following, showing that the node is no longer producing but is 
now receiving.
```
2688500ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000061e624645a2... #1566 @ 2018-05-17T00:44:48.500 signed by eosio [trxs: 0, lib: 1565, confirmed: 0]
2689004ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000061f24f2f7c5... #1567 @ 2018-05-17T00:44:49.000 signed by eosio [trxs: 1, lib: 1566, confirmed: 0]
2689004ms thread-0   controller.cpp:679            start_block          ] promoting proposed schedule (set in block 1567) to pending; current block: 1568 lib: 1567 schedule: {"version":1,"producers":[{"producer_name":"producernde1","block_signing_key":"EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai"}]} 
2689505ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 000006204d14f394... #1568 @ 2018-05-17T00:44:49.500 signed by eosio [trxs: 0, lib: 1567, confirmed: 0]
2690005ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 00000621f9736261... #1569 @ 2018-05-17T00:44:50.000 signed by eosio [trxs: 0, lib: 1568, confirmed: 0]
2690506ms thread-0   producer_plugin.cpp:213       on_incoming_block    ] Received block 00000622f381232b... #1570 @ 2018-05-17T00:44:50.500 signed by producernde1 [trxs: 0, lib: 1569, confirmed: 0]
2691005ms thread-0   producer_plugin.cpp:213       on_incoming_block    ] Received block 00000623f7677528... #1571 @ 2018-05-17T00:44:51.000 signed by producernde1 [trxs: 0, lib: 1570, confirmed: 0]
```
On the second node, you should see output similar to the following, showing that it is now producing.
``` 
2688003ms thread-0   producer_plugin.cpp:213       on_incoming_block    ] Received block 0000061dad1b516f... #1565 @ 2018-05-17T00:44:48.000 signed by eosio [trxs: 0, lib: 1564, confirmed: 0]
2688501ms thread-0   producer_plugin.cpp:213       on_incoming_block    ] Received block 0000061e624645a2... #1566 @ 2018-05-17T00:44:48.500 signed by eosio [trxs: 0, lib: 1565, confirmed: 0]
2689004ms thread-0   producer_plugin.cpp:213       on_incoming_block    ] Received block 0000061f24f2f7c5... #1567 @ 2018-05-17T00:44:49.000 signed by eosio [trxs: 1, lib: 1566, confirmed: 0]
2689004ms thread-0   controller.cpp:679            start_block          ] promoting proposed schedule (set in block 1567) to pending; current block: 1568 lib: 1567 schedule: {"version":1,"producers":[{"producer_name":"producernde1","block_signing_key":"EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai"}]} 
2689004ms thread-0   controller.cpp:679            start_block          ] promoting proposed schedule (set in block 1567) to pending; current block: 1568 lib: 1567 schedule: {"version":1,"producers":[{"producer_name":"producernde1","block_signing_key":"EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai"}]} 
2689503ms thread-0   controller.cpp:679            start_block          ] promoting proposed schedule (set in block 1567) to pending; current block: 1568 lib: 1567 schedule: {"version":1,"producers":[{"producer_name":"producernde1","block_signing_key":"EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai"}]} 
2689505ms thread-0   controller.cpp:679            start_block          ] promoting proposed schedule (set in block 1567) to pending; current block: 1568 lib: 1567 schedule: {"version":1,"producers":[{"producer_name":"producernde1","block_signing_key":"EOS5wXTc92qD3DgSgdfiDXxSFacpBGpfCuKfSuHXsgRVsVH1LJPai"}]} 
2689506ms thread-0   producer_plugin.cpp:213       on_incoming_block    ] Received block 000006204d14f394... #1568 @ 2018-05-17T00:44:49.500 signed by eosio [trxs: 0, lib: 1567, confirmed: 0]
2690006ms thread-0   producer_plugin.cpp:213       on_incoming_block    ] Received block 00000621f9736261... #1569 @ 2018-05-17T00:44:50.000 signed by eosio [trxs: 0, lib: 1568, confirmed: 0]
2690505ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 00000622f381232b... #1570 @ 2018-05-17T00:44:50.500 signed by producernde1 [trxs: 0, lib: 1569, confirmed: 0]
2691005ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 00000623f7677528... #1571 @ 2018-05-17T00:44:51.000 signed by producernde1 [trxs: 0, lib: 1570, confirmed: 0]
2691505ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 000006247aaaad54... #1572 @ 2018-05-17T00:44:51.500 signed by producernde1 [trxs: 0, lib: 1571, confirmed: 0]
```

Congratulations, you have now configured a two-node testnet! You can see that the original node is no longer producing
blocks but it is receiving them. You can verify this by running the `get info` commmand against each node.

Get info about the first node:
```
cleos get info
```

This should produce output that looks similar to this:
```
{
  "server_version": "d624664b",
  "head_block_num": 2103,
  "last_irreversible_block_num": 2102,
  "last_irreversible_block_id": "000008362319b91789c24d42937b9206c54c5a832b674248defebbe1cd9095d2",
  "head_block_id": "00000837e0c161727830b75deddcda24d44cdb468911e72cff1dff63511c777a",
  "head_block_time": "2018-05-17T00:49:17",
  "head_block_producer": "producernde1",
  "virtual_block_cpu_limit": 815566,
  "virtual_block_net_limit": 8585466,
  "block_cpu_limit": 99900,
  "block_net_limit": 1048576
}
```
Now for the second node:
```
cleos -u http://localhost:8889 get info
```

This should produce output that looks similar to this:
```
{
  "server_version": "d624664b",
  "head_block_num": 2153,
  "last_irreversible_block_num": 2152,
  "last_irreversible_block_id": "0000086818df0f7cff031c63db63e55fcaa96fb3eb2f25dfba4c57d6995363ec",
  "head_block_id": "00000869c8c0df26e8a50e05a36c1517753f338310d3d5e27f7023ee32b7b523",
  "head_block_time": "2018-05-17T00:49:42",
  "head_block_producer": "producernde1",
  "virtual_block_cpu_limit": 857378,
  "virtual_block_net_limit": 9025851,
  "block_cpu_limit": 99900,
  "block_net_limit": 1048576
}
```

In a later tutorial we will explore how to use more advanced tools to run a multi-host, multi-node testnet.


