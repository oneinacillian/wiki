> 6 week snapshot listing

**Please note that the following on the snapshots**
- Validated for no missing blocks (used hyperion 3.3.6)
- Syned up to headblock at the time of the snapshot
- The snapshot naming convention is as follows

<img src="/assets/snapshot-name.png"/> <br>

The name (for instance 11-09-22) will be the identifier for which data is contained in the snapshot. <br>
For instance, in the case above, the indexes will contain data populated from 11-00-2022 00:00:00 GMT +2

To restore the snapshot to your hyperion instance, the following will be necessary
1. Update your Elasticsearch.yml to trust the repository from which you will download the snapshotted data

```
repositories.url.allowed_urls: ["http://snapshots.oiac.io/downloads/partialsnapshots/"]
service elasticsearch restart
```

2. Add the repository using dev tools in elasticsearch <br>
**Note:** I name the repository partial-mainnet-snapshots

```
PUT _snapshot/partial-mainnet-snapshots
{
   "type": "url",
   "settings": {
       "url": "http://snapshots.oiac.io/downloads/partialsnapshots/",
       "max_restore_bytes_per_sec": "4gb",
       "max_snapshot_bytes_per_sec": "4gb" 
   }
}
```

1. List the snapshots

```
GET _snapshot/partial-mainnet-snapshots/_all
```

4. Restore the snapshot you want

```
POST _snapshot/partial-mainnet-snapshots/11-09-2022-partial-hyperion-nbnbmnl8qeumcdsn2es8qa/_restore
{
  "indices": "*"
}

