# Demo of Mount Namespace

### Step 1: Create a new mount namespace 

```
sudo unshare --mount --fork bash

```
--mount → new mount namespace

--fork → starts a new shell in that namespace

Note: By default, a new mount namespace inherits the host mount table.

Means the new mount namespace can see the host mounts like / , /home etc.


### How to make it truly isolated?

```
mkdir /tmp/newroot
mount --bind /tmp/newroot /   # temporarily replace /
```

Now / in this shell is /tmp/newroot, not the host /

If you look at / now, you won’t see host files

This is how containers prevent access to host files

We will learn about how container implement sepatate / file system in Overlay file system session.


### Step 2: tmpfs mount (memory-only filesystem)

Now create a tempfs mount - For demo purpose

```

[root@RHEL ~]# mkdir /mnt/tmpfs

mount -t tmpfs tmpfs /mnt/tmpfs

df -h /mnt/tmpfs

Filesystem      Size  Used Avail Use% Mounted on
tmpfs           853M     0  853M   0% /mnt/tmpfs
[root@RHEL ~]# 

```
Now our container/Namespace can see this mount (/mnt/tmpfs)

Lets analyse the host machine and check whether host can see this mount.

From the host run the df command:

```
[root@RHEL ~]# df -h  | grep /mnt/tmpfs
[root@RHEL ~]# df -h  | grep "/mnt/tmpfs"
[root@RHEL ~]# 
```

You can see the mount we created inside the namespace is isolated and host/another namespace cannot see it.



## Additional Types of mounts:

### Bind mount

1. Prepare host directory:

```
mkdir -p /tmp/demo
echo "host original file" > /tmp/demo/hostfile.txt
ls -l /tmp/demo

```

This is your host directory we will bind mount


2. Bind mount into namespace

```
mkdir /mnt/test
mount --bind /tmp/demo /mnt/test
ls -l /mnt/test
```

Features:

The contents are shared because it’s the same underlying directory. So host can see the actual file "/tmp/demo"

The mount point /mnt/test itself is only visible in this namespace. So host cannot see the mount "/mnt/test"
