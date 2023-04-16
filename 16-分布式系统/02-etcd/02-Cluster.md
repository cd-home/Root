### Cluster

#### Install

ssl

~~~
go install github.com/cloudflare/cfssl/cmd/...@latest
~~~

0. 初始化

~~~
cfssl print-defaults config > ca-config.json
cfssl print-defaults csr > ca-csr.json
~~~

1. 修改ca-config.json

~~~
{
    "signing": {
        "default": {
            "expiry": "438000h"
        },
        "profiles": {
            "server": {
                "expiry": "438000h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "438000h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "etcd": {
                "expiry": "438000h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
~~~

2. 修改ca-csr.json

~~~
{
    "CN": "etcd",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "GZ",
            "L": "GZ",
	    	"O": "etcd",
	    	"OU": "System"
        }
    ]
}
~~~

3. ca证书

~~~
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
~~~

4. vim etcd-csr.json

~~~
{
    "CN": "etcd",
    "hosts": [
      "127.0.0.1",
      "10.211.55.81",
      "10.211.55.82",
      "10.211.55.83",
      "localhost"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "GZ",
            "L": "GZ",
            "O": "etcd",
            "OU": "System"
        }
    ]
}
~~~

5. gen

~~~
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd etcd-csr.json | cfssljson -bare etcd
~~~

6. systemd 每个节点

~~~
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.targe

[Service]
Type=notify
WorkingDirectory=/opt/etcd
ExecStart=/usr/local/etcd --config-file=/etc/etcd/etcd.yaml
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
~~~

7. 测试

~~~
[root@k8s-hm1 etcd]# 
etcdctl --endpoints "https://10.211.55.81:2379,https://10.211.55.82:2379,https://10.211.55.83:2379" \
--cacert=/etc/etcd/ssl/ca.pem --cert=/etc/etcd/ssl/etcd.pem --key=/etc/etcd/ssl/etcd-key.pem endpoint \ health
~~~

etcd.yaml  每个节点

~~~yaml
# This is the configuration file for the etcd server.

# Human-readable name for this member.
name: 'etcd-hm'

# Path to the data directory.
data-dir: /var/lib/etcd/default.etcd

# Path to the dedicated wal directory.
wal-dir:

# Number of committed transactions to trigger a snapshot to disk.
snapshot-count: 10000

# Time (in milliseconds) of a heartbeat interval.
heartbeat-interval: 100

# Time (in milliseconds) for an election to timeout.
election-timeout: 1000

# Raise alarms when backend size exceeds the given quota. 0 means use the
# default quota.
quota-backend-bytes: 0

# List of comma separated URLs to listen on for peer traffic.
listen-peer-urls: http://0.0.0.0:2380

# List of comma separated URLs to listen on for client traffic.
listen-client-urls: http://0.0.0.0:2379

# Maximum number of snapshot files to retain (0 is unlimited).
max-snapshots: 5

# Maximum number of wal files to retain (0 is unlimited).
max-wals: 5

# Comma-separated white list of origins for CORS (cross-origin resource sharing).
cors:

# List of this member's peer URLs to advertise to the rest of the cluster.
# The URLs needed to be a comma-separated list.
initial-advertise-peer-urls: https://10.211.55.61:2380 # 每个节点修改 61 62 63

# List of this member's client URLs to advertise to the public.
# The URLs needed to be a comma-separated list.
advertise-client-urls: https://10.211.55.61:2379 # 每个节点修改 61 62 63

# Discovery URL used to bootstrap the cluster.
discovery:

# Valid values include 'exit', 'proxy'
discovery-fallback: 'proxy'

# HTTP proxy to use for traffic to discovery service.
discovery-proxy:

# DNS domain used to bootstrap initial cluster.
discovery-srv:

# Initial cluster configuration for bootstrapping.
initial-cluster: "etcd-hm1=https://10.211.55.61:2380,etcd-hm2=https://10.211.55.62:2380,etcd-hm3=https://10.211.55.63:2380"

# Initial cluster token for the etcd cluster during bootstrap.
initial-cluster-token: 'etcd-cluster'

# Initial cluster state ('new' or 'existing').
initial-cluster-state: 'new'

# Reject reconfiguration requests that would cause quorum loss.
strict-reconfig-check: false

# Enable runtime profiling data via HTTP server
enable-pprof: true

# Valid values include 'on', 'readonly', 'off'
proxy: 'off'

# Time (in milliseconds) an endpoint will be held in a failed state.
proxy-failure-wait: 5000

# Time (in milliseconds) of the endpoints refresh interval.
proxy-refresh-interval: 30000

# Time (in milliseconds) for a dial to timeout.
proxy-dial-timeout: 1000

# Time (in milliseconds) for a write to timeout.
proxy-write-timeout: 5000

# Time (in milliseconds) for a read to timeout.
proxy-read-timeout: 0

client-transport-security:
  # Path to the client server TLS cert file.
  cert-file:

  # Path to the client server TLS key file.
  key-file:

  # Enable client cert authentication.
  client-cert-auth: false

  # Path to the client server TLS trusted CA cert file.
  trusted-ca-file:

  # Client TLS using generated certificates
  auto-tls: false

peer-transport-security:
  # Path to the peer server TLS cert file.
  cert-file:

  # Path to the peer server TLS key file.
  key-file:

  # Enable peer client cert authentication.
  client-cert-auth: false

  # Path to the peer server TLS trusted CA cert file.
  trusted-ca-file:

  # Peer TLS using generated certificates.
  auto-tls: false

# The validity period of the self-signed certificate, the unit is year.
self-signed-cert-validity: 1

# Enable debug-level logging for etcd.
log-level: debug

logger: zap

# Specify 'stdout' or 'stderr' to skip journald logging even when running under systemd.
log-outputs: [stderr]

# Force to create a new one member cluster.
force-new-cluster: false

auto-compaction-mode: periodic
auto-compaction-retention: "1"

# Limit etcd to a specific set of tls cipher suites
cipher-suites: [
  TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
  TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
]
~~~

