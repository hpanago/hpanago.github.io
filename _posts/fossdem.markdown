Talk 1: 10 years of (Puppet) Config Management

- automate things that are the same on all sources
- divide et Impera (one module reponsible for one and only thing)
- Code should be easy to use and easy to read. Document the logic, do not explain the code
- Infra code upgrade
  - Unit tests make the difference
  - Refactoring is always an option
- Design with user in mind (in config management's case, you)
- Make people familiar with the tool
- Share the right info with the right people

Talk 2: Making your MySQL Replication run faster

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


Talk 3: MySLQ Un-Split brain aka Move back in time

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

Talk 4: Cfgmgmt for Cfgmtmt

- Example where pupeptservers are set up manually
- Puppetservers do not support HA (what happens in grnet when p1 can't respond? How much time
until the client, if ever speaks with another puppetmaster?)

