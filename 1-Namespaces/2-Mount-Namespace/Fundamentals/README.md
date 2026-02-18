# Mount Namespace (mnt)

### What Mount Namespace Does

Isolates filesystem mount points for a group of processes

Each namespace can have different views of / and other mounted filesystems

Processes in one namespace cannot see or modify mounts in another namespace unless explicitly shared

This is how Docker ensures containers have their own filesystem view.

### Why It’s Important

Enables containers to have isolated filesystems

Allows bind mounts, overlay filesystems, and remounting /proc per namespace

Prevents processes in one namespace from interfering with another’s mounts


### How Docker Use Mount Namespace

Each container has its own mount namespace

Docker mounts its overlay filesystem inside that namespace

Containers see their root filesystem as /, independent of host



### Key Concepts:

1. Private vs shared mounts

mount --make-private / → changes only affect this namespace

mount --make-shared / → changes propagate to other namespaces


2. Creating a Mount Namespace

```
sudo unshare --mount bash
```

Bash becomes the first process of a new mount namespace

Any mount or umount commands now affect only this namespace


3. Remount /proc inside mount namespace

```
mount -t proc proc /proc
```

Ensures /proc reflects processes from this PID namespace

Often combined with PID namespaces in containers
