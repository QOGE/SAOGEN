# Deploying ZNY-NOMP as a Mining Pool for QOGE

This document describes the verified, working configuration for running
[zny-nomp](https://github.com/ROZ-MOFUMOFU-ME/zny-nomp) (v1.4.0) as a
stratum mining pool for a Qogecoin (QOGE, yescryptR16) node.

## 1. Prerequisites

- **Node.js `^20.19.0` or `>=22.12.0`** (per `package.json` `engines`).
  This deployment uses **Node v20.20.2**. Use `nvm` to pin the version:
  ```bash
  nvm install 20.20.2
  nvm alias default 20.20.2
  ```
- **Redis** 7.x (tested with 7.0.15). Any recent Redis 6/7 build is fine —
  zny-nomp uses it for round/share/balance state and nothing exotic
  (plain hashes, sets, and pub/sub).
  Worth installing it now, before npm install, actually, since it's a real prerequisite for the pool to function even though npm install itself won't fail without it.
   ```bash
  sudo apt-get install -y redis-server
  redis-server --version
  ```
   Should print something in the 6.x/7.x range — matches what the tutorial confirms is fine.

   Worth enabling it to start automatically, since the systemd service in Step 7 expects it already running (After=network.target redis-server.service):
   ```bash
   sudo systemctl enable redis-server
   sudo systemctl start redis-server
   sudo systemctl status redis-server --no-pager
   ```
   Should show active (running). Once that's confirmed, continue with the build dependencies and npm install from the previous message.
  
- **Build dependencies** for native modules (`node-multi-hashing`, and
  friends pulled in via git dependencies) — install before `npm install`:
  ```bash
  sudo apt-get update
  sudo apt-get install -y build-essential python3-dev libpython3-dev \
      libgmp3-dev libsodium-dev
  ```

Clone and install:

```bash
git clone https://github.com/ROZ-MOFUMOFU-ME/zny-nomp.git
cd zny-nomp
npm install
```

## 2. Daemon-side configuration (`qogecoind`)

zny-nomp talks to `qogecoind` over JSON-RPC. Before touching any of the
pool config in this document, apply
`contrib/qogecoin-production.conf.example` from the `qogecoin` repo to
your node's `qogecoin.conf`. In particular, `fallbackfee`, `paytxfee`,
`mintxfee`, and `wallet=` persistence are load-bearing for payment
processing to function at all — see that file for the actual values and
rationale, so the two documents don't drift out of sync.

```bash
nano ~/.qogecoin/qogecoin.conf
```

```bash
##
## qogecoin-production.conf.example
##
## Reference production node configuration for QOGE/qogecoin.
## Every setting below reflects a specific operational lesson learned during
## this project's development, testnet, and mainnet activation simulation.
##
## HOW TO USE:
##   cp contrib/qogecoin-production.conf.example ~/.qogecoin/qogecoin.conf
##   Edit the file — change ALL placeholder credentials before starting the node.
##
## !! NEVER use the placeholder rpcuser/rpcpassword values as-is in production !!
##

##############################################################################
## Network
##############################################################################

# Accept RPC connections and inbound peer connections.
server=1
listen=1
txindex=1
##############################################################################
## RPC credentials — CHANGE THESE BEFORE USE
##
## !! PLACEHOLDER VALUES — DO NOT USE IN PRODUCTION !!
## Generate a strong random password (e.g. openssl rand -hex 32).
## Alternatively, use rpcauth= (see share/rpcauth/rpcauth.py) so the
## plaintext password never appears in this file.
##############################################################################

rpcuser=
rpcpassword=

# Bind RPC only to localhost. Never expose the RPC port to the public
# internet — there is no TLS; credentials are sent in the clear.
rpcbind=127.0.0.1
rpcallowip=127.0.0.1
rpcport=8332
##############################################################################
## Fee policy
##
## LESSON LEARNED: Fee estimation requires historical mempool data to build
## its internal model. On a low-activity or newly-synced chain (including
## testnet and early mainnet), the estimator has no data and returns an error.
## Without these fallback values, sendmany, sendtoaddress, and any
## payment-processing pipeline will fail with "Fee estimation failed. Set
## -paytxfee or read the documentation for -fallbackfee and -settxfee" on
## every single call — silently halting payment processing with no obvious
## startup-time error. Root-cause confirmed during ZNY-NOMP payment pipeline
## investigation.
##############################################################################
##############################################################################
## Fee policy
##
## LESSON LEARNED: Fee estimation requires historical mempool data to build
## its internal model. On a low-activity or newly-synced chain (including
## testnet and early mainnet), the estimator has no data and returns an error.
## Without these fallback values, sendmany, sendtoaddress, and any
## payment-processing pipeline will fail with "Fee estimation failed. Set
## -paytxfee or read the documentation for -fallbackfee and -settxfee" on
## every single call — silently halting payment processing with no obvious
## startup-time error. Root-cause confirmed during ZNY-NOMP payment pipeline
## investigation.
##############################################################################

fallbackfee=0.0001
paytxfee=0.0001
mintxfee=0.00001

##############################################################################
## Wallet loading — persistent across restarts
##
## LESSON LEARNED: A wallet loaded once via the loadwallet RPC is NOT
## reloaded automatically on daemon restart. After any restart the node
## starts with no wallet, all wallet RPCs return error -18 ("No wallet is
## loaded"), and payment processing halts silently — there is no startup-time
## warning. If this node runs a pool or payment-processing wallet, name it
## here so it is always loaded at startup. Uncomment and set the wallet name.
##
## wallet=<your-wallet-name-here>
##############################################################################
wallet=poolwallet
##############################################################################
## Change address type — bech32 (SegWit v0 / P2WPKH), NOT Taproot
##
## LESSON LEARNED: Bitcoin-Core-derived wallets default new CHANGE outputs
## (not fresh receiving addresses — change specifically) to Taproot (bech32m /
## P2TR) once Taproot is network-active, since there is no interoperability
## concern for change. Taproot outputs commit to a tweaked secp256k1 point
## Q = P + t*G visible on-chain from the moment the output is funded — the
## raw (tweaked) public key is exposed at rest, before any spend attempt.
##
## For a project whose entire purpose is minimising at-rest public-key
## exposure (SIP-QOGE-PQC-01/02, HNDL defence), Taproot change outputs are
## exactly backwards. Force bech32 (P2WPKH) instead: the public key is
## hidden behind HASH160 at rest and revealed only at spend time, during the
## SLH-DSA witness-validation window. This does not give full P2QPK-level
## protection (only Symbiont Wallet generates true P2QPK outputs), but it
## eliminates the gratuitous at-rest exposure that Taproot change would add.
##
## Applies under [main]; replicate under [test] / [regtest] if needed there.
##############################################################################

[main]
changetype=bech32
maxconnections=125
addnode=185.255.131.11:42069

[test]
changetype=bech32

[regtest]
changetype=bech32
maxconnections=125

##############################################################################
## Connections
##############################################################################

maxconnections=125
addnode=38.242.226.36
addnode=185.255.131.11


```

## 3. Pool wallet creation

Create the pool payout wallet using a **legacy (Base58) address**, not
Bech32 — zny-nomp's address handling does not correctly support Bech32
addresses.

```bash
qogecoin-cli createwallet "poolwallet"
qogecoin-cli -rpcwallet=poolwallet getnewaddress "" legacy
```

Keep the daemon running with this wallet loaded at all times — the
payment processor needs `listunspent`/`sendmany` against it continuously.

## 4. `coins/qoge.json`

```json
{
    "name": "Qogecoin",
    "symbol": "QOGE",
    "algorithm": "yescryptR16",
    "peerMagic": "fabfb5da",
    "peerMagicTestnet": "fcc1b7dc",
    "getInfo": false,
    "noNetworkInfo": false,
    "noGetnetworkhashps": true,
    "txfee": 0.0001,
    "blockTime": 60,
    "mainnet": {
        "bip32": {
            "public": "0488b21e"
        },
        "pubKeyHash": "78",
        "scriptHash": "66"
    },
    "explorer": {
        "txURL": "https://explorer.qoge.org/tx/",
        "blockURL": "https://explorer.qoge.org/block/"
    }
}

```

Note: `name` is `"Qogecoin"`. zny-nomp lowercases this internally
(`coin.name.toLowerCase()`) to derive the Redis key prefix — all Redis
keys for this pool live under `qogecoin:*`, not `qoge:*`. Keep this in
mind if you ever inspect Redis directly.

## 5. `pool_configs/qoge.json`

```json
{
    "enabled": true,
    "coin": "qoge.json",
    "BTCover17": true,
    "_comment_BTCover17": "Set true if the daemon is Bitcoin-Core-derived 0.17+ (uses getaddressinfo). Set false for older daemons (validateaddress). Must match the actual daemon version exactly,>

    "address": "qJkYaooBiLgFY3APdTuHenALqGj5iRL9Jj",
    "_comment_address": "Legacy/Base58 address from step 3 above. Must be owned by the daemon's loaded wallet.",

    "rewardRecipients": {
        "qJkYaooBiLgFY3APdTuHenALqGj5iRL9Jj": 1.0
    },
    "_comment_rewardRecipients": "This key MUST exactly match the address above. A mismatched or empty-string key here silently breaks payment processing even though the daemon connection and add>

    "paymentProcessing": {
        "minConf": 10,
        "enabled": true,
        "paymentMode": "prop",
        "_comment_paymentMode": "prop, pplnt",
        "paymentInterval": 600,
        "minimumPayment": 0.001,
        "maxBlocksPerPayment": 30,
        "_comment_maxBlocksPerPayment": "Throttles how many pending blocks are processed per interval. Can be temporarily raised (e.g. to 10+) to drain a large backlog faster, then lowered back d>
        "daemon": {
            "host": "127.0.0.1",
            "port": 8332,
            "user": "xxxxxx",
            "password": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        }
    },

    "tlsOptions": {
        "enabled": false,
        "serverKey": "",
        "serverCert": "",
        "ca": ""
    },

    "ports": {
        "3032": {
            "diff": 0.001,
            "tls": false,
            "varDiff": {
                "minDiff": 0.0001,
                "maxDiff": 16,
                "targetTime": 15,
                "retargetTime": 60,
                "variancePercent": 30
            }
        }
    },

    "poolId": "main",
    "_comment_poolId": "use it for region identification: eu, us, asia or keep default if you have one stratum instance for one coin",

    "daemons": [
        {
            "host": "127.0.0.1",
            "port": 8332,
            "user": "qogecoin",
            "password": "qP8fvsiJaaabGtVgw1F6xCUw81ya4G1MAd"
        }
    ],

    "p2p": {
        "enabled": false,
        "host": "127.0.0.1",
        "port": 8333,
        "disableTransactions": true
    },

    "mposMode": {
        "enabled": false,
        "host": "127.0.0.1",
        "port": 3306,
        "user": "",
        "password": "",
        "database": "",
        "checkPassword": true,
        "autoCreateWorker": false
    }
}

```

> **Warning**: replace every `CHANGE_ME_*` / `YOUR_POOL_WALLET_ADDRESS_HERE`
> placeholder with your real values before deploying. RPC credentials here
> must match `rpcuser`/`rpcpassword` in your `qogecoin.conf`.

`paymentInterval: 60` is the steady-state value — this can be lowered
temporarily during testing/backlog-draining and should be raised back to
a production-appropriate value (60s or higher) afterward.

## 6. `config.json` (portal-level)

```json
{
    "logLevel": "debug",
    "logColors": true,
    "cliPort": 17117,
    "clustering": {
        "enabled": false,
        "forks": "1"
    },
    "defaultPoolConfigs": {
        "blockRefreshInterval": 500,
        "jobRebroadcastTimeout": 55,
        "connectionTimeout": 600,
        "emitInvalidBlockHashes": false,
        "validateWorkerUsername": true,
        "tcpProxyProtocol": false,
        "banning": {
            "enabled": true,
            "time": 600,
            "invalidPercent": 50,
            "checkThreshold": 500,
            "purgeInterval": 300
        },
        "redis": {
            "host": "127.0.0.1",
            "port": 6379,
            "password": ""
        }
    },
    "website": {
        "enabled": true,
        "host": "127.0.0.1",
        "port": 8080,
        "stratumHost": "postquantum.qoge.org",
        "stats": {
            "updateInterval": 10,
            "historicalRetention": 14400,
            "hashrateWindow": 300
        }
    },
    "redis": {
        "host": "127.0.0.1",
        "port": 6379
    }
}

```

Raise `clustering.forks` above `1` once you're running more than one
stratum port/coin at meaningful load; `1` is sufficient for a single-coin
deployment.

## 7. systemd service

`/etc/systemd/system/zny-nomp.service`:

```ini
[Unit]
Description=ZNY-NOMP QOGE Mining Pool
After=network-online.target redis-server.service qogecoind.service
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/root/zny-nomp
ExecStartPre=/bin/sleep 20
ExecStart=/root/.nvm/versions/node/v20.20.2/bin/node init.js
Restart=always
RestartSec=10
User=root
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload   # always required after creating/editing any unit file
sudo systemctl enable zny-nomp # survives reboots
sudo systemctl start zny-nomp
sudo systemctl status zny-nomp --no-pager
```

Note the `ExecStart` uses the full path to the nvm-managed Node binary —
systemd does not source your shell's nvm setup, so `node` alone on
`PATH` will not resolve correctly.

## 8. Verification checklist

After starting the pool, confirm each of the following:

1. **Daemon connection succeeds at startup** — log shows the pool
   spawning without a "Could not connect" or address-ownership error:
   ```bash
   journalctl -u zny-nomp -n 50 --no-pager | grep -iE "daemon|error"
   ```

2. **A test share is accepted** — point a miner (or `cpuminer`) at
   `stratum+tcp://<host>:3421` and confirm an accepted-share log line
   appears.

3. **`blocksPending` → `blocksConfirmed` behaves correctly** once a
   block clears `minConf` confirmations:
   ```bash
   redis-cli smembers qogecoin:blocksPending
   redis-cli smembers qogecoin:blocksConfirmed
   ```
   A newly mined block should appear in `blocksPending`, then move to
   `blocksConfirmed` on a payment-processing pass after it has ≥`minConf`
   confirmations.

4. **A real payment log line appears with no error**, on the next
   `paymentInterval` tick after confirmation:
   ```bash
   journalctl -u zny-nomp -f | grep -i "Payment"
   ```
   Look for a successful `sendmany` result; there should be no
   "Error with payment processing" or "Daemon does not own pool address"
   lines.

5. **Cross-check on-chain**: confirm the payout transaction actually
   landed in the pool wallet's outgoing transactions:
   ```bash
   qogecoin-cli -rpcwallet=poolwallet listtransactions "*" 10
   ```
