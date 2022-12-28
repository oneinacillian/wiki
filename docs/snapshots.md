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

| Snapshot                                           | Repository                | Indices | Shards | Failed shards | Date created                | Duration |
|----------------------------------------------------|---------------------------|---------|--------|---------------|-----------------------------|----------|
| 13-11-2022-partial-hyperion-99zpmf08reqoppoovszmsa | partial-mainnet-snapshots | 15      | 41     | 0             | Dec 28, 2022 9:30 AM GMT+2  | 39s      |
| 11-11-2022-partial-hyperion-wuohvc7yq82appnpqpypwg | partial-mainnet-snapshots | 15      | 41     | 0             | Dec 26, 2022 6:10 PM GMT+2  | 48s      |
| 10-11-2022-partial-hyperion-8485dlr1ttwsgigj08kbma | partial-mainnet-snapshots | 15      | 41     | 0             | Dec 24, 2022 10:07 AM GMT+2 | 30s      |
| 06-11-2022-partial-hyperion-_r1fvotkrmonbqp2ptpfgw | partial-mainnet-snapshots | 15      | 41     | 0             | Dec 23, 2022 10:33 PM GMT+2 | 65s      |
| 05-11-2022-partial-hyperion-m1i26xnrqe6ftrkqmqo8za | partial-mainnet-snapshots | 15      | 41     | 0             | Dec 20, 2022 1:03 PM GMT+2  | 42s      |
| 04-11-2022-partial-hyperion-emotqkkys4ojrtg5dbthsa | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 18, 2022 11:13 AM GMT+2 | 28s      |
| 03-11-2022-partial-hyperion-hnfm1u7xtu20dhbjgcziaw | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 17, 2022 7:43 AM GMT+2  | 27s      |
| 02-11-2022-partial-hyperion-zrpiuvuitj6uneunze8azg | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 16, 2022 9:34 AM GMT+2  | 35s      |
| 01-11-2022-partial-hyperion-cedu12r4tiahhcuuthsi1g | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 14, 2022 2:45 PM GMT+2  | 8s       |
| 31-10-2022-partial-hyperion-pw-ujdc4rr6uxg4nvastyq | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 14, 2022 11:12 AM GMT+2 | 35s      |
| 30-10-2022-partial-hyperion-fys_rg89sw6eiaeqho2ncw | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 12, 2022 5:22 PM GMT+2  | 22s      |
| 26-10-2022-partial-hyperion-enj1utkvsi2t7hx-yjvczw | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 12, 2022 7:22 AM GMT+2  | 59s      |
| 25-10-2022-partial-hyperion-kv0frsr6rvgzrbhrs82paw | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 08, 2022 8:51 PM GMT+2  | 26s      |
| 24-10-2022-partial-hyperion-fe608yuztswylja8yr2sqa | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 07, 2022 1:56 PM GMT+2  | 41s      |
| 23-10-2022-partial-hyperion-xbhcmlb2qamfxerceskiya | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 05, 2022 7:50 PM GMT+2  | 13s      |
| 22-10-2022-partial-hyperion-ld2cc5-lqbe83j5g4uk9bw | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 05, 2022 7:15 AM GMT+2  | 30s      |
| 21-10-2022-partial-hyperion-rlzt-kfkrpqx-xp6jldtjq | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 04, 2022 10:11 AM GMT+2 | 38s      |
| 18-10-2022-partial-hyperion-dgcjlsjmqes8xq99ktmbgg | partial-mainnet-snapshots | 13      | 33     | 0             | Dec 03, 2022 10:03 AM GMT+2 | 61s      |
| 17-10-2022-partial-hyperion-hz1_ijhmqxcb2mqkj-gxzg | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 30, 2022 7:47 AM GMT+2  | 24s      |
| 16-10-2022-partial-hyperion-u8t7bggwsziai3g4khk8og | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 29, 2022 7:20 AM GMT+2  | 33s      |
| 15-10-2022-partial-hyperion-ltftyliyqiov6yz_vxv8og | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 27, 2022 11:41 AM GMT+2 | 18s      |
| 14-10-2022-partial-hyperion-pjf76n5prvm8fposhca_bw | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 26, 2022 9:52 PM GMT+2  | 27s      |
| 13-10-2022-partial-hyperion-qbnlzswnsuizyb8t-6eeyw | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 26, 2022 5:36 PM GMT+2  | 40s      |
| 12-10-2022-partial-hyperion-xvgrp-l1slgbfyw2y_d_oa | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 24, 2022 10:31 PM GMT+2 | 34s      |
| 11-10-2022-partial-hyperion-fesewejtrhgv92xp515jpg | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 23, 2022 8:40 PM GMT+2  | 21s      |
| 10-10-2022-partial-hyperion-fjo3ryzttdubtrzb-wxocg | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 22, 2022 9:11 PM GMT+2  | 32s      |
| 09-10-2022-partial-hyperion-13ibqjmnq8eweed_f79q6q | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 21, 2022 8:35 PM GMT+2  | 15s      |
| 08-10-2022-partial-hyperion-ymtnf2jws3w-ffwefvqyiw | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 21, 2022 7:43 AM GMT+2  | 22s      |
| 07-10-2022-partial-hyperion-paeeudburdea4nvcz37z4w | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 19, 2022 10:38 PM GMT+2 | 31s      |
| 06-10-2022-partial-hyperion-konaakl_rsqwfo9z1byjow | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 18, 2022 5:18 PM GMT+2  | 26s      |
| 05-10-2022-partial-hyperion-taozmbatr4gidxjbxmvqhq | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 17, 2022 4:55 PM GMT+2  | 23s      |


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
