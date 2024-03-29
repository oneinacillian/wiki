> 6 week snapshot listing <br>

It is clear that the elastic indexes are growing at a rapid pace in terms of sizing. <br>
This will make it extremely difficult to maintain full hyperion history, or even get started since you need to pull an entire snapshot before any maintenance can be done. <br>
Since the OIG still validate 6 weeks of Hyperion history as partial and for running partial Hyperion you receive scoring, I started creating snapshots daily that contains all data later than 6 weeks. <br>

**Please note the following on the snapshots**

- Validated for no missing blocks (used hyperion 3.3.6) <br>
- Syned up to headblock at the time of the snapshot <br>
- The snapshot naming convention is as follows <br>

<img src="/assets/snapshot-name.png"/> <br>

The name (for instance 11-09-22) will be the identifier for which data is contained in the snapshot. <br>
For instance, in the case above, the indexes will contain data populated from 11-09-2022 00:00:00 GMT +2

**To restore the snapshot to your hyperion instance, the following will be necessary**

> Update your Elasticsearch.yml to trust the repository from which you will download the snapshotted data

```
repositories.url.allowed_urls: ["https://snapshots.oiac.io/downloads/partialsnapshots/"]
service elasticsearch restart
```

> Add the repository using dev tools in elasticsearch <br>
**Note:** I named the repository partial-mainnet-snapshots

```
PUT _snapshot/partial-mainnet-snapshots
{
   "type": "url",
   "settings": {
       "url": "https://snapshots.oiac.io/downloads/partialsnapshots/",
       "max_restore_bytes_per_sec": "4gb",
       "max_snapshot_bytes_per_sec": "4gb" 
   }
}
```

> List the snapshots

```
GET _snapshot/partial-mainnet-snapshots/_all
```

> Restore the snapshot you want (in your case the latest will make sense) <br>

<img src="/assets/snapshot_latest.png"/> <br>

```
POST _snapshot/partial-mainnet-snapshots/02-10-2022-partial-hyperion-yrvehaihro2tg-wyx9gvja/_restore
{
  "indices": "*"
}
```

## Current Snapshots

Please find the latest hyperion snapshots for partial restores => [here](./assets/listed-snapshots.txt)

## Latest Restore Validated

```
POST _snapshot/snaps-30/10-10-2022-partial-hyperion-fjo3ryzttdubtrzb-wxocg/_restore
{
  "indices": "*"
}
```

| Index                  | Status   | Last activity              | Shards completed | Shards in progress |
|------------------------|----------|----------------------------|------------------|--------------------|
| wax-perm-v1            | Complete | Nov 23, 2022 7:36 AM GMT+2 | 2                | 0                  |
| wax-abi-v1             | Complete | Nov 23, 2022 7:35 AM GMT+2 | 2                | 0                  |
| wax-delta-v1-000021    | Complete | Nov 23, 2022 7:35 AM GMT+2 | 4                | 0                  |
| wax-block-v1           | Complete | Nov 23, 2022 7:34 AM GMT+2 | 2                | 0                  |
| wax-table-accounts-v1  | Complete | Nov 23, 2022 7:34 AM GMT+2 | 2                | 0                  |
| wax-table-voters-v1    | Complete | Nov 23, 2022 7:31 AM GMT+2 | 2                | 0                  |
| wax-link-v1            | Complete | Nov 23, 2022 7:31 AM GMT+2 | 2                | 0                  |
| wax-table-proposals-v1 | Complete | Nov 23, 2022 7:31 AM GMT+2 | 2                | 0                  |
| wax-logs               | Complete | Nov 23, 2022 7:17 AM GMT+2 | 1                | 0                  |
| wax-action-v1-000021   | Complete | Nov 23, 2022 6:42 AM GMT+2 | 4                | 0                  |
| wax-delta-v1-000022    | Complete | Nov 23, 2022 4:30 AM GMT+2 | 4                | 0                  |
| wax-action-v1-000022   | Complete | Nov 23, 2022 2:26 AM GMT+2 | 4                | 0                  |
| wax-logs-v1            | Complete | Nov 22, 2022 9:25 PM GMT+2 | 2                | 0                  |

