Tools/Programs included in eosio respository. 

* [Programs](#programs)
    * [nodeos](#nodeos)
    * [cleos](#cleos)
    * [keosd](#keosd)
    * [launcher](#launcher)
* [Tools](#tools)
    * [eosiocpp](#eosiocpp)

## Programs

### nodeos

The core EOSIO daemon that can be configured with plugins to run a node. Example uses are block production, dedicated API endpoints, and local development. 

### cleos

`cleos` is a command line tool that interfaces with the REST API exposed by `nodeos`. In order to use `cleos` you will need to have the end point (IP address and port number) to a `nodeos` instance and also configure `cleos` to load the 'eosio::chain_api_plugin'.  `cleos` contains documentation for all of its commands. For a list of all commands known to `cleos`, simply run it with no arguments:

```base
Command Line Interface to EOSIO Client
Usage: ./programs/cleos/cleos [OPTIONS] SUBCOMMAND

Options:
  -h,--help                   Print this help message and exit
  -u,--url TEXT=http://localhost:8888/
                              the http/https URL where nodeos is running
  --wallet-url TEXT=http://localhost:8900/
                              the http/https URL where keosd is running
  -r,--header                 pass specific HTTP header; repeat this option to pass multiple headers
  -n,--no-verify              don't verify peer certificate when using HTTPS
  -v,--verbose                output verbose actions on error

Subcommands:
  version                     Retrieve version information
  create                      Create various items, on and off the blockchain
  get                         Retrieve various items and information from the blockchain
  set                         Set or update blockchain state
  transfer                    Transfer EOS from account to account
  net                         Interact with local p2p network connections
  wallet                      Interact with local wallet
  sign                        Sign a transaction
  push                        Push arbitrary transactions to the blockchain
  multisig                    Multisig contract commands
  system                      Send eosio.system contract action to the blockchain.
```

To get help with any particular subcommand, run it with no arguments as well:
```base
Create various items, on and off the blockchain
Usage: ./cleos create SUBCOMMAND

Subcommands:
  key                         Create a new keypair and print the public and private keys
  account                     Create an account, buy ram, stake for bandwidth for the account


Create an account, buy ram, stake for bandwidth for the account
Usage: ./programs/cleos/cleos create account [OPTIONS] creator name OwnerKey [ActiveKey]

Positionals:
  creator TEXT                The name of the account creating the new account (required)
  name TEXT                   The name of the new account (required)
  OwnerKey TEXT               The owner public key for the new account (required)
  ActiveKey TEXT              The active public key for the new account

Options:
  -h,--help                   Print this help message and exit
  -x,--expiration             set the time in seconds before a transaction expires, defaults to 30s
  -f,--force-unique           force the transaction to be unique. this will consume extra bandwidth and remove any protections against accidently issuing the same transaction multiple times
  -s,--skip-sign              Specify if unlocked wallet keys should be used to sign transaction
  -j,--json                   print result as json
  -d,--dont-broadcast         don't broadcast transaction to the network (just print to stdout)
  -r,--ref-block TEXT         set the reference block num or block id used for TAPOS (Transaction as Proof-of-Stake)
  -p,--permission TEXT ...    An account and permission level to authorize, as in 'account@permission'
  --max-cpu-usage-ms UINT     set an upper limit on the milliseconds of cpu usage budget, for the execution of the transaction (defaults to 0 which means no limit)
  --max-net-usage UINT        set an upper limit on the net usage budget, in bytes, for the transaction (defaults to 0 which means no limit)
```
### keosd

An EOSIO wallet daemon that loads wallet related plugins, such as the HTTP interface and RPC API

### launcher

The launcher application simplifies the distribution of multiple nodeos nodes across a LAN or a wider network. It can be configured via CLI to compose per-node configuration files, distribute these files securely amongst the peer hosts and then start up the multiple instances of nodeos.

## Tools

### eosiocpp

#### Using eosiocpp to generate the ABI specification file

**eosiocpp** can generate the ABI specification file by inspecting the content of types declared in the contract source code.

To indicate that a type must be exported to the ABI (as an action or a table), the **@abi** annotation must be used in the **comment attached to the type declaration**.

The syntax for the annotation is as following.

- @abi action [name name2 ... nameN]
- @abi table [index_type name]
To generate the ABI file, **eosiocpp** must be called with the **-g** option.

```base
➜ eosiocpp -g abi.json types.hpp
Generated abi.json ...
```

**eosiocpp** can also be used to generate helper functions that serialize/deserialize the types defined in the ABI spec.
```base
➜ eosiocpp -g abi.json -gs types.hpp
Generated abi.json ...
Generated types.gen.hpp ...
```

#### Examples

#### Declaring an action
```base
#include <eosiolib/eosio.hpp>

class example : public eosio::contract {
   //@abi action
   void exampleaction( uint64_t param1, uint64_t param2, std::string param3 ) {
   }
};

{
  "types": [],
  "structs": [{
      "name": "exampleaction",
      "base": "",
      "fields": [{
           "name": "param1",
           "type": "uint64"
        },{
           "name": "param2",
           "type": "uint64"
        },{        
           "name": "param3",
           "type": "string"
        }
      ]
    }
  ],
  "actions": [{
      "name": "exampleaction",
      "type": "exampleaction",
      "ricardian_contract": ""
    }
  ],
  "tables": [],
  "ricardian_clauses": [],
  "abi_extensions": []
}
```

### Declaring a table

```base
#include <eosiolib/eosio.hpp>

//@abi table my_table
struct my_record {
  uint64_t    ssn;
  std::string fullname;
  uint64_t primary_key() const { return key; }
};
{
  "types": [],
  "structs": [{
      "name": "my_record",
      "base": "",
      "fields": [{
        "name": "ssn",
        "type": "uint64"
      },{
        "name": "fullname",
        "type": "string"
      }
      ]
    }
  ],
  "actions": [],
  "tables": [{
      "name": "my_table",
      "index_type": "i64",
      "key_names": [
        "ssn"
      ],
      "key_types": [
        "uint64"
      ],
      "type": "my_record"
    }
  ],
  "ricardian_clauses": [],
  "abi_extensions": []
}
```

### Example of typedef exporting
```base
#include <eosiolib/eosio.hpp>
struct simple {
  uint64_t u64;
};

typedef simple simple_alias;
typedef eosio::name name_alias;

class examplecontract : eosio::contract {
//@abi action
   void actionone( uint32_t param1, name_alias param2, simple_alias param3 ) {}
};
{
  "types": [{
      "new_type_name": "simple_alias",
      "type": "simple"
    },{
      "new_type_name": "name_alias",
      "type": "name"
    }
  ],
  "structs": [{
      "name": "simple",
      "base": "",
      "fields": [{
        "type": "uint64",
        "name": "u64"
      }]
    },{
      "name": "actionone",
      "base": "",
      "fields": [{
        "type": "uint32",
        "name": "param1"
      },{
        "type": "name_alias",
        "name": "param2"
      },{
        "type": "simple_alias",
        "name": "param3"
      }
     ]
    }
  ],
  "actions": [{
      "name": "actionone",
      "type": "actionone",
      "ricardian_contract": ""
    }
  ],
  "tables": [],
  "ricardian_clauses": [],
  "abi_extensions": []
}
```

#### Using the generated serialization/deserialization functions and explicit user defined apply
```base
#include <eosiolib/eosio.hpp>

struct simple {
  uint32_t u32;
};

struct my_complex_type {
  uint64_t u64;
  std::string str;
  simple simple;
  eosio::bytes bytes;
  public_key pub;
};

typedef my_complex_type complex;

//@abi action
struct test_action {
  uint32_t u32;
  complex cplx;
};

extern "C" {
   void apply( uint64_t code, uint64_t action, uint64_t receiver ) {
      if( code == N(mycontract) ) {
         if( action == N(testaction) ) {
            eosio::print("test_action content\n");
            test_action testact = eosio::unpack_action_data<test_action>();
            eosio::print_f( "Test action : % %", testact.u32, testact.cplx.u64 );
        }
     }
   }
}

```
### NOTE: table names and action names cannot use an underscore ("_").
### Calling contract with test values
```base
cleos push action testaccount testaction '{"u32":"1000", "cplx":{"u64":"472", "str":"hello", "bytes":"B0CA", "pub":"EOS8CY2pCW5THmzvPTgEh5WLEAxgpVFXaPogPvgvVpVWCYMRdzmwx", "simple":{"u32":"123"}}}' -p testaccount
```