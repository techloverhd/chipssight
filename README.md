# *ChipsSight*

*ChipsSight* is an open-source CHIPS blockchain explorer with complete REST
and websocket APIs adapted from [insight](https://github.com/bitpay/insight) by [SHossain](https://github.com/himu007). ChipsSight runs in NodeJS, uses AngularJS for the
front-end and LevelDB for storage. 

## Prerequisites

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

* **Node.js v0.10.x** - Download and Install [Node.js](https://www.nodejs.org/download/).

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

    http://localhost:3001


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
