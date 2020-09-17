# *ChipsSight*

*ChipsSight* is an open-source CHIPS blockchain explorer with complete REST
and websocket APIs ported from [insight](https://github.com/bitpay/insight) by [SHossain](https://github.com/himu007). ChipsSight runs in NodeJS, uses AngularJS for the
front-end and LevelDB for storage. 

## Prerequisites

* **git** - Required for cloning the repos.

* **CHIPS daemon** - Compile from [source](https://github.com/jl777/chips3) and sync fully.

* **chips.conf** - Paste the following inside `chips.conf` file:
```
server=1
daemon=1
txindex=1
listen=1
rpcuser=username
rpcpassword=password
rpcallowip=127.0.0.1
p2pport=57777
rpcport=57776
```

* **Node.js v0.10.48** - Download and Install [Node.js](https://nodejs.org/download/release/v0.10.48/).

* **NPM** - Node.js package manager, should be automatically installed when you get node.js.

## Quick Install
  Check the Prerequisites section above before installing.

  To install ChipsSight, clone the main repository:

    $ git clone https://github.com/techloverhd/chipssight.git && cd chipssight

  Install dependencies:

    $ npm install
    
  Run the main application:

    $ INSIGHT_NETWORK=livenet BITCOIND_USER=username BITCOIND_PASS=password node node_modules/.bin/insight-bitcore-api
    
  Then open a browser and go to:

    http://localhost:3000

  Please note that the app will need to sync its internal database
  with the blockchain state, which may take some time. You can check
  sync progress from within the web interface.
  
  
## Nginx Setup

To use Nginx as a reverse proxy for Insight, use the following base [configuration](https://gist.github.com/matiu/bdd5e55ff0ad90b54261)


## Development

To run ChipsSight locally for development mode:

Install bower dependencies:

```
$ bower install
```

To compile and minify the web application's assets:

```
$ grunt compile
```

There is a convinient Gruntfile.js for automation during editing the code

```
$ grunt
```

## Configuration

All configuration is specified in the [config](https://github.com/techloverhd/chipssight-api/tree/master/config/) folder, particularly the [config.js](https://github.com/techloverhd/chipssight-api/tree/master/config/config.js) file. There you can specify your application name and database name. Certain configuration values are pulled from environment variables if they are defined:

```
BITCOIND_HOST         # RPC bitcoind host
BITCOIND_PORT         # RPC bitcoind Port
BITCOIND_P2P_HOST     # P2P bitcoind Host (will default to BITCOIND_HOST, if specified)
BITCOIND_P2P_PORT     # P2P bitcoind Port
BITCOIND_USER         # RPC username
BITCOIND_PASS         # RPC password
BITCOIND_DATADIR      # bitcoind datadir. 'testnet3' will be appended automatically if testnet is used. NEED to finish with '/'. e.g: `/vol/data/`
INSIGHT_NETWORK [= 'livenet' | 'testnet']
INSIGHT_PORT          # chipssight api port
INSIGHT_DB            # Path where to store chipssight's internal DB. (defaults to $HOME/.insight)
INSIGHT_SAFE_CONFIRMATIONS=6  # Nr. of confirmation needed to start caching transaction information
INSIGHT_IGNORE_CACHE  # True to ignore cache of spents in transaction, with more than INSIGHT_SAFE_CONFIRMATIONS confirmations. This is useful for tracking double spents for old transactions.
ENABLE_CURRENCYRATES # if "true" will enable a plugin to obtain historic conversion rates for various currencies
ENABLE_RATELIMITER # if "true" will enable the ratelimiter plugin
LOGGER_LEVEL # defaults to 'info', can be 'debug','verbose','error', etc.
ENABLE_HTTPS # if "true" it will server using SSL/HTTPS
ENABLE_EMAILSTORE # if "true" will enable a plugin to store data with a validated email address
INSIGHT_EMAIL_CONFIRM_HOST # Only meanfull if ENABLE_EMAILSTORE is enable. Hostname for the confirm URLs. E.g: 'https://insight.bitpay.com'

```

Make sure that chipsd is configured to [accept incoming connections using 'rpcallowip'](https://en.bitcoin.it/wiki/Running_Bitcoin).

In case the network is changed (testnet to livenet or vice versa) levelDB database needs to be deleted. This can be performed running:
```util/sync.js -D``` and waiting for *chipssight* to synchronize again.  Once the database is deleted, the sync.js process can be safely interrupted (CTRL+C) and continued from the synchronization process embedded in main app.

## Synchronization

The initial synchronization process scans the blockchain from the paired chipsd server to update addresses and balances. *chipssight-api* needs exactly one trusted chipsd node to run. This node must have finished downloading the blockchain before running *chipssight-api*.

While *chipssight* is synchronizing the website can be accessed (the sync process is embedded in the webserver), but there may be missing data or incorrect balances for addresses. The 'sync' status is shown at the `/api/sync` endpoint.

The blockchain can be read from chipsd's raw `.dat` files or RPC interface.
Reading the information from the `.dat` files is much faster so it's the
recommended (and default) alternative. `.dat` files are scanned in the default
location for each platform (for example, `~/.chips` on Linux). In case a
non-standard location is used, it needs to be defined (see the Configuration section).

While synchronizing the blockchain, *chipssight-api* listens for new blocks and
transactions relayed by the chipsd node. Those are also stored on *chipssight-api*'s database.
In case *chipssight-api* is shutdown for a period of time, restarting it will trigger
a partial (historic) synchronization of the blockchain. Depending on the size of
that synchronization task, a reverse RPC or forward `.dat` syncing strategy will be used.

If chipsd is shutdown, *chipssight-api* needs to be stopped and restarted
once chipsd is restarted.

### Syncing old blockchain data manually

  Old blockchain data can be manually synced issuing:

    $ util/sync.js

  Check util/sync.js --help for options, particulary -D to erase the current DB.

  *NOTE*: there is no need to run this manually since the historic synchronization
  is built in into the web application. Running *chipssight-api* normally will trigger
  the historic sync automatically.


### DB storage requirement

To store the blockchain and address related information, *chipssight-api* uses LevelDB.
Two DBs are created: txs and blocks. By default these are stored on

  ``~/.insight/``

Please note that some older versions of ChipsSight-API store that on `<insight's root>/db`.

This can be changed at config/config.js.

## API

By default, chipssight provides a REST API at `/api` (`http://explorer.chips.cash/api/`), but this prefix is configurable from the var `apiPrefix` in the `config.js` file.

The end-points are:


### Block
```
  http://explorer.chips.cash/api/block/[:hash]
  http://explorer.chips.cash/api/block/0000006e75f6aa0efdbf7db03132aa4e4d0c84951537a6f5a7c39a0a9d30e1e7
```
### Block index
Get block hash by height
```
  http://explorer.chips.cash/api/block-index/[:height]
  http://explorer.chips.cash/api/block-index/0
```
This would return:
```
{"blockHash":"0000006e75f6aa0efdbf7db03132aa4e4d0c84951537a6f5a7c39a0a9d30e1e7"}
```
which is the hash of the Genesis block (0 height)

_for the rest of examples use the URL before the `/api/` endpoints like avobe._

### Transaction
```
  /api/tx/[:txid]
  /api/tx/525de308971eabd941b139f46c7198b5af9479325c2395db7f2fb5ae8562556c
  /api/raw/[:rawid]
  /api/raw/525de308971eabd941b139f46c7198b5af9479325c2395db7f2fb5ae8562556c
```
### Address
```
  /api/addr/[:addr][?noTxList=1&noCache=1]
  /api/addr/mmvP3mTe53qxHdPqXEvdu8WdC7GfQ2vmx5?noTxList=1
```
### Address Properties
```
  /api/addr/[:addr]/balance
  /api/addr/[:addr]/totalReceived
  /api/addr/[:addr]/totalSent
  /api/addr/[:addr]/unconfirmedBalance
```
The response contains the value in Satoshis.
### Unspent Outputs
```
  /api/addr/[:addr]/utxo[?noCache=1]
```
Sample return:
``` json
[
    {
      "address": "n2PuaAguxZqLddRbTnAoAuwKYgN2w2hZk7",
      "txid": "dbfdc2a0d22a8282c4e7be0452d595695f3a39173bed4f48e590877382b112fc",
      "vout": 0,
      "ts": 1401276201,
      "scriptPubKey": "76a914e50575162795cd77366fb80d728e3216bd52deac88ac",
      "amount": 0.001,
      "confirmations": 3
    },
    {
      "address": "n2PuaAguxZqLddRbTnAoAuwKYgN2w2hZk7",
      "txid": "e2b82af55d64f12fd0dd075d0922ee7d6a300f58fe60a23cbb5831b31d1d58b4",
      "vout": 0,
      "ts": 1401226410,
      "scriptPubKey": "76a914e50575162795cd77366fb80d728e3216bd52deac88ac",
      "amount": 0.001,
      "confirmation": 6,
      "confirmationsFromCache": true
    }
]
```
Please note that in case confirmations are cached (which happens by default when the number of confirmations is bigger that INSIGHT_SAFE_CONFIRMATIONS) the response will include the pair confirmationsFromCache:true, and confirmations will equal INSIGHT_SAFE_CONFIRMATIONS. See noCache and INSIGHT_IGNORE_CACHE options for details.



### Unspent Outputs for multiple addresses
GET method:
```
  /api/addrs/[:addrs]/utxo
  /api/addrs/2NF2baYuJAkCKo5onjUKEPdARQkZ6SYyKd5,2NAre8sX2povnjy4aeiHKeEh97Qhn97tB1f/utxo
```

POST method:
```
  /api/addrs/utxo
```

POST params:
```
addrs: 2NF2baYuJAkCKo5onjUKEPdARQkZ6SYyKd5,2NAre8sX2povnjy4aeiHKeEh97Qhn97tB1f
```

### Transactions by Block
```
  /api/txs/?block=HASH
  /api/txs/?block=00000000fa6cf7367e50ad14eb0ca4737131f256fc4c5841fd3c3f140140e6b6
```
### Transactions by Address
```
  /api/txs/?address=ADDR
  /api/txs/?address=mmhmMNfBiZZ37g1tgg2t8DDbNoEdqKVxAL
```

### Transactions for multiple addresses
GET method:
```
  /api/addrs/[:addrs]/txs[?from=&to=]
  /api/addrs/2NF2baYuJAkCKo5onjUKEPdARQkZ6SYyKd5,2NAre8sX2povnjy4aeiHKeEh97Qhn97tB1f/txs?from=0&to=20
```

POST method:
```
  /api/addrs/txs
```

POST params:
```
addrs: 2NF2baYuJAkCKo5onjUKEPdARQkZ6SYyKd5,2NAre8sX2povnjy4aeiHKeEh97Qhn97tB1f
from (optional): 0
to (optional): 20
```

Sample output:
```
{ totalItems: 100,
  from: 0,
  to: 20,
  items:
    [ { txid: '3e81723d069b12983b2ef694c9782d32fca26cc978de744acbc32c3d3496e915',
       version: 1,
       locktime: 0,
       vin: [Object],
       vout: [Object],
       blockhash: '00000000011a135e5277f5493c52c66829792392632b8b65429cf07ad3c47a6c',
       confirmations: 109367,
       time: 1393659685,
       blocktime: 1393659685,
       valueOut: 0.3453,
       size: 225,
       firstSeenTs: undefined,
       valueIn: 0.3454,
       fees: 0.0001 },
      { ... },
      { ... },
      ...
      { ... }
    ]
 }
```

Note: if pagination params are not specified, the result is an array of transactions.


### Transaction broadcasting
POST method:
```
  /api/tx/send
```
POST params:
```
  rawtx: "signed transaction as hex string"

  eg

  rawtx: 01000000017b1eabe0209b1fe794124575ef807057c77ada2138ae4fa8d6c4de0398a14f3f00000000494830450221008949f0cb400094ad2b5eb399d59d01c14d73d8fe6e96df1a7150deb388ab8935022079656090d7f6bac4c9a94e0aad311a4268e082a725f8aeae0573fb12ff866a5f01ffffffff01f0ca052a010000001976a914cbc20a7664f2f69e5355aa427045bc15e7c6c77288ac00000000

```
POST response:
```
  {
      txid: [:txid]
  }

  eg

  {
      txid: "c7736a0a0046d5a8cc61c8c3c2821d4d7517f5de2bc66a966011aaa79965ffba"
  }
```

### Historic blockchain data sync status
```
  /api/sync
```

### Live network p2p data sync status
```
  /api/peer
```

### Status of the CHIPS network
```
  /api/status?q=xxx
```

Where "xxx" can be:

 * getInfo
 * getDifficulty
 * getTxOutSetInfo
 * getBestBlockHash
 * getLastBlockHash

## Web Socket API
The web socket API is served using [socket.io](http://socket.io).

The following are the events published by chipssight:

'tx': new transaction received from network. This event is published in the 'inv' room. Data will be a app/models/Transaction object.
Sample output:
```
{
  "txid":"00c1b1acb310b87085c7deaaeba478cef5dc9519fab87a4d943ecbb39bd5b053",
  "processed":false
  ...
}
```


'block': new block received from network. This event is published in the 'inv' room. Data will be a app/models/Block object.
Sample output:
```
{
  "hash":"000000004a3d187c430cd6a5e988aca3b19e1f1d1727a50dead6c8ac26899b96",
  "time":1389789343,
  ...
}
```

'<chipsAddress>': new transaction concerning <chipsAddress> received from network. This event is published in the '<chipsAddress>' room.

'status': every 1% increment on the sync task, this event will be triggered. This event is published in the 'sync' room.

Sample output:
```
{
  blocksToSync: 164141,
  syncedBlocks: 475,
  upToExisting: true,
  scanningBackward: true,
  isEndGenesis: true,
  end: "000000000933ea01ad0ee984209779baaec3ced90fa3f408719526f8d77f4943",
  isStartGenesis: false,
  start: "000000009f929800556a8f3cfdbe57c187f2f679e351b12f7011bfc276c41b6d"
}
```

### Example Usage

The following html page connects to the socket.io chipssight API and listens for new transactions.

html
```
<html>
<body>
  <script src="http://<chipssight-server>:<port>/socket.io/socket.io.js"></script>
  <script>
    eventToListenTo = 'tx'
    room = 'inv'

    var socket = io("http://<chipssight-server>:<port>/");
    socket.on('connect', function() {
      // Join the room.
      socket.emit('subscribe', room);
    })
    socket.on(eventToListenTo, function(data) {
      console.log("New transaction received: " + data.txid)
    })
  </script>
</body>
</html>
```

## Contribute

Contributions and suggestions are welcomed at [ChipsSight github repository](https://github.com/techloverhd/chipssight).


## License
(The MIT License)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
