# Linux namespaces are a kernel feature that isolate system resources between processes.

This is the foundation of:

Docker, Kubernetes, Any container runtime

Namespaces isolate: Processes (PID), Network stack, Mount points, Hostname,IPC, Users (UID mapping)


### FUNDAMENTALS

To see all the available namespaces : lsns

Note: It is a Linux command used to display information about Linux namespaces currently active in the system.