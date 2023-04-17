[TOC]

### NameSpace Cgroup

》Arm64 Centos 8

#### Cgroups

》Linux Control Groups; 限制进程使用资源上限 [cpu, memory, net, io, devices, pids]

~~~bash
[root@g7 ~]# mount -t cgroup

cgroup on /sys/fs/cgroup/
~~~

概念

- [ ] tasks: task可理解为进程(表现出来就是进程ID和线程ID列表)
- [ ] cgroups: 控制组, 按照某种标准划分的tasks, 资源是以进程组为单位实现的, 一个进程加入某控制组就会受到其资源限制
- [ ] subsystem: 子系统, 即是可限制的具体资源

~~~bash
[root@g7 ~]# cat /proc/cgroups
#subsys_name	hierarchy	num_cgroups	enabled
cpuset	4	3	1
cpu	2	93	1
cpuacct	2	93	1
blkio	10	93	1
memory	8	156	1
devices	3	93	1
freezer	6	3	1
net_cls	9	3	1
perf_event	5	3	1
net_prio	9	3	1
pids	7	122	1

[root@g7 ~]# ll /sys/fs/cgroup/
total 0
dr-xr-xr-x. 6 root root  0 Aug  3 21:21 blkio
lrwxrwxrwx. 1 root root 11 Aug  3 21:21 cpu -> cpu,cpuacct
dr-xr-xr-x. 6 root root  0 Aug  3 21:21 cpu,cpuacct
lrwxrwxrwx. 1 root root 11 Aug  3 21:21 cpuacct -> cpu,cpuacct
dr-xr-xr-x. 3 root root  0 Aug  3 21:21 cpuset
dr-xr-xr-x. 6 root root  0 Aug  3 21:21 devices
dr-xr-xr-x. 3 root root  0 Aug  3 21:21 freezer
dr-xr-xr-x. 6 root root  0 Aug  3 21:21 memory
lrwxrwxrwx. 1 root root 16 Aug  3 21:21 net_cls -> net_cls,net_prio
dr-xr-xr-x. 3 root root  0 Aug  3 21:21 net_cls,net_prio
lrwxrwxrwx. 1 root root 16 Aug  3 21:21 net_prio -> net_cls,net_prio
dr-xr-xr-x. 3 root root  0 Aug  3 21:21 perf_event
dr-xr-xr-x. 6 root root  0 Aug  3 21:21 pids
dr-xr-xr-x. 6 root root  0 Aug  3 21:21 systemd
~~~

- [ ] hierarchy: cgroup 是层级组织关系, 子字节继承父节点的配置属性

~~~bash
[root@g7 ~]# ll /sys/fs/cgroup/memory/
total 0
-rw-r--r--.  1 root root 0 Aug  3 22:24 cgroup.clone_children
--w--w--w-.  1 root root 0 Aug  3 22:24 cgroup.event_control
-rw-r--r--.  1 root root 0 Aug  3 22:24 cgroup.procs
-r--r--r--.  1 root root 0 Aug  3 22:24 cgroup.sane_behavior
drwxr-xr-x.  3 root root 0 Aug  3 22:29 default
drwxr-xr-x.  2 root root 0 Aug  3 21:21 init.scope
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.failcnt
--w-------.  1 root root 0 Aug  3 22:24 memory.force_empty
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.kmem.failcnt
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.kmem.limit_in_bytes
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.kmem.max_usage_in_bytes
-r--r--r--.  1 root root 0 Aug  3 22:24 memory.kmem.slabinfo
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.kmem.tcp.failcnt
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.kmem.tcp.limit_in_bytes
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.kmem.tcp.max_usage_in_bytes
-r--r--r--.  1 root root 0 Aug  3 22:24 memory.kmem.tcp.usage_in_bytes
-r--r--r--.  1 root root 0 Aug  3 22:24 memory.kmem.usage_in_bytes
-rw-r--r--.  1 root root 0 Aug  3 21:21 memory.limit_in_bytes
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.max_usage_in_bytes
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.memsw.failcnt
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.memsw.limit_in_bytes
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.memsw.max_usage_in_bytes
-r--r--r--.  1 root root 0 Aug  3 22:24 memory.memsw.usage_in_bytes
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.move_charge_at_immigrate
-r--r--r--.  1 root root 0 Aug  3 22:24 memory.numa_stat
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.oom_control
----------.  1 root root 0 Aug  3 22:24 memory.pressure_level
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.soft_limit_in_bytes
-r--r--r--.  1 root root 0 Aug  3 22:24 memory.stat
-rw-r--r--.  1 root root 0 Aug  3 22:24 memory.swappiness
-r--r--r--.  1 root root 0 Aug  3 22:24 memory.usage_in_bytes
-rw-r--r--.  1 root root 0 Aug  3 21:21 memory.use_hierarchy
-rw-r--r--.  1 root root 0 Aug  3 22:24 notify_on_release
-rw-r--r--.  1 root root 0 Aug  3 22:24 release_agent
drwxr-xr-x. 89 root root 0 Aug  3 21:21 system.slice
-rw-r--r--.  1 root root 0 Aug  3 22:24 tasks
drwxr-xr-x.  3 root root 0 Aug  3 21:21 user.slice
~~~

