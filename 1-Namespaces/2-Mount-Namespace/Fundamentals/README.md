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




### Mount namespace fundamentals

1. Each mount namespace has its own list of mount points (a private mount table).

2. When you create a new mount namespace (unshare --mount):

It initially inherits the parent’s mount table

From then on, mount/umount operations are private to this namespace unless propagation is set otherwise

Important: The underlying filesystem objects are shared unless you use tmpfs or overlayfs.


### Mount propagation types

Linux allows you to control how mount changes propagate between namespace

| Propagation type | Meaning                                                                        |
| ---------------- | ------------------------------------------------------------------------------ |
| `private`        | Default in containers; changes **do not propagate** to parent/other namespaces |
| `shared`         | Mount/umount changes **propagate** to other shared namespaces                  |
| `slave`          | Changes propagate **from master to slave**, but not back                       |
| `unbindable`     | Mount cannot be propagated at all                                              |


Docker sets / to private → prevents accidental propagation to host

Docker and Linux containers remount /proc and /sys inside each mount namespace:

/proc shows only processes in this PID namespace

/proc/self/mounts shows mounts visible to this namespace

This is why containers see isolated /proc, even if host has many other mounts



### What we learned so far:

1. Mount namespace basics

Each namespace has its own private mount table

Inherits mounts from parent but future mounts/umounts are private

2. tmpfs mounts

Memory-backed, ephemeral, truly isolated

Perfect for /tmp or /dev/shm inside containers

3. Bind mounts

Mount points isolated to namespace

Files themselves are shared with host → content is visible outside

Propagation (private, shared, slave) controls visibility

4. /proc and /sys remounts

Reflect namespace’s PID and mount table

Containers see only their processes and mounts

5. OverlayFS + container root

Container / = merged lower (image) + upper (writable) layer

Mount namespace ensures this root is isolated from host

6. Mount propagation rules

private, shared, slave → control how mount changes flow between namespaces

7. Real Docker analogy

OverlayFS + tmpfs + bind mounts + /proc remounts = full container filesystem isolation