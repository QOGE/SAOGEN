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
        "pubKeyHash": "41",
        "scriptHash": "69"
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
    "_comment_BTCover17": "Set true if the daemon is Bitcoin-Core-derived 0.17+ (uses getaddressinfo). Set false for older daemons (validateaddress). Must match the actual daemon version exactly, or payment processing silently refuses to start.",

    "address": "YOUR_POOL_WALLET_ADDRESS_HERE",
    "_comment_address": "Legacy/Base58 address from step 3 above. Must be owned by the daemon's loaded wallet.",

    "rewardRecipients": {
        "YOUR_POOL_WALLET_ADDRESS_HERE": 1.0
    },
    "_comment_rewardRecipients": "This key MUST exactly match the address above. A mismatched or empty-string key here silently breaks payment processing even though the daemon connection and address checks all pass.",

    "paymentProcessing": {
        "minConf": 10,
        "enabled": true,
        "paymentMode": "prop",
        "_comment_paymentMode": "prop, pplnt",
        "paymentInterval": 60,
        "minimumPayment": 0.001,
        "maxBlocksPerPayment": 3,
        "_comment_maxBlocksPerPayment": "Throttles how many pending blocks are processed per interval. Can be temporarily raised (e.g. to 10+) to drain a large backlog faster, then lowered back down for steady-state operation.",
        "daemon": {
            "host": "127.0.0.1",
            "port": CHANGE_ME_rpcport,
            "user": "CHANGE_ME_rpcuser",
            "password": "CHANGE_ME_rpcpass"
        }
    },

    "tlsOptions": {
        "enabled": false,
        "serverKey": "",
        "serverCert": "",
        "ca": ""
    },

    "ports": {
        "3421": {
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
            "port": CHANGE_ME_rpcport,
            "user": "CHANGE_ME_rpcuser",
            "password": "CHANGE_ME_rpcpass"
        }
    ],

    "p2p": {
        "enabled": false,
        "host": "127.0.0.1",
        "port": CHANGE_ME_p2pport,
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
        "enabled": true,
        "forks": 1
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
        "enabled": false
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
Description=ZNY-NOMP Mining Pool
After=network.target redis-server.service

[Service]
Type=simple
User=YOUR_SERVICE_USER
WorkingDirectory=/path/to/zny-nomp
Environment=NODE_ENV=production
ExecStart=/home/YOUR_SERVICE_USER/.nvm/versions/node/v20.20.2/bin/node init.js
Restart=always
RestartSec=5
StandardOutput=append:/var/log/zny-nomp/nomp.log
StandardError=append:/var/log/zny-nomp/nomp.log

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo mkdir -p /var/log/zny-nomp
sudo systemctl daemon-reload
sudo systemctl enable zny-nomp
sudo systemctl start zny-nomp
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