如果在某个子系统下, 创建一个控制组[文件即是接口, 可创建、可删除], 即可发现该目录下自动创建父级同样的资源管理文件.

例如, 在cpu子系统下, 自己创建控制组

~~~bash
[root@g7 ~]# mkdir /sys/fs/cgroup/cpu/cg.test
[root@g7 ~]# ll /sys/fs/cgroup/cpu/cg.test
total 0
-rw-r--r--. 1 root root 0 Aug  4 00:15 cgroup.clone_children
-rw-r--r--. 1 root root 0 Aug  4 00:15 cgroup.procs
-rw-r--r--. 1 root root 0 Aug  4 00:15 cpu.cfs_period_us
-rw-r--r--. 1 root root 0 Aug  4 00:15 cpu.cfs_quota_us
-rw-r--r--. 1 root root 0 Aug  4 00:15 cpu.shares
-r--r--r--. 1 root root 0 Aug  4 00:15 cpu.stat
-r--r--r--. 1 root root 0 Aug  4 00:15 cpuacct.stat
-rw-r--r--. 1 root root 0 Aug  4 00:15 cpuacct.usage
-r--r--r--. 1 root root 0 Aug  4 00:15 cpuacct.usage_all
-r--r--r--. 1 root root 0 Aug  4 00:15 cpuacct.usage_percpu
-r--r--r--. 1 root root 0 Aug  4 00:15 cpuacct.usage_percpu_sys
-r--r--r--. 1 root root 0 Aug  4 00:15 cpuacct.usage_percpu_user
-r--r--r--. 1 root root 0 Aug  4 00:15 cpuacct.usage_sys
-r--r--r--. 1 root root 0 Aug  4 00:15 cpuacct.usage_user
-rw-r--r--. 1 root root 0 Aug  4 00:15 notify_on_release
-rw-r--r--. 1 root root 0 Aug  4 00:15 tasks
~~~

运行个进程测试: 注意记住进程号(可通过top查看)

~~~python
while True:
    pass
~~~

~~~bash
top - 00:12:31 up  2:50,  1 user,  load average: 0.43, 0.12, 0.03
Tasks: 137 total,   2 running, 135 sleeping,   0 stopped,   0 zombie
%Cpu(s): 50.0 us,  0.0 sy,  0.0 ni, 50.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   3398.4 total,   2380.8 free,    291.2 used,    726.4 buff/cache
MiB Swap:   4040.0 total,   4040.0 free,      0.0 used.   3056.9 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   3169 root      20   0   12164   8236   5156 R 100.0   0.2   0:18.66 python3
      1 root      20   0  107056  13168   7916 S   0.0   0.4   0:01.42 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.01 kthreadd
~~~

将进程号3169写入tasks, 10000写入quota, 占period的十分之一

~~~bash
[root@g7 ~]# echo 3169 > /sys/fs/cgroup/cpu/cg.test/tasks
[root@g7 ~]# echo 10000 > /sys/fs/cgroup/cpu/cg.test/cpu.cfs_quota_us
[root@g7 ~]# top
top - 00:17:14 up  2:55,  1 user,  load average: 0.93, 0.69, 0.31
Tasks: 137 total,   2 running, 135 sleeping,   0 stopped,   0 zombie
%Cpu(s):  6.2 us,  0.3 sy,  0.0 ni, 93.3 id,  0.0 wa,  0.2 hi,  0.0 si,  0.0 st
MiB Mem :   3398.4 total,   2380.5 free,    291.4 used,    726.4 buff/cache
MiB Swap:   4040.0 total,   4040.0 free,      0.0 used.   3056.7 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   3169 root      20   0   12164   8236   5156 R   9.9   0.2   4:39.76 python3
   1454 root      20   0   46992   5024   3772 S   0.3   0.1   0:01.90 sshd
~~~

运行一个nginx容器测试内存限制

~~~bash
[root@g7 ~]# nerdctl run -d --name ng -m 50m nginx:alpine

