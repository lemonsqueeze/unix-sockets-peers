
Unix Sockets Peers
==================

Implement ideas from:
  http://stackoverflow.com/questions/11897662/identify-other-end-of-a-unix-domain-socket-connection
to find which process is connected to a given unix socket.

This should always work: when `inode+1/-1` hack doesn't work, address is read from kernel memory.

Needed: perl, gdb, and decent netstat and lsof.  
You must be root.  
Kernel debug symbols are not required (see below).  
This is linux only.  


Demo
----

    # netstat -na --unix -p  
    Proto RefCnt Flags       Type       State         I-Node   PID/Program name    Path
    unix  3      [ ]         STREAM     CONNECTED     6825     982/Xorg            /tmp/.X11-unix/X0
    ...

What is this Xorg socket connected to ??

    # ./socket_peer 6825
    1497 compiz

Even better, get info for all processes at once. This one adds an extra column to netstat's output:

    # ./socket_peers
    Proto RefCnt Flags       Type       State         I-Node   PID/Program name     Peer PID/Program name  Path
    unix  3      [ ]         STREAM     CONNECTED     6825     982/Xreal            1497/compiz            /tmp/.X11-unix/X0
    unix  3      [ ]         STREAM     CONNECTED     6824     1497/compiz          982/Xreal                 
    ...


Do without kernel debug symbols
--------------------------------

Run find_gdb_offset script to find the right offset to use on your system, fix the scripts and you're good to go.
