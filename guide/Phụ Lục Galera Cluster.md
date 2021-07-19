Link tham khảo: Recovery cluster https://www.symmcom.com/docs/how-tos/databases/how-to-recover-mariadb-galera-cluster-after-partial-or-full-crash

Cluster control: https://severalnines.com/resources/database-management-tutorials/galera-cluster-mysql-tutorial

### Full Cluster Recovery 

In this scenario, all nodes failed or did not gracefully shutdown. Total loss of quorum occurred and the cluster is not accepting any SQL requests. After a hard crash such as this, even if all the nodes come back online the MariaDB service will be unable to start. This is due to the unclean shutdown and none of the nodes were able to do the last commit. A Galera cluster can crash in various ways resulting in different methods to recover from a full crash.  

Recovery Based On Highest seqno Value
This method is helpful when there is a slight possibility that at least one node was able to gracefully shutdown during the crash. The node with the latest data will have the highest seqno value among all the crashed nodes. We can find the clue in the content of which/var/lib/mysql/grastate.dat will show the value of seqno. Depending on the nature of crash either all of the nodes will have identical negative seqno value or one of the nodes will have the highest positive seqno value.

Following shows the content of grastate.dat in node 3. This node has negative seqno and no group ID. This the case when a node crashes during Data Definition Language (DDL) processing:

```
$ cat /var/lib/mysql/grastate.dat
# GALERA saved state
version: 2.1
uuid: 00000000-0000-0000-0000-000000000000
seqno: -1
safe_to_bootstrap: 0
```

Following shows the content of grastate.dat in node 2. This node crashed during transaction processing resulting in negative seqno but with group ID:

```
$ cat /var/lib/mysql/grastate.dat
# GALERA saved state
version: 2.1
uuid: 886dd8da-3d07-11e8-a109-8a3c80cebab4
seqno: -1
safe_to_bootstrap: 0
```

Following is the content of grastate.dat on node 1 with highest seqno value:

```
$ cat /var/lib/mysql/grastate.dat
# GALERA saved state
version: 2.1
uuid: 886dd8da-3d07-11e8-a109-8a3c80cebab4
seqno: 31929
safe_to_bootstrap: 1
```

Note that a node will only have positive highest seqno value when the node was able to gracefully shutdown. This is the node need to be recovered first.

If all the nodes contain the value of -1 for seqno and 0 for safe_to_bootstrap, that is an indication that a full cluster crash has occurred. At this point, we could start the cluster using the command galera_new_cluster. But it is not recommended at all since there is no way to know that each node has an identical copy of the database data. 

Before restarting the node 1 we need to make a change in the cluster configuration file /etc/my.cnf.d/server.cnf to remove the mention of IPs of cluster nodes. Following is the content of [galera] section of the configuration before any changes:

```
[galera]
# Mandatory settings
wsrep_on=ON
wsrep_provider=/usr/lib64/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://10.20.20.167,10.20.20.168"
wsrep_cluster_name='galeraCluster01'
wsrep_node_address='10.0.0.51'
wsrep_node_name='galera-01'
wsrep_sst_method=rsync
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
```

Note that wsrep_cluster_address shows the IP of all member nodes. We need to remove the addresses as follows:

```
wsrep_cluster_address="gcomm://"
```

We can now restart the mariadb service in this node:

```
$ systemctl restart mariadb
```

Only after verifying that the service started successfully we can proceed to restart services on the other nodes one at a time. Only after all nodes are successfully running, we need to edit the cluster configuration on node 1 to add the IP addresses of all the member nodes and restart the service:

```
wsrep_cluster_address="gcomm://10.20.20.167,10.20.20.168"
```

The Galera cluster should be up and running at this point and all nodes should be syncing with the surviving node.  


## Tóm tắt:

Kiểm tra seqno của từng node. Node nào có seqno cao hơn tức có dữ liệu mới hơn.

### Recovery galera cluster thủ công

1. Tại node có dữ liệu mới hơn:

To restore a Galera cluster manually:

On all Galera dbs nodes:

Stop all the MySQL processes.

Verify that the MySQL processes are stopped:

```
ps aux | grep mysql
```

Identify the last shutdown Galera node:

In the /var/lib/mysql/grastate.dat file on every Galera node, compare the seqno value. The Galera node that contains the maximum seqno value is the last shutdown node.

If the seqno value is equal on all three nodes, identify the node on which the /var/lib/mysql/gvwstate.dat file exists. The Galera node that contains this file is the last shutdown node.

Remove the grastate.dat and ib_logfiles:

```
rm /var/lib/mysql/grastate.dat
rm /var/lib/mysql/ib_logfile*
```

Log in to the last shutdown Galera node.

In /etc/mysql/my.cnf:

Comment out the wsrep_cluster_address line:

```
...
#wsrep_cluster_address="gcomm://192.168.0.1,192.168.0.2,192.168.0.3"
...
```
Add the wsrep_cluster_address parameter without any IP address specified.
```
...
wsrep_cluster_address=gcomm://
...
```

Start MySQL:

```
service mysql start
```

Validate the current status of the Galera cluster:

2. Start MySQL tại node còn lại

```
service mysql start
```

