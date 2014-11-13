
Unix Sockets Peers
==================

Find which process is connected to a given unix socket.

Needed
------

perl, gdb, and decent netstat and lsof.  
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

Or get info for all processes at once. This one adds an extra column to netstat's output:

    # ./netstat_unix
    Proto RefCnt Flags       Type       State         I-Node   PID/Program name     Peer PID/Program name  Path
    unix  3      [ ]         STREAM     CONNECTED     6825     982/Xreal            1497/compiz            /tmp/.X11-unix/X0
    unix  3      [ ]         STREAM     CONNECTED     6824     1497/compiz          982/Xreal                 
    ...


Kernel debug symbols
--------------------

To do without debug symbols, run `find_gdb_offset` script to find the right offset to use on your system, fix the scripts and you're good to go. The scripts will catch it if wrong offset is used.

    # find_gdb_offset 6825
    Offset found, now change hardcoded values in the scripts to:
      my $struct_unix_sock__peer_offset=104;


If you want to use debug symbols (slow), set `$use_kernel_debug_symbols` and change `vmlinux` path.

Details
-------

Implements ideas from:  
http://stackoverflow.com/questions/11897662/identify-other-end-of-a-unix-domain-socket-connection

Note: can't use the inode +1/-1 hack:  
- sometimes it fails, which is fine (can use other method then)
- when it succeeds it works most of the time but there are cases where it yields wrong socket, so no way to rely on it.

So use kernel address from lsof output, and get peer address from kernel with gdb.


TODO
----

- Find a way without using gdb.
