# PID Namespace Demo


###  Step 1:  Observe the Host Reality

```
[root@RHEL ~]# echo $$
1472

[root@RHEL ~]# ps -ef | head
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 11:17 ?        00:00:02 /usr/lib/systemd/systemd --switched-root --system --deserialize=51
root           2       0  0 11:17 ?        00:00:00 [kthreadd]
root           3       2  0 11:17 ?        00:00:00 [pool_workqueue_release]
root           4       2  0 11:17 ?        00:00:00 [kworker/R-rcu_gp]
root           5       2  0 11:17 ?        00:00:00 [kworker/R-sync_wq]
root           6       2  0 11:17 ?        00:00:00 [kworker/R-slub_flushwq]
root           7       2  0 11:17 ?        00:00:00 [kworker/R-netns]
root           9       2  0 11:17 ?        00:00:00 [kworker/0:0H-events_highpri]
root          11       2  0 11:17 ?        00:00:00 [kworker/u8:0-events_unbound]
```

Note:

PID 1 exists already (system init)

Every process has a unique number

The system has one global PID table (in the root namespace)

This is the initial PID namespace created by the kernel at boot.


### Step 2: Create a New PID Namespace

```
[root@RHEL ~]# sudo unshare --fork --pid --mount-proc bash

[root@RHEL ~]# echo $$
1


[root@RHEL ~]# ps -ef | head
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 11:45 pts/2    00:00:00 bash
root          29       1  0 11:46 pts/2    00:00:00 ps -ef
root          30       1  0 11:46 pts/2    00:00:00 head
[root@RHEL ~]# 

```
Note:

When you used: --pid

The kernel:

1. Created a new PID namespace

2. Forked a child process

3. That child became PID 1 inside that namespace

4. But from the host’s perspective, it still has a normal PID.


So the same process now has:

One PID in the host namespace

One PID in the new namespace

This is PID translation across namespace boundaries.

# Step 3:  Confirm Process Dual Identity

From the PID Namespace:

```
[root@RHEL ~]# sleep 1234 &
[1] 32
[root@RHEL ~]# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 11:45 pts/2    00:00:00 bash
root          32       1  0 11:49 pts/2    00:00:00 sleep 1234  # Observe the PID in PID-namespace
root          33       1  0 11:49 pts/2    00:00:00 ps -ef
[root@RHEL ~]# 
```

From the host machine:

```
[root@RHEL ~]# ps -ef | grep "sleep 1234"

root        1574    1539  0 11:49 pts/2    00:00:00 sleep 1234  # Observe the PID in host machine
```

That means:

Inside namespace = PID 32

Outside namespace = PID 1574

Same process. Two identities. Two namespace contexts.


