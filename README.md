# zipd-rpc

[![Build Status](https://github.com/zipevo/zipd-rpc/actions/workflows/test.yml/badge.svg)](https://github.com/zipevo/zipd-rpc/actions/workflows/test.yml)
[![NPM Package](https://img.shields.io/npm/v/@zipevo/zipd-rpc.svg)](https://www.npmjs.org/package/@zipevo/zipd-rpc)

Zip Client Library to connect to Zip Core (zipd) via RPC

## Install

zipd-rpc runs on [node](http://nodejs.org/), and can be installed via [npm](https://npmjs.org/):

```bash
npm install @zipevo/zipd-rpc
```

## Usage

### RpcClient

Config parameters : 

	- protocol : (string - optional) - (default: 'https') - Set the protocol to be used. Either `http` or `https`.
	- user : (string - optional) - (default: 'user') - Set the user credential.
	- pass : (string - optional) - (default: 'pass') - Set the password credential.
	- host : (string - optional) - (default: '127.0.0.1') - The host you want to connect with.
	- port : (integer - optional) - (default: 9998) - Set the port on which perform the RPC command.

Promise vs callback based

  - `require('@zipevo/zipd-rpc/promise')` to have promises returned
  - `require('@zipevo/zipd-rpc')` to have callback functions returned
	
### Examples

Config:

```javascript
var config = {
    protocol: 'http',
    user: 'zip',
    pass: 'local321',
    host: '127.0.0.1',
    port: 19998
};
```

Promise based:

```javascript
var RpcClient = require('@zipevo/zipd-rpc/promise');
var rpc = new RpcClient(config);

rpc.getRawMemPool()
    .then(ret => {
        return Promise.all(ret.result.map(r => rpc.getRawTransaction(r)))
    })
    .then(rawTxs => {
        rawTxs.forEach(rawTx => {
            console.log(`RawTX: ${rawTx.result}`);
        })
    })
    .catch(err => {
        console.log(err)
    })
```

Callback based (legacy):

```javascript
var run = function() {
  var bitcore = require('@zipevo/zipcore-lib');
  var RpcClient = require('@zipevo/zipd-rpc');
  var rpc = new RpcClient(config);

  var txids = [];

  function showNewTransactions() {
    rpc.getRawMemPool(function (err, ret) {
      if (err) {
        console.error(err);
        return setTimeout(showNewTransactions, 10000);
      }

      function batchCall() {
        ret.result.forEach(function (txid) {
          if (txids.indexOf(txid) === -1) {
            rpc.getRawTransaction(txid);
          }
        });
      }

      rpc.batch(batchCall, function(err, rawtxs) {
        if (err) {
          console.error(err);
          return setTimeout(showNewTransactions, 10000);
        }

        rawtxs.map(function (rawtx) {
          var tx = new bitcore.Transaction(rawtx.result);
          console.log('\n\n\n' + tx.id + ':', tx.toObject());
        });

        txids = ret.result;
        setTimeout(showNewTransactions, 2500);
      });
    });
  }

  showNewTransactions();
};
```

### Help

You can dynamically access to the help of each method by doing

```
const RpcClient = require('@zipevo/zipd-rpc');
var client = new RPCclient({
    protocol:'http',
    user: 'zip',
    pass: 'local321', 
    host: '127.0.0.1', 
    port: 19998,
    timeout: 1000
});

var cb = function (err, data) {
    console.log(data)
};

// Get full help
client.help(cb);

// Get help of specific method
client.help('getinfo',cb);
```

## Contributing

Feel free to dive in! [Open an issue](https://github.com/zipevo/zip-std-template/issues/new) or submit PRs.

## License

[MIT](LICENSE) &copy; Zip Core Group, Inc.
