[TOC]

### Etcd

#### Install

~~~bash
$ wget https://github.com/etcd-io/etcd/releases/download/v3.5.0/etcd-v3.5.0-linux-arm64.tar.gz
~~~

systemd

~~~bash
[Unit]
Description=Etcd Server

[Service]
Type=notify
#EnvironmentFile=/etc/etcd/etcd.conf
ExecStart=/usr/local/bin/etcd \
	--listen-client-urls 'http://0.0.0.0:2379' \
    --advertise-client-urls 'http://0.0.0.0:2379'
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
~~~

#### etcdctl

健康检查

~~~bash
$ etcdctl endpoint health
~~~

##### put

~~~bash
# 新建/修改
$ etcdctl put k v
~~~

##### get

~~~bash
$ etcdctl get k
# 获取全部
$ etcdctl get --from-key ''
$ etcdctl get --prefix /ns/
~~~

##### del

~~~bash
$ etcdctl del k
$ etcdctl del --from-key ''
$ etcdctl del --prefix /ns/
~~~

##### watch

~~~bash
$ etcdctl watch k
$ etcdctl watch --prefix k
# 监控多个值
$ etcdctl watch -i
watch name
watch age
~~~

##### lease

~~~bash
# 设置租约
$ etcdctl lease grant 60
lease 694d831d9e27c415 granted with TTL(60s)
# 一个租约可以绑定多个key
$ $ etcdctl --lease=694d831d9e27c415 put k v

# 撤销租约
$ etcdctl lease revoke 694d831d9e27c415

# 续租
$ etcdctl lease keep-alive 694d831d9e27c415

# 查看租约情况
$ etcdctl lease timetolive --keys 694d831d9e27c415
~~~