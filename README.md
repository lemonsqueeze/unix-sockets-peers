
Unix Sockets Peers
==================

Find out which process is connected to a given unix socket.  
For now the kernel doesn't expose it so this is needed.

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
    unix  3      [ ]         STREAM     CONNECTED     6825     982/Xorg             1497/compiz            /tmp/.X11-unix/X0
    unix  3      [ ]         STREAM     CONNECTED     6824     1497/compiz          982/Xorg                 
    unix  3      [ ]         SEQPACKET  CONNECTED     207142   3770/chromium-brows  17783/UMA-Session-R       
    unix  3      [ ]         STREAM     CONNECTED     204903   1523/pulseaudio      3703/thunderbird       
    unix  3      [ ]         STREAM     CONNECTED     204902   3703/thunderbird     1523/pulseaudio           
    unix  3      [ ]         STREAM     CONNECTED     204666   1523/pulseaudio      3703/thunderbird       
    ...

Use `netstat_unix --dump` to get easy to parse output. Format is:

    name:pid:socket:peer_socket:peer_pid:peer_name

    # ./netstat_unix --dump
    gconfd-2:1268:5467:5468:1202:gnome-terminal
    gconfd-2:1268:7852:7850:1541:notification-a
    notification-a:1541:7834:7835:1235:dbus-daemon
    Xorg:993:6532:6530:1346:gnome-panel



Kernel debug symbols
--------------------

To do without debug symbols, run `find_gdb_offset` script to find the right offset to use on your system, fix the scripts and you're good to go. `netstat_unix` will catch it if wrong offset is used.

    # ./find_gdb_offset 6825
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
