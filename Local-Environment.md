## Contents
*  [Overview](Local-Environment#eosio-overview)
*  [Getting the Code](Local-Environment#1-getting-the-code)
*  [Building EOSIO](Local-Environment#2-building-eosio)
    * [Automated build script](Local-Environment#autobuild)
    * [Build validation](Local-Environment#basicvalidation)
    * [Install the executables](Local-Environment#installexecutables) 
*  [Creating and Launching a Single Node Testnet](Local-Environment#4-creating-and-launching-a-single-node-testnet)
* [Docker](Local-Environment#3-docker)
* [Troubleshooting Guide](Local-Environment#5-troubleshooting-guide)
    

<a name="eosio-overview"></a>
## Overview

EOSIO comes with a number of programs.  The primary ones that you will use, and the ones that are covered here, are:

* `nodeos` - server-side blockchain node component
* `cleos` - command line interface to interact with the blockchain and to manage wallets
* `keosd` - component that manages EOSIO wallets

The basic relationship between these components is illustrated in the following diagram.  In the sections that follow, you will build the EOSIO components, and deploy them in a single host, single node test network (testnet) configuration. 
![Basic Relationships Between EOSIO Components](assets/Basic-EOSIO-System-Architecture.png)


<a name="1-getting-the-code"></a>
## Getting the Code

To download all of the code, clone the `eos` repository and its submodules.

```bash
git clone https://github.com/EOSIO/eos --recursive
```

If a repository is cloned without the `--recursive` flag, the submodules can be retrieved after the fact by running this command from within the repo:

```bash
git submodule update --init --recursive
```

Throughout the EOSIO documentation and tutorials, reference will be made to the top level of your local EOSIO source repository.  The location where you just cloned the `eos` repository is that location.  The notation ${EOSIO_SOURCE} represents the same thing.  For example, if you ran the `git clone` operation in a folder called `~/myprojects`, then `${EOSIO_SOURCE}=~/myprojects/eos`.  _**Note that ${EOSIO_SOURCE} is used in the documentation for notational purposes only.  No environment variable is implied or required.**_ More commonly, for simplicity, this and other documents might simply refer to `eos`.  This is equivalent to `${EOSIO_SOURCE}`.

<a name="2-building-eosio"></a>
## Building EOSIO
Building EOSIO is done via an automated build script.  The build places content in the `eos/build` folder.  The executables can be found in subfolders within the `eos/build/programs` folder.

<a name="autobuild"></a>
### Automated build script

There is an automated build script that can install all dependencies and build EOSIO.  The script supports the following operating systems. We are working on supporting other Linux/Unix distributions in future releases.

1. Amazon 2017.09 and higher.  
2. Centos 7.  
3. Fedora 25 and higher (Fedora 27 recommended).  
4. Mint 18.  
5. Ubuntu 16.04 (Ubuntu 16.10 recommended).  
6. MacOS Darwin 10.12 and higher (MacOS 10.13.x recommended).  

### System Requirements (all platforms)
- 8GB RAM free required
- 20GB Disk free required

### Run the build script
Run the build script from the `eos` folder.

```bash
cd eos
./eosio_build.sh
```

<a name="basicvalidation"></a>
### Build validation 

Optionally, a set of tests can be run against your build to perform some basic validation.  To run the test suite after building, start `mongod` and the run `make test`.

On Linux platforms:
```bash
~/opt/mongodb/bin/mongod -f ~/opt/mongodb/mongod.conf &
```

On MacOS:
```bash
/usr/local/bin/mongod -f /usr/local/etc/mongod.conf &
```

Followed by this on all platforms:
```bash
cd build
make test
```

<a name="installexecutables"></a>
### Install the executables 
 
For ease of contract development, content can be installed in the `/usr/local` folder using the `make install` target.  This step is run from the `build` folder.  Adequate permission is required to install, so we use the `sudo` command with `make install`.

```bash
cd build
sudo make install
```

<a name="4-creating-and-launching-a-single-node-testnet"></a>
## Creating and Launching a Single Node Testnet

After successfully building the project, the `nodeos` binary should be present in the `build/programs/nodeos` folder.  `nodeos` can be run directly from the `build` folder using `programs/nodeos/nodeos`, or you can `cd build/programs/nodeos` to change into the folder and run the `nodeos` command from there.  Here, we run the command within the `build/programs/nodeos` folder.

You can start your own single-node blockchain with this single command:

```
cd build/programs/nodeos
./nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::account_history_api_plugin 
```

When running `nodeos` you should get log messages similar to below. It means the blocks are successfully produced.

```
1575001ms thread-0   chain_controller.cpp:235      _push_block          ] initm #1 @2017-09-04T04:26:15  | 0 trx, 0 pending, exectime_ms=0
1575001ms thread-0   producer_plugin.cpp:207       block_production_loo ] initm generated block #1 @ 2017-09-04T04:26:15 with 0 trxs  0 pending
1578001ms thread-0   chain_controller.cpp:235      _push_block          ] initc #2 @2017-09-04T04:26:18  | 0 trx, 0 pending, exectime_ms=0
1578001ms thread-0   producer_plugin.cpp:207       block_production_loo ] initc generated block #2 @ 2017-09-04T04:26:18 with 0 trxs  0 pending
...
eosio generated block 046b9984... #101527 @ 2018-04-01T14:24:58.000 with 0 trxs
eosio generated block 5e527ee2... #101528 @ 2018-04-01T14:24:58.500 with 0 trxs
...
```
At this point, `nodeos` is running with a single producer, `eosio`.

The following diagram depicts the single host testnet that we just created.  For the curious, in this configuration, we chose to use `nodeos` for wallet management. The `eosio::wallet_api_plugin` that we specified on the `nodeos` command line has a dependency on `eosio::wallet_plugin` that does the wallet management, causing it to load automatically.  Other alternatives are discussed in the [Comprehensive Accounts & Wallets Tutorial](Tutorial-Comprehensive-Accounts-and-Wallets) tutorial.  Regardless of configuration chosen, `cleos` is used to manage the wallets, manage the accounts, and invoke actions on the blockchain.
![Single Node Testnet without `keosd`](assets/Single-Host-Testnet-No-keosd.png)

> ### Advanced Steps
> The more advanced user will likely have need to modify the configuration.  `nodeos` uses a custom configuration folder.  The location of this folder is determined by your system.
> 
> - Mac OS: `~/Library/Application Support/eosio/nodeos/config`
> - Linux: `~/.local/share/eosio/nodeos/config`
> 
> The build seeds this folder with a default `genesis.json` file.  A configuration folder can be specified using the `--config-dir` command line argument to `nodeos`.  If you use this option, you will need to manually copy a `genesis.json` file to your config folder.
> 
> `nodeos` will need a properly configured `config.ini` file in order to do meaningful work.  On startup, `nodeos` looks in the config folder for `config.ini`.  If one is not found, a default `config.ini` file is created.  If you do not
> already have a `config.ini` file ready to use, run `nodeos` and then close it immediately with <kbd>Ctrl-C</kbd>.  A default configuration (`config.ini`) will have been created in the config folder.  Edit the `config.ini` file, adding/updating the following settings to the defaults already in place:
> 
> ```
>    # Load the testnet genesis state, which creates some initial block producers with the default key
>    genesis-json = /path/to/eos/source/genesis.json
>    # Enable production on a stale chain, since a single-node test chain is pretty much always stale
>    enable-stale-production = true
>    # Enable block production with the testnet producers
>    producer-name = eosio
>    # Load the block producer plugin, so you can produce blocks
>    plugin = eosio::producer_plugin
>    # Wallet plugin
>    plugin = eosio::wallet_api_plugin
>    # As well as API and HTTP plugins
>    plugin = eosio::chain_api_plugin
>    plugin = eosio::http_plugin
>    # This will be used by the validation step below, to view account history
>    plugin = eosio::account_history_api_plugin
> ```
> 
> Now it should be possible to run `nodeos` and see it begin producing blocks.
> ```bash
> programs/nodeos/nodeos
> ```
> 
> `nodeos` stores runtime data (e.g., shared memory and log content) in a custom data folder.  The location of this folder is determined by your system.
> 
> - Mac OS: `~/Library/Application Support/eosio/nodeos/data`
> - Linux: `~/.local/share/eosio/nodeos/data`
> 
> A data folder can be specified using the `--data-dir` command line argument to `nodeos`.

<a name="3-docker"></a>
## Docker

Simple and fast setup of EOSIO on Docker is also available.  You can find up to date information about EOSIO Docker in the [Docker Readme](https://github.com/EOSIO/eos/blob/master/Docker/README.md).
                                                                   

### Install Dependencies
 - [Docker](https://docs.docker.com) Docker 17.05 or higher is required

### Build EOSIO image

```bash
$ git clone https://github.com/EOSIO/eos.git --recursive
$ cd eos/Docker
$ docker build . -t eosio/eos
```

### Start nodeos docker container only

```bash
$ docker run --name nodeos -p 8888:8888 -p 9876:9876 -t eosio/eos nodeosd.sh arg1 arg2
```

By default, all data is persisted in a docker volume. It can be deleted if the data is outdated or corrupted:
``` bash
$ docker inspect --format '{{ range .Mounts }}{{ .Name }} {{ end }}' nodeos
fdc265730a4f697346fa8b078c176e315b959e79365fc9cbd11f090ea0cb5cbc
$ docker volume rm fdc265730a4f697346fa8b078c176e315b959e79365fc9cbd11f090ea0cb5cbc
```

Alternately, you can directly mount host directory into the container
```bash
$ docker run --name nodeos -v /path-to-data-dir:/opt/eos/bin/data-dir -p 8888:8888 -p 9876:9876 -t eosio/eos nodeosd.sh arg1 arg2
```

### Get chain info

```bash
$ curl http://127.0.0.1:8888/v1/chain/get_info
```

### Start both nodeos and keosd containers

```bash
$ docker-compose up
```

After `docker-compose up`, two services named nodeos and keosd will be started. nodeos service will expose ports 8888 and 9876 to the host. keosd service does not expose any port to the host, it is only accessible to cleos when runing cleos is running inside the keosd container as described in "Execute cleos commands" section.


#### Execute cleos commands

You can run the `cleos` commands via a bash alias.

```bash
$ alias cleos='docker-compose exec keosd /opt/eos/bin/cleos -H nodeos'
$ cleos get info
$ cleos get account inita
```

Upload sample exchange contract

```bash
$ cleos set contract exchange contracts/exchange/exchange.wast contracts/exchange/exchange.abi
```

If you don't need keosd afterwards, you can stop the keosd service using

```bash
$ docker-compose stop keosd
```
#### Change default configuration

You can use docker compose override file to change the default configurations. For example, create an alternate config file `config2.ini` and a `docker-compose.override.yml` with the following content.

```yaml
version: "2"

services:
  nodeos:
    volumes:
      - nodeos-data-volume:/opt/eos/bin/data-dir
      - ./config2.ini:/opt/eos/bin/data-dir/config.ini
```

Then restart your docker containers as follows:

```bash
$ docker-compose down
$ docker-compose up
```

#### Clear data-dir
The data volume created by docker-compose can be deleted as follows:

```bash
$ docker volume rm docker_nodeos-data-volume
```


<a name="5-troubleshooting-guide"></a>
## Troubleshooting Guide

1. *You get an error such as `St9exception: content of memory does not match data expected by executable` when trying to start `nodeos`*
> Try restarting `nodeos` with `--resync`
2. How do I find which version of `nodeos` I'm running or connecting to?
> Use `cleos -H ${nodeos_host} -p ${nodeos_port} get info` and you will see the version number in the field called `server_version`