```
curl -v http://127.0.0.1:7000/v2/health | json_pp
*   Trying 127.0.0.1:7000...
* TCP_NODELAY set
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Connected to 127.0.0.1 (127.0.0.1) port 7000 (#0)
> GET /v2/health HTTP/1.1
> Host: 127.0.0.1:7000
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< vary: Origin
< access-control-allow-origin: *
< content-type: application/json; charset=utf-8
< content-length: 1016
< Date: Thu, 24 Nov 2022 02:36:23 GMT
< Connection: keep-alive
< Keep-Alive: timeout=5
< 
{ [1016 bytes data]
100  1016  100  1016    0     0   1587      0 --:--:-- --:--:-- --:--:--  1587
* Connection #0 to host 127.0.0.1 left intact
{
   "features" : {
      "deferred_trx" : false,
      "failed_trx" : false,
      "index_all_deltas" : true,
      "index_deltas" : true,
      "index_transfer_memo" : true,
      "resource_limits" : false,
      "resource_usage" : false,
      "streaming" : {
         "deltas" : true,
         "enable" : true,
         "traces" : true
      },
      "tables" : {
         "accounts" : true,
         "proposals" : true,
         "voters" : true
      }
   },
   "health" : [
      {
         "service" : "RabbitMq",
         "status" : "OK",
         "time" : 1669257382974
      },
      {
         "service" : "NodeosRPC",
         "service_data" : {
            "chain_id" : "1064487b3cd1a897ce03ae5b6a865651747e2e152090f99c1d19d44e01aea5a4",
            "head_block_num" : 215586809,
            "head_block_time" : "2022-11-24T02:36:23.000",
            "last_irreversible_block" : 215586474,
            "time_offset" : -49
         },
         "status" : "OK",
         "time" : 1669257382951
      },
      {
         "service" : "Elasticsearch",
         "service_data" : {
            "active_shards" : "100.0%",
            "first_indexed_block" : 207796238,
            "head_offset" : -1,
            "last_indexed_block" : 215586810,
            "missing_blocks" : 0,
            "missing_pct" : "0.00%",
            "total_indexed_blocks" : 7790572
         },
         "status" : "OK",
         "time" : 1669257383566
      }
   ],
   "host" : "hyperion.oiac.io",
   "query_time_ms" : 632.49,
   "version" : "3.3.6",
   "version_hash" : "3b74decbe6b07cad76cedafa57ef2746f0588bde"
}
```

> Indexer <br>

```
/root/.pm2/logs/wax-indexer-out.log last 15 lines:
0|wax-inde | 2022-11-24T02:41:11: [3003206 - 00_master] W:40 | R:2 | C:2 | A:694.6 | D:989.6 | I:1828.4
0|wax-inde | 2022-11-24T02:41:16: [3003206 - 00_master] W:40 | R:2 | C:2 | A:581 | D:580.8 | I:1465.8
0|wax-inde | 2022-11-24T02:41:21: [3003206 - 00_master] W:40 | R:2 | C:2 | A:649.4 | D:905.4 | I:1807.8
0|wax-inde | 2022-11-24T02:41:26: [3003206 - 00_master] W:40 | R:2 | C:1.8 | A:742.4 | D:980.6 | I:1881.8
0|wax-inde | 2022-11-24T02:41:31: [3003206 - 00_master] W:40 | R:1.8 | C:2 | A:827 | D:1069 | I:2099
0|wax-inde | 2022-11-24T02:41:36: [3003206 - 00_master] W:40 | R:2.2 | C:2.2 | A:740.2 | D:943.6 | I:1933.8
0|wax-inde | 2022-11-24T02:41:41: [3003206 - 00_master] W:40 | R:2 | C:2 | A:552.2 | D:787.2 | I:1440
0|wax-inde | 2022-11-24T02:41:46: [3003206 - 00_master] W:40 | R:2 | C:2 | A:655.8 | D:778.2 | I:1792.6
0|wax-inde | 2022-11-24T02:41:51: [3003206 - 00_master] W:40 | R:2 | C:1.8 | A:877.6 | D:1170 | I:2207.6
0|wax-inde | 2022-11-24T02:41:56: [3003206 - 00_master] W:40 | R:2 | C:2.2 | A:691 | D:1012.6 | I:1707.2
0|wax-inde | 2022-11-24T02:42:01: [3003206 - 00_master] W:40 | R:2 | C:2 | A:506.2 | D:611.4 | I:1501.2
0|wax-inde | 2022-11-24T02:42:06: [3003206 - 00_master] W:40 | R:2 | C:2 | A:706 | D:970.4 | I:1890.2
0|wax-inde | 2022-11-24T02:42:11: [3003206 - 00_master] W:40 | R:2 | C:2 | A:1009.2 | D:1437.8 | I:2538.8
0|wax-inde | 2022-11-24T02:42:16: [3003206 - 00_master] W:40 | R:2 | C:2 | A:860.2 | D:954 | I:2359.4
0|wax-inde | 2022-11-24T02:42:21: [3003206 - 00_master] W:40 | R:2 | C:2 | A:878.6 | D:1104.6 | I:2242.4
```

> Estimated restore time (based on only having 4 shards available) <br>

```
21:25 -> 07:36 (9 hours, 11 minutes)
Note that you can restore quicker if you have more nodes available (i.e. more shards to do the work)
```