# 容器默认都在 default 
[root@g7 ~]# ll /sys/fs/cgroup/memory/default/
total 0
drwxr-xr-x. 2 root root 0 Aug  3 23:18 a388edb818f7e1c96c5c3a0778ad6e6111d266db2eb8a1bcf37402bb34c66926
-rw-r--r--. 1 root root 0 Aug  3 22:29 cgroup.clone_children
--w--w--w-. 1 root root 0 Aug  3 22:29 cgroup.event_control
-rw-r--r--. 1 root root 0 Aug  3 22:29 cgroup.procs
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.failcnt
--w-------. 1 root root 0 Aug  3 22:29 memory.force_empty
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.kmem.failcnt
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.kmem.limit_in_bytes
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.kmem.max_usage_in_bytes
-r--r--r--. 1 root root 0 Aug  3 22:29 memory.kmem.slabinfo
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.kmem.tcp.failcnt
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.kmem.tcp.limit_in_bytes
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.kmem.tcp.max_usage_in_bytes
-r--r--r--. 1 root root 0 Aug  3 22:29 memory.kmem.tcp.usage_in_bytes
-r--r--r--. 1 root root 0 Aug  3 22:29 memory.kmem.usage_in_bytes
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.limit_in_bytes
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.max_usage_in_bytes
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.memsw.failcnt
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.memsw.limit_in_bytes
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.memsw.max_usage_in_bytes
-r--r--r--. 1 root root 0 Aug  3 22:29 memory.memsw.usage_in_bytes
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.move_charge_at_immigrate
-r--r--r--. 1 root root 0 Aug  3 22:29 memory.numa_stat
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.oom_control
----------. 1 root root 0 Aug  3 22:29 memory.pressure_level
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.soft_limit_in_bytes
-r--r--r--. 1 root root 0 Aug  3 22:29 memory.stat
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.swappiness
-r--r--r--. 1 root root 0 Aug  3 22:29 memory.usage_in_bytes
-rw-r--r--. 1 root root 0 Aug  3 22:29 memory.use_hierarchy
-rw-r--r--. 1 root root 0 Aug  3 22:29 notify_on_release
-rw-r--r--. 1 root root 0 Aug  3 22:29 tasks
~~~

总结

cgroup就是通关Linux提供的文件即接口的形式, 来进行资源限制. 在对应子系统(cpu, memery)下建立控制组, 修改子系统下具体文件的配置, 然后将进程号写入tasks即可.

#### NameSpace

Linux 提供用于隔离进程树、网络接口、挂载点、进程见通信等资源的方式; 有如下的命名空间;

修改了进程对整个系统的视图

|    namespace     |                   description                   |
| :--------------: | :---------------------------------------------: |
|  ipc namespace   | IPC资源, 进程间通信(共享内存、消息队列、信号量) |
|  net namespace   |           网络设备、网络栈、端口隔离            |
|  mnt namespace   |               文件系统挂载点隔离                |
|  pid namespace   |                    进程隔离                     |
|  user namespace  |                  用户和组隔离                   |
|  uts namespace   |                 主机和域名隔离                  |
| cgroup namespace |                cgroup根目录隔离                 |

~~~bash
[root@g7 ~]# nerdctl run -d --name ng -m 50m nginx:alpine
[root@g7 ~]# lsns
        NS TYPE   NPROCS   PID USER   COMMAND
4026531835 cgroup    134     1 root   /usr/lib/systemd/systemd ...
4026531836 pid       131     1 root   /usr/lib/systemd/systemd ...
4026531837 user      134     1 root   /usr/lib/systemd/systemd ...
4026531838 uts       131     1 root   /usr/lib/systemd/systemd ...
4026531839 ipc       131     1 root   /usr/lib/systemd/systemd ...
4026531840 mnt       124     1 root   /usr/lib/systemd/systemd ...
4026531860 mnt         1    22 root   kdevtmpfs
4026531961 net       131     1 root   /usr/lib/systemd/systemd 
4026532119 mnt         1   628 root   /usr/lib/systemd/systemd-udevd
4026532235 mnt         2   765 root   /sbin/auditd
4026532236 mnt         1   789 root   /usr/sbin/ModemManager
4026532237 mnt         1   802 chrony /usr/sbin/chronyd
4026532238 mnt         1   837 root   /usr/sbin/NetworkManager --no-daemon
4026532239 mnt         3  2849 root   nginx: master process nginx -g daemon off;
4026532240 uts         3  2849 root   nginx: master process nginx -g daemon off;
4026532241 ipc         3  2849 root   nginx: master process nginx -g daemon off;
4026532242 pid         3  2849 root   nginx: master process nginx -g daemon off;
4026532244 net         3  2849 root   nginx: master process nginx -g daemon off;

[root@g7 ~]# ll /proc/2849/ns/
total 0
lrwxrwxrwx. 1 root root 0 Aug  3 23:18 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 root root 0 Aug  3 23:18 ipc -> 'ipc:[4026532241]'
lrwxrwxrwx. 1 root root 0 Aug  3 23:18 mnt -> 'mnt:[4026532239]'
lrwxrwxrwx. 1 root root 0 Aug  3 23:18 net -> 'net:[4026532244]'
lrwxrwxrwx. 1 root root 0 Aug  3 23:18 pid -> 'pid:[4026532242]'
lrwxrwxrwx. 1 root root 0 Aug  3 23:22 pid_for_children -> 'pid:[4026532242]'
lrwxrwxrwx. 1 root root 0 Aug  3 23:22 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0 Aug  3 23:22 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0 Aug  3 23:18 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 root root 0 Aug  3 23:18 uts -> 'uts:[4026532240]'
~~~

