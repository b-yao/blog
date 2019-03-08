# How to Fix Ceph Install Usermod Error

When following the documentation of Ceph, after finish [preflight][1] we have a cluster that can be accessed from the ceph admin intance. Then come to quick ceph deploy: [storage cluster quick start][2]. When we install ceph packages, at the command
```
ceph-deploy install node1 node2 node3
```
I could not get through on Ubuntu 16.04 or 18.04, with ceph-deploy 2.0.1 and trying to deploy ceph mimic (13.2.4). The error is basically because ceph-deploy trying to set the ceph user in the node, but at the same time it is ssh over to the instance so the change is declined:
```
[node1][DEBUG ] Setting system user ceph properties..usermod: user ceph is currently used by process 4637
[node1][DEBUG ] dpkg: error processing package ceph-common (--configure):
[node1][DEBUG ]  subprocess installed post-installation script returned error exit status 8
[node1][DEBUG ] dpkg: dependency problems prevent configuration of ceph-base:
[node1][DEBUG ]  ceph-base depends on ceph-common (= 13.2.4-1xenial); however:
[node1][WARNIN] No apport report written because the error message indicates its a followup error from a previous failure.
[node1][DEBUG ]   Package ceph-common is not configured yet.
[node1][WARNIN] No apport report written because the error message indicates its a followup error from a previous failure.
[node1][DEBUG ]
[node1][WARNIN] No apport report written because MaxReports is reached already
[node1][DEBUG ] dpkg: error processing package ceph-base (--configure):
[node1][WARNIN] No apport report written because MaxReports is reached already
[node1][DEBUG ]  dependency problems - leaving unconfigured
[node1][WARNIN] No apport report written because MaxReports is reached already
[node1][DEBUG ] dpkg: dependency problems prevent configuration of ceph-mgr:
[node1][WARNIN] No apport report written because MaxReports is reached already
[node1][DEBUG ]  ceph-mgr depends on ceph-base (= 13.2.4-1xenial); however:
[node1][WARNIN] No apport report written because MaxReports is reached already
[node1][DEBUG ]   Package ceph-base is not configured yet.
```

If I Try to inspect the process "4637" above, it is only possible watching it while executing the `ceph-deploy install node3` command
```
ceph@ceph-node-3:~$ watch -n1 'ps aux | grep ceph'
ceph        4637  0.0  0.0 110080  3564 ?        S    17:11   0:00 sshd: ceph@pts/1
```
This is hilarious!

## Solution
So we manually help ceph-deploy to do the following, which includes the critial usermod change:
- Change the ceph user line in /etc/passwd to
  ```
  ceph:x:1001:1001:Ceph storage service:/var/lib/ceph:/bin/bash
  ```
- Move the .ssh authorized key over so the ceph-admin instance can reach the node:
  ```
  sudo cp -rp /home/ceph/.ssh /var/lib/ceph
  ```
  
## Ask for better solution! 
 
  


[1]: http://docs.ceph.com/docs/master/start/quick-start-preflight/
[2]: http://docs.ceph.com/docs/master/start/quick-ceph-deploy/
