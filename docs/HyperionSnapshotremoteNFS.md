# The following will be covered here

- Configure NFS server on new (elasticsearch) Hyperion server
- Configure NFS client on current (elasticsearch) Hyperion server
- Test NFS share
- Configure snapshot repository with throttling enabled on current Hyperion server
- Configure snapshot repository on new Hyperion server <br> 

> Please note that you will need to have a VPN configured between the 2 Hyperion server <br>
> In this example: <br>
> 172.168.40.185 = new Hyperion server <br> 
> 172.168.40.160 = current Hyperion server <br>

## Configure NFS server on new Hyperion server <br>

```
apt-get install nfs-kernel-server
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```

Continue to set up your export (client access i.e current Hyperion server)

- edit /etc/export <br>
- add the following line **/data/snapshots 172.168.40.160(rw,sync,no_root_squash)** <br>
  /data/snapshots = the path on disk to where NFS should map <br>
  172.168.40.160(rw,sync,no_root_squash) = granted access to private host on the VPN (in this case, current hyperion server) <br>

## Configure NFS client on current Hyperion server <br>

```
apt-get install nfs-common
mount -t nfs 172.168.40.185:/data/snapshots /data/snapshots
```

Take note for the mount command above: <br>
- 172.168.40.185:/data/snapshots = VPN address of NFS server (new Hyperion) and NFS directory mapping <br>
- /data/snapshots will be the mount point the current Hyperion server will use to access the NFS share on the new Hyperion server <br>

## Test NFS share

On your current Hyperion server, copy a large file to the /data/snapshots directory and confirm it is placed on the new Hyperion server, also within the /data/snapshots directory <br>
If this works, NFS is configure and you can continue to configure your snapshot repository <br>

## Configure snapshot repository with throttling enabled on current Hyperion server

> Please note to add your path.repo to your elasticsearch.yml <br>

```
path.repo: ["/data/snapshots/"]
```

Create a new snapshot repository <br>

```
PUT /_snapshot/hyperion_testnet_snapshots
{
  "type": "fs",
  "settings": {
    "location": "/data/snapshots",
    "compress": true,
    "max_restore_bytes_per_sec": "4gb",
    "max_snapshot_bytes_per_sec": "10mb"    
  }
}
```

<span style="color:red">**Please note the above**</span> <br>

- "type": "fs" = snapshot will be created on a shared filesystem
- "location": "/data/snapshots" = the directory we mapped to the NFS share
- "max_snapshot_bytes_per_sec": "10mb" = You might need to throttle your snapshot throughput to ensure your network capacity does not impact your current API hosting

You can then go ahead and configure the snapshot policy and kick off the index snapshotting

```
PUT /_slm/policy/hyperion_testnet_snapshot
{
  "schedule": "0 0 0 1 * ?", 
  "name": "hyperion_testnet_snapshot", 
  "repository": "hyperion_testnet_snapshots", 
  "config": { 
    "indices": ["wax-*"], 
    "ignore_unavailable": false,
    "include_global_state": false
  }
}
```

> note that you can also do a few indexes at one time, depending on how large the disk is on the new Hyperion server for accommodating the snapshot as well as the restored index

## Configure snapshot repository on new Hyperion server

Create the snapshot repository on the new hyperion server as well

```
PUT /_snapshot/hyperion_testnet_snapshots
{
  "type": "fs",
  "settings": {
    "location": "/data/snapshots",
    "compress": true,
    "max_restore_bytes_per_sec": "4gb",
    "max_snapshot_bytes_per_sec": "10mb"    
  }
}
```

You can then restore the snapshots which has been created, either using dev tools, or Kibana UI <br>
<span style="color:red">**Keep in mind that before restoring the snapshot, ensure the elasticsearch user ownership on the /data/snapshots directory**</span> <br>