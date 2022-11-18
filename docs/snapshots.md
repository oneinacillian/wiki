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
| 06-10-2022-partial-hyperion-konaakl_rsqwfo9z1byjow | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 18, 2022 5:18 PM GMT+2  | 26s      |
| 05-10-2022-partial-hyperion-taozmbatr4gidxjbxmvqhq | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 17, 2022 4:55 PM GMT+2  | 23s      |
| 04-10-2022-partial-hyperion-q2yijctcs1mpihbz3uhrha | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 16, 2022 6:33 PM GMT+2  | 21s      |
| 03-10-2022-partial-hyperion-yikroyuwtz-1r6wrdhvs3q | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 15, 2022 8:11 PM GMT+2  | 29s      |
| 02-10-2022-partial-hyperion-yrvehaihro2tg-wyx9gvja | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 14, 2022 3:18 PM GMT+2  | 11s      |
| 01-10-2022-partial-hyperion-p4u5tj2crdwccxpgefhujw | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 14, 2022 7:46 AM GMT+2  | 22s      |
| 29-09-2022-partial-hyperion-znziqb85qvwue2yrybj5kq | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 13, 2022 7:08 PM GMT+2  | 82s      |
| 25-09-2022-partial-hyperion-o11gjpk4seemhmisah-nlq | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 07, 2022 9:40 PM GMT+2  | 17s      |
| 24-09-2022-partial-hyperion-2ecalmxrtw2pb8b7bvckda | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 07, 2022 7:31 AM GMT+2  | 22s      |
| 23-09-2022-partial-hyperion-oio39fxqtqwhqwyjc8r5xq | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 06, 2022 3:41 PM GMT+2  | 36s      |
| 22-09-2022-partial-hyperion-niu-a2wpsocn_h-uww2wuq | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 04, 2022 4:05 PM GMT+2  | 22s      |
| 21-09-2022-partial-hyperion-fjgymra9rc2pxgscxcdxha | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 03, 2022 9:37 PM GMT+2  | 25s      |
| 20-09-2022-partial-hyperion-827oflansto_8qiigfrv6g | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 02, 2022 10:04 PM GMT+2 | 24s      |
| 19-09-2022-partial-hyperion-hcvo9vcoteak8qlu5ykugg | partial-mainnet-snapshots | 13      | 33     | 0             | Nov 01, 2022 9:38 PM GMT+2  | 27s      |
| 18-09-2022-partial-hyperion-ndtvbvswqoeo9ui8oaeogg | partial-mainnet-snapshots | 13      | 33     | 0             | Oct 31, 2022 5:15 PM GMT+2  | 15s      |
| 17-09-2022-partial-hyperion-m-djzh_gqtgb8coqyfjquq | partial-mainnet-snapshots | 13      | 33     | 0             | Oct 31, 2022 7:29 AM GMT+2  | 21s      |
| 16-09-2022-partial-hyperion-hkxtaejkqfy0kleaokcuow | partial-mainnet-snapshots | 13      | 33     | 0             | Oct 30, 2022 12:05 AM GMT+2 | 28s      |
| 15-09-2022-partial-hyperion-h-vjskkct2ql3wx5g8dzka | partial-mainnet-snapshots | 13      | 33     | 0             | Oct 28, 2022 5:13 PM GMT+2  | 12s      |
| 14-09-2022-partial-hyperion-glttgjkwq_quacaj6amx4a | partial-mainnet-snapshots | 13      | 33     | 0             | Oct 28, 2022 9:07 AM GMT+2  | 30s      |
| 13-09-2022-partial-hyperion-3rtoqlmkscwhsbkcpceplw | partial-mainnet-snapshots | 13      | 33     | 0             | Oct 26, 2022 9:48 PM GMT+2  | 25s      |
| 12-09-2022-partial-hyperion-33jyfujbtie1nx_1qd-oma | partial-mainnet-snapshots | 13      | 33     | 0             | Oct 25, 2022 8:25 PM GMT+2  | 25s      |
| 11-09-2022-partial-hyperion-nbnbmnl8qeumcdsn2es8qa | partial-mainnet-snapshots | 13      | 33     | 0             | Oct 24, 2022 8:56 PM GMT+2  | 23s      |
| 10-09-2022-partial-hyperion-8iylp-ivsz2xxvmixv1n3q | partial-mainnet-snapshots | 13      | 33     | 0             | Oct 23, 2022 7:29 PM GMT+2  | 26s      |

