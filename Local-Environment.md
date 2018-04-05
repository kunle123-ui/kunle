## Contents
*  [Getting the code](Local-Environment#1-getting-the-code)
*  [Building EOSIO](Local-Environment#2-building-eosio)
    * [Automated build script](Local-Environment#autobuild)
    * [Manual build](Local-Environment#manualbuild)
    * [Build validation](Local-Environment#basicvalidation)
    * [Install the executables](Local-Environment#installexecutables) 
    * [Manual installation of the dependencies](Local-Environment#manualdep)
       * [Clean install Amazon 2017.09 and higher](Local-Environment#manualdepamazon)
       * [Clean install Centos 7 and higher](Local-Environment#manualdepcentos)
       * [Clean install Fedora 25 and higher](Local-Environment#manualdepfedora)
       * [Clean install Ubuntu 16.04 & Linux Mint 18](Local-Environment#manualdepubuntu)
       * [Clean install MacOS Sierra 10.12.6 & higher](Local-Environment#manualdepmacos)
* [Docker](Local-Environment#3-docker)
* [Creating and Launching a Single Node Testnet](Local-Environment#4-creating-and-launching-a-single-node-testnet)
    * [Validate the Environment - "Currency" Contract Walkthrough](Local-Environment#validatewithcurrencycontract)
* [Troubleshooting Guide](Local-Environment#5-troubleshooting-guide)
    

<a name="1-getting-the-code"></a>
## Getting the code

To download all of the code, clone the eos repository and its submodules.

```bash
git clone https://github.com/EOSIO/eos --recursive
```

If a repository is cloned without the `--recursive` flag, the submodules can be retrieved after the fact by running this command from within the repo:

```bash
git submodule update --init --recursive
```

<a name="2-building-eosio"></a>
## Building EOSIO

EOSIO comes with a number of programs:

* nodeos - server-side blockchain node component
* cleos - command line interface to interact with the blockchain
* keosd - EOSIO wallet
* eosio-launcher - application to assist with deploying a multi-node blockchain network; [more on eosio-launcher](https://github.com/EOSIO/eos/blob/master/testnet.md)

The build places content in the `eos/build` folder, where `eos` is the top level of your cloned repository.  The executables can be found in subfolders within the `eos/build/programs` folder.

<a name="autobuild"></a>
### Automated build script

There is an automated build script that can install all dependencies and build EOSIO.  The script supports the following operating systems.  
We are working on supporting other Linux/Unix distributions in future releases.

1. Amazon 2017.09 and higher.  
2. Centos 7.  
3. Fedora 25 and higher (Fedora 27 recommended).  
4. Mint 18.  
5. Ubuntu 16.04 (Ubuntu 16.10 recommended).  
6. MacOS Darwin 10.12 and higher (MacOS 10.13.x recommended).  

Run the build script from the `eos` folder.

```bash
cd eos
./eosio_build.sh
```

<a name="manualbuild"></a>
### Manual build

To manually build, use the following steps to create a `build` folder within your `eos` folder and then perform the build.  The steps below assume the `eos` repository was cloned into your home (i.e., `~`) folder.  It is also assumes that the necessary dependencies have been installed.  See [Manual Installation of the Dependencies](#manualdep).

```bash
cd ~
mkdir -p ~/eos/build && cd ~/eos/build
```

On Linux platforms, use this `cmake` command:
``` bash
cmake -DBINARYEN_BIN=~/binaryen/bin -DWASM_ROOT=~/wasm-compiler/llvm -DOPENSSL_ROOT_DIR=/usr/local/opt/openssl -DOPENSSL_LIBRARIES=/usr/local/opt/openssl/lib -DBUILD_MONGO_DB_PLUGIN=true ..
```

On MacOS, use this `cmake` command:
``` bash
cmake -DBINARYEN_BIN=~/binaryen/bin -DWASM_ROOT=/usr/local/wasm -DOPENSSL_ROOT_DIR=/usr/local/opt/openssl -DOPENSSL_LIBRARIES=/usr/local/opt/openssl/lib -DBUILD_MONGO_DB_PLUGIN=true ..
```

Then on all platforms:
``` bash
make -j$( nproc )
```

Out-of-source builds are also supported. To override clang's default choice in compiler, add these flags to the CMake command:

`-DCMAKE_CXX_COMPILER=/path/to/c++ -DCMAKE_C_COMPILER=/path/to/cc`

For a debug build, add `-DCMAKE_BUILD_TYPE=Debug`. Other common build types include `Release` and `RelWithDebInfo`.

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
 
For ease of contract development, content can be installed in the `/usr/local` folder using the `make install` target.  This step is run from the `build` folder.  Adequate permission is required to install.

```bash
cd build
sudo make install
```

<a name="manualdep"></a>
## Manual installation of the dependencies

If you prefer to manually build dependencies, follow the steps below.

This project is written primarily in C++14 and uses CMake as its build system. An up-to-date Clang and the latest version of CMake is recommended.

Dependencies:

* Clang 4.0.0
* CMake 3.5.1
* Boost 1.66
* OpenSSL
* LLVM 4.0
* [secp256k1-zkp (Cryptonomex branch)](https://github.com/cryptonomex/secp256k1-zkp.git)

<a name="manualdepamazon"></a>
### Clean install Amazon 2017.09 and higher

Install the development toolkit:

```bash
sudo yum update
sudo yum install git gcc72.x86_64 gcc72-c++.x86_64 autoconf automake libtool make bzip2 \
				 bzip2-devel.x86_64 openssl-devel.x86_64 gmp.x86_64 gmp-devel.x86_64 \
				 libstdc++72.x86_64 python36-devel.x86_64 libedit-devel.x86_64 \
				 ncurses-devel.x86_64 swig.x86_64 gettext-devel.x86_64

```

Install CMake 3.10.2:

```bash
cd ~
curl -L -O https://cmake.org/files/v3.10/cmake-3.10.2.tar.gz
tar xf cmake-3.10.2.tar.gz
rm -f cmake-3.10.2.tar.gz
ln -s cmake-3.10.2/ cmake
cd cmake
./bootstrap
make
sudo make install
```

Install Boost 1.66:

```bash
cd ~
curl -L https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.bz2 > boost_1.66.0.tar.bz2
tar xf boost_1.66.0.tar.bz2
echo "export BOOST_ROOT=$HOME/boost_1_66_0" >> ~/.bash_profile
source ~/.bash_profile
cd boost_1_66_0/
./bootstrap.sh "--prefix=$BOOST_ROOT"
./b2 install
```
Install [MongoDB (mongodb.org)](https://www.mongodb.com):

```bash
mkdir ${HOME}/opt
cd ${HOME}/opt
curl -OL https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-amazon-3.6.3.tgz
tar xf mongodb-linux-x86_64-amazon-3.6.3.tgz
rm -f mongodb-linux-x86_64-amazon-3.6.3.tgz
ln -s ${HOME}/opt/mongodb-linux-x86_64-amazon-3.6.3/ ${HOME}/opt/mongodb
mkdir ${HOME}/opt/mongodb/data
mkdir ${HOME}/opt/mongodb/log
touch ${HOME}/opt/mongodb/log/mongod.log
		
tee > /dev/null ${HOME}/opt/mongodb/mongod.conf <<mongodconf
systemLog:
 destination: file
 path: ${HOME}/opt/mongodb/log/mongod.log
 logAppend: true
 logRotate: reopen
net:
 bindIp: 127.0.0.1,::1
 ipv6: true
storage:
 dbPath: ${HOME}/opt/mongodb/data
mongodconf

export PATH=${HOME}/opt/mongodb/bin:$PATH
mongod -f ${HOME}/opt/mongodb/mongod.conf
```
Install [mongo-cxx-driver (release/stable)](https://github.com/mongodb/mongo-cxx-driver):

```bash
cd ~
curl -LO https://github.com/mongodb/mongo-c-driver/releases/download/1.9.3/mongo-c-driver-1.9.3.tar.gz
tar xf mongo-c-driver-1.9.3.tar.gz
cd mongo-c-driver-1.9.3
./configure --enable-static --enable-ssl=openssl --disable-automatic-init-and-cleanup --prefix=/usr/local
make -j$( nproc )
sudo make install
git clone https://github.com/mongodb/mongo-cxx-driver.git --branch releases/stable --depth 1
cd mongo-cxx-driver/build
cmake -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
sudo make -j$( nproc )
```

Install [secp256k1-zkp (Cryptonomex branch)](https://github.com/cryptonomex/secp256k1-zkp.git):

```bash
cd ~
git clone https://github.com/cryptonomex/secp256k1-zkp.git
cd secp256k1-zkp
./autogen.sh
./configure
make -j$( nproc )
sudo make install
```

By default LLVM and clang do not include the WASM build target, so you will have to build it yourself:

```bash
mkdir  ~/wasm-compiler
cd ~/wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git
cd ..
mkdir build
cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=.. -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ../
make -j$( nproc ) 
make install
```

Your environment is set up. Now you can <a href="#runanode">build EOS and run a node</a>.

<a name="manualdepcentos"></a>
### Clean install Centos 7 and higher

Install the development toolkit:
* Installation on Centos requires installing/enabling the Centos Software Collections
  Repository.
  [Centos SCL](https://wiki.centos.org/AdditionalResources/Repositories/SCL):

```bash
sudo yum --enablerepo=extras install centos-release-scl
sudo yum update
sudo yum install -y devtoolset-7
scl enable devtoolset-7 bash
sudo yum install -y python33.x86_64
scl enable python33 bash
sudo yum install git autoconf automake libtool make bzip2 \
				 bzip2-devel.x86_64 openssl-devel.x86_64 gmp-devel.x86_64 \
				 ocaml.x86_64 doxygen libicu-devel.x86_64 python-devel.x86_64 \
				 gettext-devel.x86_64

```

Install CMake 3.10.2:

```bash
cd ~
curl -L -O https://cmake.org/files/v3.10/cmake-3.10.2.tar.gz
tar xf cmake-3.10.2.tar.gz
cd cmake-3.10.2
./bootstrap
make -j$( nproc )
sudo make install
```

Install Boost 1.66:

```bash
cd ~
curl -L https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.bz2 > boost_1.66.0.tar.bz2
tar xf boost_1.66.0.tar.bz2
echo "export BOOST_ROOT=$HOME/boost_1_66_0" >> ~/.bash_profile
source ~/.bash_profile
cd boost_1_66_0/
./bootstrap.sh "--prefix=$BOOST_ROOT"
./b2 install
```

Install [MongoDB (mongodb.org)](https://www.mongodb.com):

```bash
mkdir ${HOME}/opt
cd ${HOME}/opt
curl -OL https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-amazon-3.6.3.tgz
tar xf mongodb-linux-x86_64-amazon-3.6.3.tgz
rm -f mongodb-linux-x86_64-amazon-3.6.3.tgz
ln -s ${HOME}/opt/mongodb-linux-x86_64-amazon-3.6.3/ ${HOME}/opt/mongodb
mkdir ${HOME}/opt/mongodb/data
mkdir ${HOME}/opt/mongodb/log
touch ${HOME}/opt/mongodb/log/mongod.log
		
tee > /dev/null ${HOME}/opt/mongodb/mongod.conf <<mongodconf
systemLog:
 destination: file
 path: ${HOME}/opt/mongodb/log/mongod.log
 logAppend: true
 logRotate: reopen
net:
 bindIp: 127.0.0.1,::1
 ipv6: true
storage:
 dbPath: ${HOME}/opt/mongodb/data
mongodconf

export PATH=${HOME}/opt/mongodb/bin:$PATH
mongod -f ${HOME}/opt/mongodb/mongod.conf
```

Install [mongo-cxx-driver (release/stable)](https://github.com/mongodb/mongo-cxx-driver):

```bash
cd ~
curl -LO https://github.com/mongodb/mongo-c-driver/releases/download/1.9.3/mongo-c-driver-1.9.3.tar.gz
tar xf mongo-c-driver-1.9.3.tar.gz
cd mongo-c-driver-1.9.3
./configure --enable-static --enable-ssl=openssl --disable-automatic-init-and-cleanup --prefix=/usr/local
make -j$( nproc )
sudo make install
git clone https://github.com/mongodb/mongo-cxx-driver.git --branch releases/stable --depth 1
cd mongo-cxx-driver/build
cmake -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
sudo make -j$( nproc )
```

Install [secp256k1-zkp (Cryptonomex branch)](https://github.com/cryptonomex/secp256k1-zkp.git):

```bash
cd ~
git clone https://github.com/cryptonomex/secp256k1-zkp.git
cd secp256k1-zkp
./autogen.sh
./configure
make -j$( nproc )
sudo make install
```

By default LLVM and clang do not include the WASM build target, so you will have to build it yourself:

```bash
mkdir  ~/wasm-compiler
cd ~/wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git
cd ..
mkdir build
cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=.. -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly 
-DLLVM_ENABLE_RTTI=1 -DCMAKE_BUILD_TYPE=Release ../
make -j$( nproc ) 
make install
```

Your environment is set up. Now you can <a href="#runanode">build EOS and run a node</a>.

<a name="manualdepfedora"></a>
### Clean install Fedora 25 and higher

Install the development toolkit:

```bash
sudo yum update
sudo yum install git gcc.x86_64 gcc-c++.x86_64 autoconf automake libtool make cmake.x86_64 \
				 bzip2-devel.x86_64 openssl-devel.x86_64 gmp-devel.x86_64 \
				 libstdc++-devel.x86_64 python3-devel.x86_64 libedit.x86_64 \
				 mongodb.x86_64 mongodb-server.x86_64 ncurses-devel.x86_64 \
				 swig.x86_64 gettext-devel.x86_64

```

Install Boost 1.66:

```bash
cd ~
curl -L https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.bz2 > boost_1.66.0.tar.bz2
tar xf boost_1.66.0.tar.bz2
echo "export BOOST_ROOT=$HOME/boost_1_66_0" >> ~/.bash_profile
source ~/.bash_profile
cd boost_1_66_0/
./bootstrap.sh "--prefix=$BOOST_ROOT"
./b2 install
```
Install [mongo-cxx-driver (release/stable)](https://github.com/mongodb/mongo-cxx-driver):

```bash
cd ~
curl -LO https://github.com/mongodb/mongo-c-driver/releases/download/1.9.3/mongo-c-driver-1.9.3.tar.gz
tar xf mongo-c-driver-1.9.3.tar.gz
cd mongo-c-driver-1.9.3
./configure --enable-static --enable-ssl=openssl --disable-automatic-init-and-cleanup --prefix=/usr/local
make -j$( nproc )
sudo make install
git clone https://github.com/mongodb/mongo-cxx-driver.git --branch releases/stable --depth 1
cd mongo-cxx-driver/build
cmake -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
sudo make -j$( nproc )
```

Install [secp256k1-zkp (Cryptonomex branch)](https://github.com/cryptonomex/secp256k1-zkp.git):

```bash
cd ~
git clone https://github.com/cryptonomex/secp256k1-zkp.git
cd secp256k1-zkp
./autogen.sh
./configure
make -j$( nproc )
sudo make install
```

By default LLVM and clang do not include the WASM build target, so you will have to build it yourself:

```bash
mkdir  ~/wasm-compiler
cd ~/wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git
cd ..
mkdir build
cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=.. -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ../
make -j$( nproc ) install
```

Your environment is set up. Now you can <a href="#runanode">build EOS and run a node</a>.

<a name="manualdepubuntu"></a>
### Clean install Ubuntu 16.04 & Linux Mint 18

Install the development toolkit:

```bash
sudo apt-get update
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
sudo apt-get install clang-4.0 lldb-4.0 libclang-4.0-dev cmake make \
                     libbz2-dev libssl-dev libgmp3-dev \
                     autotools-dev build-essential \
                     libbz2-dev libicu-dev python-dev \
                     autoconf libtool git mongodb
```

Install Boost 1.66:

```bash
cd ~
wget -c 'https://sourceforge.net/projects/boost/files/boost/1.66.0/boost_1_66_0.tar.bz2/download' -O boost_1.66.0.tar.bz2
tar xjf boost_1.66.0.tar.bz2
cd boost_1_66_0/
echo "export BOOST_ROOT=$HOME/boost_1_66_0" >> ~/.bash_profile
source ~/.bash_profile
./bootstrap.sh "--prefix=$BOOST_ROOT"
./b2 install
source ~/.bash_profile
```

Install [mongo-cxx-driver (release/stable)](https://github.com/mongodb/mongo-cxx-driver):

```bash
cd ~
curl -LO https://github.com/mongodb/mongo-c-driver/releases/download/1.9.3/mongo-c-driver-1.9.3.tar.gz
tar xf mongo-c-driver-1.9.3.tar.gz
cd mongo-c-driver-1.9.3
./configure --enable-static --enable-ssl=openssl --disable-automatic-init-and-cleanup --prefix=/usr/local
make -j$( nproc )
sudo make install
git clone https://github.com/mongodb/mongo-cxx-driver.git --branch releases/stable --depth 1
cd mongo-cxx-driver/build
cmake -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
sudo make -j$( nproc )
```

Install [secp256k1-zkp (Cryptonomex branch)](https://github.com/cryptonomex/secp256k1-zkp.git):

```bash
cd ~
git clone https://github.com/cryptonomex/secp256k1-zkp.git
cd secp256k1-zkp
./autogen.sh
./configure
make
sudo make install
```

By default LLVM and clang do not include the WASM build target, so you will have to build it yourself:

```bash
mkdir  ~/wasm-compiler
cd ~/wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git
cd ..
mkdir build
cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=.. -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ../
make -j4 install
```

Your environment is set up. Now you can <a href="#runanode">build EOS and run a node</a>.

<a name="manualdepmacos"></a>
### Clean install MacOS Sierra 10.12.6 & higher

macOS additional Dependencies:

* Brew
* Newest XCode
* MongoDB C++ driver

Upgrade your XCode to the newest version:

```bash
xcode-select --install
```

Install homebrew:

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Install the dependencies:

```bash
brew update
brew install git automake libtool cmake boost openssl@1.0 llvm@4 gmp ninja gettext mongodb
brew link gettext --force
```
Install [mongo-cxx-driver (release/stable)](https://github.com/mongodb/mongo-cxx-driver):

```bash
cd ~
brew install --force pkgconfig
brew unlink pkgconfig && brew link --force pkgconfig
curl -LO https://github.com/mongodb/mongo-c-driver/releases/download/1.9.3/mongo-c-driver-1.9.3.tar.gz
tar xf mongo-c-driver-1.9.3.tar.gz
rm -f mongo-c-driver-1.9.3.tar.gz
cd mongo-c-driver-1.9.3
./configure --enable-static --enable-ssl=darwin --disable-automatic-init-and-cleanup --prefix=/usr/local
make -j$( sysctl -in machdep.cpu.core_count )
sudo make install
cd ..
rm -rf mongo-c-driver-1.9.3

git clone https://github.com/mongodb/mongo-cxx-driver.git --branch releases/stable --depth 1
cd mongo-cxx-driver/build
cmake -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
make -j$( sysctl -in machdep.cpu.core_count )
sudo make install
cd ..
rm -rf mongo-cxx-driver
```

Install [secp256k1-zkp (Cryptonomex branch)](https://github.com/cryptonomex/secp256k1-zkp.git):

```bash
cd ~
git clone https://github.com/cryptonomex/secp256k1-zkp.git
cd secp256k1-zkp
./autogen.sh
./configure
make -j$( sysctl -in machdep.cpu.core_count )
sudo make install
```

Build LLVM and clang for WASM:

```bash
mkdir  ~/wasm-compiler
cd ~/wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git
cd ..
mkdir build
cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=.. -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ../
make -j$( sysctl -in machdep.cpu.core_count )
make install
```

Your environment is set up. Now you can <a href="#runanode">build EOS and run a node</a>.


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
$ docker run --name nodeos -p 8888:8888 -p 9876:9876 -t eosio/eos start_nodeos.sh arg1 arg2
```

By default, all data is persisted in a docker volume. It can be deleted if the data is outdated or corrupted:
``` bash
$ docker inspect --format '{{ range .Mounts }}{{ .Name }} {{ end }}' nodeos
fdc265730a4f697346fa8b078c176e315b959e79365fc9cbd11f090ea0cb5cbc
$ docker volume rm fdc265730a4f697346fa8b078c176e315b959e79365fc9cbd11f090ea0cb5cbc
```

Alternately, you can directly mount host directory into the container
```bash
$ docker run --name nodeos -v /path-to-data-dir:/opt/eos/bin/data-dir -p 8888:8888 -p 9876:9876 -t eosio/eos start_nodeos.sh arg1 arg2
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

<a name="4-creating-and-launching-a-single-node-testnet"></a>
## Creating and Launching a Single Node Testnet

After successfully building the project, the `nodeos` binary should be present in the `build/programs/nodeos` folder.  `nodeos` can be run directly from the `build` folder using `programs/nodeos/nodeos`.

You can start your own single-node blockchain with this single command:

```
$ nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::account_history_api_plugin 
...
eosio generated block 046b9984... #101527 @ 2018-04-01T14:24:58.000 with 0 trxs
eosio generated block 5e527ee2... #101528 @ 2018-04-01T14:24:58.500 with 0 trxs

```

The more advanced user will likely have need to modify the configuration.  `nodeos` uses a custom configuration folder.  The location of this folder is determined by your system.

- Mac OS: `~/Library/Application Support/eosio/nodeos/config`
- Linux: `~/.local/share/eosio/nodeos/config`

The build seeds this folder with a default `genesis.json` file.  For the more advanced user, a configuration folder can be specified using the `--config-dir` command line argument to `nodeos`.  If you use this option, you will need to manually copy a genesis.json file to your config folder.

`nodeos` will need a properly configured `config.ini` file in order to do meaningful work.  On startup, `nodeos` looks in the config folder for `config.ini`.  If one is not found, a default `config.ini` file is created.  If you do not
already have a `config.ini` file ready to use, run `nodeos` and then close it immediately with <kbd>Ctrl-C</kbd>.  A default configuration (`config.ini`) will have been created in the config folder.  Edit the `config.ini` file, adding/updating the following settings to the defaults already in place:

```
# Load the testnet genesis state, which creates some initial block producers with the default key
genesis-json = /path/to/eos/source/genesis.json
# Enable production on a stale chain, since a single-node test chain is pretty much always stale
enable-stale-production = true
# Enable block production with the testnet producers
producer-name = eosio
# Load the block producer plugin, so you can produce blocks
plugin = eosio::producer_plugin
# Wallet plugin
plugin = eosio::wallet_api_plugin
# As well as API and HTTP plugins
plugin = eosio::chain_api_plugin
plugin = eosio::http_plugin
# This will be used by the validation step below, to view account history
plugin = eosio::account_history_api_plugin
```

Now it should be possible to run `nodeos` and see it begin producing blocks.
```bash
programs/nodeos/nodeos
```
When running `nodeos` you should get log messages similar to below. It means the blocks are successfully produced.

```
1575001ms thread-0   chain_controller.cpp:235      _push_block          ] initm #1 @2017-09-04T04:26:15  | 0 trx, 0 pending, exectime_ms=0
1575001ms thread-0   producer_plugin.cpp:207       block_production_loo ] initm generated block #1 @ 2017-09-04T04:26:15 with 0 trxs  0 pending
1578001ms thread-0   chain_controller.cpp:235      _push_block          ] initc #2 @2017-09-04T04:26:18  | 0 trx, 0 pending, exectime_ms=0
1578001ms thread-0   producer_plugin.cpp:207       block_production_loo ] initc generated block #2 @ 2017-09-04T04:26:18 with 0 trxs  0 pending
...
```
At this point, `nodeos` is running with a single producer, `eosio`, that is defined in the `genesis.json` file.

`nodeos` stores runtime data (e.g., shared memory and log content) in a custom data folder.  The location of this folder is determined by your system.

- Mac OS: `~/Library/Application Support/eosio/nodeos/data`
- Linux: `~/.local/share/eosio/nodeos/data`

A data folder can be specified using the `--data-dir` command line argument to `nodeos`.


<a name="validatewithcurrencycontract"></a>
## Validate the Environment - "Currency" Contract Walkthrough

EOS comes with example contracts that can be uploaded and run for testing purposes.  We will validate our single node setup using the sample contract "currency".  It is assumed `nodeos` is running as described above.

### Create a wallet

Every contract requires an associated account, so first you need to create a wallet.  To create a wallet, you need to have the wallet_api_plugin loaded into the `nodeos` process.  This can be accomplished in one of two ways:
* Via a plugin entry in the config.ini file (i.e. `plugin = eosio::wallet_api_plugin`)
* Via a plugin command-line option when invoking nodeos (i.e. `--plugin eosio::wallet_api_plugin`)

Recall that in the previous section above, we added the wallet plugin to the `config.ini` file before starting `nodeos`.  Thus, our currently running `nodeos` already has the necessary plugin.

Use the `wallet create` subcommand of `cleos` to create a wallet.

```bash
cd ~/eos/build/programs/cleos/
./cleos wallet create # Outputs a password that you need to save to be able to lock/unlock the wallet
```
**Important:**
Save the wallet password for future reference.

## Load the Bios Contract

Set `eosio.bios` as the default system contract.  This contract enables you to have direct control over the resource allocation of other accounts and to access other privileged API calls.

```
$ ./cleos set contract eosio ../../contracts/eosio.bios -p eosio
```

### Create an account for the "currency" contract

The account named "currency" will be used for the "currency" contract.  Generate two public/private key pairs that will be later assigned as the `public-OwnerKey` and the `public-ActiveKey`.

```bash
cd ~/eos/build/programs/cleos/
./cleos create key # OwnerKey
./cleos create key # ActiveKey
```

This will output two pairs of public and private keys of the form:

```
Private key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Public key: EOSXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

**Important:**
Save the values for future reference.

Import the two private keys into the wallet.

```bash
./cleos wallet import <private-OwnerKey>
./cleos wallet import <private-ActiveKey>
```

Create the `currency` account using the `cleos create account` command.  The create will be authorized by the `eosio` account.  The two public keys generated above will be associated with the account, one as its `OwnerKey` and the other as its `ActiveKey`.

```bash
./cleos create account eosio currency <public-OwnerKey> <public-ActiveKey>
```

You should get a JSON response back with a transaction ID confirming it was executed successfully, e.g.:
```bash
executed transaction: fe5c9db1b5173dd4bd1ed79c23056104427ab62b0086cf117175abb322532d93  346 bytes  101544 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"currency","owner":{"threshold":1,"keys":[{"key":"EOS6eRfSRYNcrsLmLMomWbBk..." +
 ```

You can verify that the account was successfully created:
```bash
./cleos get account currency
```

If all has gone well, you will receive an output similar to the following:

```json
{
  "account_name": "currency",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS8kjeKVzFfqYyqcG8EnRLvMyLjJ7nmSM8p7QqDazGnjMEtQd1dp",
            "weight": 1
          }
        ],
        "accounts": []
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS6eRfSRYNcrsLmLMomWbBk317gz2TcBqArL7JwaqvaYkWYALe73",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}
```

### Upload the sample "currency" contract to the blockchain

Before uploading a contract, verify that there is no current contract:

```bash
./cleos get code currency
code hash: 0000000000000000000000000000000000000000000000000000000000000000
```

Upload the sample currency **contract** using the currency **account**:
```bash
./cleos set contract currency ../../contracts/currency
```

The response should be a `transaction_id` with some JSON.  This indicates your contract was successfully uploaded.

You can also verify that the code has been set with the following command:
```bash
./cleos get code currency
```

It will return something like:
```bash
code hash: 9b9db1a7940503a88535517049e64467a6e8f4e9e03af15e9968ec89dd794975
```

Before using the currency contract, you must first create, then issue the currency.

```bash
./cleos push action currency create '{"issuer":"currency","maximum_supply":"1000000.0000 CUR","can_freeze":"0","can_recall":"0","can_whitelist":"0"}' --permission currency@active
./cleos push action currency issue '{"to":"currency","quantity":"1000.0000 CUR","memo":""}' --permission currency@active
```

Next verify the currency contract has the proper initial balance:

```bash
./cleos get table currency currency accounts
{
  "rows": [{
      "balance": "1000.0000 CUR",
      "frozen": 0,
      "whitelist": 1
    }
  ],
  "more": false
}
```

### Transfer funds using the "currency" contract

The following command shows a "transfer" action being sent to the currency contract, transferring "20.0000 CUR" from the currency account to the "eosio" account.

```bash
./cleos push action currency transfer '{"from":"currency","to":"eosio","quantity":"20.0000 CUR","memo":"my first transfer"}' --permission currency@active
```

A successfully submitted transaction will generate a transaction ID and JSON output similar to the following.

```bash
executed transaction: de83ee65f983be89bebd2fc5d5ba066acaadcdebdbfc15f8f1221b98f76551ea  271 bytes  109135 cycles
#      currency <= currency::transfer           {"from":"currency","to":"eosio","quantity":"20.0000 CUR","memo":"my first transfer"}
>> transfer
#         eosio <= currency::transfer           {"from":"currency","to":"eosio","quantity":"20.0000 CUR","memo":"my first transfer"}
```

### Check the "currency" contract balances

Check the state of both accounts involved in the previous transaction as follows:

```bash
./cleos get table currency eosio accounts
{
  "rows": [{
      "balance": "20.0000 CUR",
      "frozen": 0,
      "whitelist": 1
    }
  ],
  "more": false
}
./cleos get table currency currency accounts
{
  "rows": [{
      "balance": "980.0000 CUR",
      "frozen": 0,
      "whitelist": 1
    }
  ],
  "more": false
}
```

As expected, the receiving account **eosio** now has a balance of **20**, and the sending account **currency** now has **20** less than its initial issue.


<a name="5-troubleshooting-guide"></a>
## Troubleshooting Guide

1. *You get an error such as `St9exception: content of memory does not match data expected by executable` when trying to start `nodeos`*
> Try restarting `nodeos` with `--resync`
2. How do I find which version of `nodeos` I'm running or connecting to?
> Use `cleos -H ${nodeos_host} -p ${nodeos_port} get info` and you will see the version number in the field called `server_version`
