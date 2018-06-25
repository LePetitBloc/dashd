# [Dash core Wallet Daemon (`dashd`)](https://github.com/LePetitBloc/dashd)

[![Docker Stars][docker-svg]][docker-url]

Dash core Wallet Daemon for headless wallets and **masternodes**. 

> For a list of all available wallet images see https://lepetitbloc.github.io/wallets/

## Issues
- Parameters value issues should be reported at:
https://github.com/LePetitBloc/wallets

- All other issues should be reported at:
https://github.com/LePetitBloc/wallets-builder

## Usage
The container can either be used as a classic headless wallet or a **masternode**, only the *command* arguments will differ.

### Headless wallet
1. Run a **wallet** container and specify at least the `rpcuser` and `rpcpassword` to interact with the **Dash** daemon:
```
docker run --name dash-wallet --restart always -d lepetitbloc/dashd:v0.12.2.3 -rpcuser=dash-wallet -rpcpassword=tfsz322s
```
> We recommend to mount a volume for easier access to the *data* and the *configuration* files.
> You should also create a configuration for your RPC credentials (see `./wallet/.dashcore/dash.conf`) to avoid retyping them when using the internal `dash-cli`.
> ```
> docker run --name dash-wallet --restart always -d -v ${PWD}/wallet/:/home/dash/ lepetitbloc/dashd:v0.12.2.3
> ```

> :warning: Ensure that a `data` directory **exists** and is **writable** in the mounted host directory.

2. Once your wallet is running, you can print your main address:
```
docker exec dash-wallet dash-cli getaccountaddress ""
```
> GK92mbjS9bCAUuU7DEyyuK9US1qLqkoyce

2. **Encrypt your wallet:**
> You can use `pwgen` first to generate your *passphrase*:
> ```
> pwgen 32 1
> ```
> quohd4kaw9guvi8ie7phaighawaiLoo6
```
docker exec dash-wallet dash-cli encryptwallet quohd4kaw9guvi8ie7phaighawaiLoo6
```
> Wallet encrypted; Dash Core server stopping, restart to run with encrypted wallet. The keypool has been flushed, you need to make a new backup.

:bangbang: Don't forget to **backup** your *passphrase*.

### Masternode

#### Prerequisites
1. You must have received *1000 DASH* on your **wallet** in a **single transaction**, and **must** have waited for, at least, **1 confirmation**.
> **Note1:** If the *1000 DASH* came from multiple transactions, you can send them back to yourself.

> **Note2:** Beware of the transaction cost, you should own *1001 DASH* as a safety measure.

2. Only then you may find the corresponding transaction `hash` and `index` :
```
docker exec dash-wallet dash-cli masternode outputs
```
>```
>{
>  "8e835a7d867d335434925c32f38902268e131e99a5821557d3e77f8ca3829fd8" : "0"
>}
>```

3. Then generate a **masternode** private key:
```
masternode genkey
```
>```
>7ev3RXQXYfztreEz8wmPKgJUpNiqkAkkdxt24C3ZKtg5qEVfou9
>```

4. And finally creates the `./wallet/.dashcore//masternode.conf` file, and fill in following this template:
> `mn01 masternode:21529 YouMasterNodePrivateKey TransactionHash 0 YourWalletAddress:100`
```
touch ./wallet/.dashcore//masternode.conf
```

#### Setup
1. As a classic wallet, create a `./masternode/.dashcore/dash.conf` configuration file
```
rpcuser=dash-mn01
rpcpassword=tfsz322s
masternode=1
masternodeprivkey=7ev3RXQXYfztreEz8wmPKgJUpNiqkAkkdxt24C3ZKtg5qEVfou9
externalip=YOUR.EXTERNAL.IP:9999

```

2. Run a container as a **masternode**:
```
docker run --name dash-masternode --restart always -d -p 9999:9999 -p 9998:9998 -v ${PWD}/masternode/:/home/dash/ lepetitbloc/dashd:v0.12.2.3 -masternode=1
```

3. Check the the number of `blocks` until the chain is sync:
```
docker exec dash-masternode dash-cli getinfo
```

4. Once the chain synced you can start the **masternode** from your **wallet**:
```
docker exec dash-wallet dash-cli dash-masternode start-all
```
> You might need to unlock your **wallet** first:
> ```
>  docker exec dash-wallet dash-cli walletpassphrase quohd4kaw9guvi8ie7phaighawaiLoo6 60
> ```
> :warning: Mind the **space** before the command above, that's not a typo, it’s meant to avoid storing your passphrase in *history*.

5. Then check the **masternode** status with your initial **transaction hash**:
```
docker exec dash-wallet dash-cli masternodelist | grep 6d94f70499c3f7ba2c59acaa5c04e54ef123d0e460bb07c55ace6464deaf3c85
```
> `"6d94f70499c3f7ba2c59acaa5c04e54ef123d0e460bb07c55ace6464deaf3c85-1": "ENABLED",`

## docker-compose
You could setup **both** at the same time using `docker-compose`.
Check the provided `docker-compose.yml` as an example and tweak it to your needs!
```
docker-compose up --build
```


## Resources
- Website https://www.dash.org/
- Github https://github.com/dashpay/dash



## Parent project
- https://github.com/LePetitBloc/wallets
- https://github.com/LePetitBloc/wallets-builder

## Parent image
- Berkeley DB v4.8.30.NC
`FROM lepetitbloc/bdb:4.8.30.NC`
> https://github.com/LePetitBloc/bdb/tree/4.8.30.NC
> https://hub.docker.com/r/lepetitbloc/bdb/

## Licence
MIT

[docker-url]: https://hub.docker.com/r/lepetitbloc/dashd/
[docker-svg]: https://img.shields.io/docker/stars/lepetitbloc/dashd.svg
