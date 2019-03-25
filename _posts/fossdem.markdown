### Talk 1: Ten years of Puppet installations: what now?

- automate things that are the same on all sources
- divide et Impera (one module reponsible for one and only thing)
- Code should be easy to use and easy to read. Document the logic, do not explain the code
- Infra code upgrade
  - Unit tests make the difference
  - Refactoring is always an option
- Design with user in mind (in config management's case, you)
- Make people familiar with the tool
- Share the right info with the right people

#### Link: https://fosdem.org/2019/schedule/event/ten_years_puppet/

### Talk 2: Making your MySQL Replication run faster

- You always need monitoring
  - For master: sustained throughtput under normal load and peak load at disaster time
  
- Masters do both read and writes, while slaves do only writes
- They should both have same hardware specs.

- Parallel_applier = LOGICAL_CLOCK
- track non-conflicting transactions
  - transactions are considered independent if they change non-overlapping rows
- write-set based dependency tracing 
  - detect if they can tun on parallel on the slave or a multi-master setup
- slave_preserve_commit_order = on
- with these options a slave can catch up faster after a "slave stop"
- if network is slow, the above operation will be slow (so it adds network load)

#### Link: https://fosdem.org/2019/schedule/event/mysql_replication_faster/

### Talk 3: MySQL Un-Split brain aka Move back in time

- Problem desc: DC went down
  - Master and replicas -> network disconnected
- Failover to another DC but the old master is still getting writes
- At the time the problem was observed, the newly promoted master already has 30 minutes
worth of writes 
(they fixed it with -> GET state FROM backup;)

Theoritical:
- gh_mysql-rewind
- GTIDs
  - they keep records for every transaction even thrown away ones
- So get the GTIDs that have diverged between the old master and the new one, since
we can not just replicate, they both have data the other one does not.

- MariaDB has the flashback option. MySQL does not
- So mysql --binlog & mysql binlog --flashback which is the reverse of the actual binlog
- The wanted result is to get the "bad" master back in time at exactly the time of the split.
(Yes, any new writes on the bad master are considered faulty and we don't want them).

MariaDB does not spea MySQL GTIDs.
`awk` -> fixed.
And the then `cat` the output to MysSQL master.
Then we go back. But at what moment in time?
Match the old GTID with the ones create from the unsplit
- RESET MASTER
- SET gtid_purged:<number>

Do some mathematical computations between old GTID and new one to find the correct GTIDs.

- We can *not* yet predict where gh_mysql_rewind will drop up at.
- The state of the project is just a 20K bash script running in production
- The do manul fuckups to test it

#### Link: https://fosdem.org/2019/schedule/event/unplitmysql/

### Talk 4: Cfgmgmt for Cfgmtmt

- Example where pupeptservers are set up manually
- Puppetservers do not support HA (what happens in grnet when p1 can't respond? How much time
until the client, if ever speaks with another puppetmaster?)
- Foreman -> lifecycle management
  -> manage services via smartproxy -> pxe/tftpd/dhcp/dns
  
? But how about foreman and bare metal provisioning

The speaker user cloud init -> foreman-installer -> puppet -> ansible -> lxc setup 
just to provision a virtual machine

#### Link: https://fosdem.org/2019/schedule/event/use_configmanagement/

### Talk 5: Declare your Linux Network State (Linux networking with nmstate)

- nmstate -> absract network configuration e.g. higher than iproute2

#### Link: https://fosdem.org/2019/schedule/event/declare_linux_network_state/

### Talk 6: Sysadmins, too, deserve stability

- ansible from redhat
- do differrent things on various hosts with different OS

#### Link: https://fosdem.org/2019/schedule/event/sysadmins_deserve_interface_stability/

### Talk 7: Ceph wire protocol revisited - Messenger V2

- Messenger is a wire-protocol specification and software implementation
- Small communication lirary, used for other Ceph components in order to talk to each other
- Abstracs the actual transport protocol (sockets, RDMA, DPDK)
- Automatic handling of temporary connections/failures
- It has an API

- Wire protocol -> Ceph component -> port 3300 -> v1 will still be available on port 6789

- only userspace libraries will support v2
- supports encryption-on-wire
- can authenticate via cephx (connectors and acceptors, that is)
- uses non-blocking sockets
  - Depends on OS sockets to report their status (open, closed, can receive or not)
  
#### Link: https://fosdem.org/2019/schedule/event/ceph_msgrv2/
  
### Talk 8: How we use Gluster

- Gluster as a Service
- uses nfs to give storage to hosts
- ganesha proxy, haproxy in active_active mode
- They also use the Heketi tol
- Ansible to deploy Heketi and create nfs volumes

#### Link: https://fosdem.org/2019/schedule/event/gluster_as_a_service/

### Talk 9: The Container Storage Interface, Explained

- Tries to be storage agnostic 
  - Use this instead of different libraries for each type of storage one may have
- Attaches after the container orchestrator

#### Link: https://fosdem.org/2019/schedule/event/container_storage_interface_explained/

### Talk 10: What's new in Ceph Nautilus

- We get a better Dashboard -> built-in, self-hosted part of Ceph
- Management functions
- Monitoring
-orchestrator.py
  -> blink devices!
- Unified CLL for Ceph daemons
  - ceph orchestrator osd ...
  - ceph orchestrator ...
- pg_num can be redused and automatically tuned in the background, based on expected usage
- osds and mons report underlying storage metrics, scraping SMART metrics
 - on failure prediction it automatically marks osds as "out"
 
 - We also get crash reports on /var/lib/ceph/crash
 - It can send data back to Ceph org via telemetry
 - Messenger v2
  - future support for both ipv4 and ipv6(and you may switch in between)
 - You can tell an osd how much RAM to use (monitors RSS size)
 - Has a progress bar when ceph cluster is under recovery (use `ceph progress`)
 - Also can pin osds to certain nodes via NUMA managment
 - "Misplaced" is not a HEALTH_WARN anymore by default
 
 - Bluestore
  - New "bitmap" allcoator
  - per_poll utilization metrics
  - balance memory allocation between RocksDB cache, Bluestore inodes, data
- New "clay" erasure code plugin
  - better recovery
  - efficiency when <m nodes fail in a n+m cluster
  
- RGW
  - subscribe to events like PUT
  - push interface to AMQ, Kafka
  - Beast frontend for RGW based on boost.asio
  
- RBD
  - namespace support
  - rbd_mirror made simpler
  - pool - lees config overrides -> simpler
  - creation, access, modification timestaps
  
- CephFS
  - multi_fs volume support with stable release
  - subvolume concept
    - sub_dir of a volume with quota, unique cephx user and restricted to a Rados namespace
 
 - CephFs NAS Gateways 
  - clustered nfs_ganeesha which uses RADOS for configuration
  - cephfs CLI 
  - performance improvments
 
- Expose storage to Kubernetes
- Run ceph clusters in Kubernetes
- Kubernetes as a distributed "OS"
- Rook storage orchestrator for Kubernetes ("operator for Ceph in Kubernetes")

#### Link: https://fosdem.org/2019/schedule/event/ceph_project_status_update/
