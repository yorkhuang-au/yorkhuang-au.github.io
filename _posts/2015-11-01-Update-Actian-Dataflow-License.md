---
layout: post
title: Update Actian Dataflow License
---

## Update Actian Dataflow License

There are two steps. The first is to update license on the server. Then update license on the client side.

### Update Actian Dataflow License on the servers

1. For all nodes, delete the old license files in /opt/dataflow, copy the new license pvDxxx.slc and then change the ownership to actian:actian, change permission to 644.

```
$ su - actian
$ cd /opt/dataflow
$ rm pcD*.slc
$ copy <new pcDxxx.slc license> .
$ chown actian:actian <new pcDxxx.slc license>
$ chmod 644 <new pcDxxx.slc license>
```

2. On the first node (control node), stop and start dataflow

```
Under root user - 
$ cd /etc/init.d/
$ ./dataflow stop
$ ./dataflow status
$ ./dataflow start
$ ./dataflow status
```

### Update Actian Dataflow License on the client sides

Delete and copy the <new pcDxxx.slc license> to the knime folder.

```
cd c:\opt\knime2.11.3
del pcD*.slc
copy <new pcDxxx.slc license> .
```
