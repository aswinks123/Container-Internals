## What Is a PID Namespace ##

A PID namespace isolates:

Process ID number space

Process visibility

Parent-child relationships

*** Inside a new PID namespace:***

PID numbering starts at 1

Processes outside are invisible

The first process becomes init (PID 1) of that namespace

### Core Rules of PID Namespace

*** 1 PID 1 Is Special : ***

The first process becomes PID 1.

PID 1 has special signal handling behavior.

If PID 1 exits -> the namespace dies.


2 PID Namespaces Are Hierarchical

PID namespaces form a tree.

Parent namespace:

Can see child namespace processes.

Child namespace:

Cannot see parent processes.

This is a one-way visibility model.

3 A Process Has Multiple PIDs

A process can have:

One PID in its own namespace

Another PID in parent namespace

Another in grandparent

This is called PID translation across namespaces.

4. Killing PID 1 Inside Namespace

If PID 1 exits:

Kernel sends SIGKILL to all processes in that namespace.

This is why container crashes when main process exits.

5 Zombies + Orphans

PID 1 is responsible for:

Reaping zombie processes.

Adopting orphaned processes.

If PID 1 doesn’t handle SIGCHLD properly -> zombies accumulate.

This is why container best practice says:

Use tini

Or proper init system

6 PID Namespace Does NOT Isolate Resources

It only isolates:

Process IDs

Visibility

It does NOT isolate:

Filesystem

Network

Memory

That is other namespaces.

7 Kernel Implementation

PID namespaces are implemented using:

clone() system call

CLONE_NEWPID flag

Note: unshare internally uses this.



Command to create a PID namespace:

sudo unshare --fork --pid --mount-proc bash

Explanation: 

unshare: creates a new namespace for the process.
--fork: --fork makes unshare fork a child process, and that child becomes PID 1 inside the namespace. In this exampe bash is PID 1

--pid: Creates a new PID namespace

--mount-proc: /proc must be remounted otherwise ps will still show host processes. Without this, isolation won’t look correct.

bash: Starts a bash shell inside this new namespace. This will be PID 1
