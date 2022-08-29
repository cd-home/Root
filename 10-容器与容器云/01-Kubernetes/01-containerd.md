[TOC]

### containerd

》容器运行时

- [ ] 管理容器的生命周期(从创建容器到销毁容器)
- [ ] 拉取/推送镜像
- [ ] 存储管理（管理镜像及容器数据的存储）
- [ ] 调用 runc 运行容器(与 runc 等容器运行时交互)
- [ ] 管理容器网络接口及网络

#### Arch

TODO

#### install

Arch Centos 8 arm64

~~~bash
$ wget https://github.com/containerd/containerd/releases/download/v1.6.6/cri-containerd-cni-1.6.6-linux-arm64.tar.gz
~~~

如果网络很差,可使用加速

~~~bash
$ wget https://download.fastgit.org/containerd/containerd/releases/download/v1.6.6/cri-containerd-cni-1.6.6-linux-arm64.tar.gz
~~~

查看压缩包, 方便知道解压的路径

~~~bash
$ tar -tf cri-containerd-cni-1.6.6-linux-arm64.tar.gz
~~~

~~~bash
etc/
etc/cni/
etc/cni/net.d/
etc/cni/net.d/10-containerd-net.conflist
etc/crictl.yaml
etc/systemd/
etc/systemd/system/
etc/systemd/system/containerd.service
usr/
usr/local/
usr/local/sbin/
usr/local/sbin/runc
usr/local/bin/
usr/local/bin/containerd-shim
usr/local/bin/containerd
usr/local/bin/critest
usr/local/bin/crictl
usr/local/bin/containerd-shim-runc-v1
usr/local/bin/containerd-stress
usr/local/bin/containerd-shim-runc-v2
usr/local/bin/ctd-decoder
usr/local/bin/ctr
opt/
opt/cni/
opt/cni/bin/
opt/cni/bin/static
opt/cni/bin/vlan
opt/cni/bin/bandwidth
opt/cni/bin/portmap
opt/cni/bin/ipvlan
opt/cni/bin/firewall
opt/cni/bin/macvlan
opt/cni/bin/dhcp
opt/cni/bin/ptp
opt/cni/bin/sbr
opt/cni/bin/vrf
opt/cni/bin/loopback
opt/cni/bin/host-device
opt/cni/bin/tuning
opt/cni/bin/bridge
opt/cni/bin/host-local
opt/containerd/
opt/containerd/cluster/
opt/containerd/cluster/gce/
opt/containerd/cluster/gce/cloud-init/
opt/containerd/cluster/gce/cloud-init/node.yaml
opt/containerd/cluster/gce/cloud-init/master.yaml
opt/containerd/cluster/gce/configure.sh
opt/containerd/cluster/gce/cni.template
opt/containerd/cluster/gce/env
opt/containerd/cluster/version
~~~

即解压到 / 即可

~~~bash
$ tar -C / -zxvf cri-containerd-cni-1.6.6-linux-arm64.tar.gz
~~~

配置文件

~~~bash
$ containerd cinfig default > /etc/containerd/config.toml
~~~

解压的文件包含了systemd配置, 所以可以通过systemctl管理

~~~bash
$ systemctl start containerd
~~~

注意一下cni的位置

#### ctr

》containerd CLI

~~~bash
NAME:
   ctr -
        __
  _____/ /______
 / ___/ __/ ___/
/ /__/ /_/ /
\___/\__/_/

containerd CLI


USAGE:
   ctr [global options] command [command options] [arguments...]

VERSION:
   v1.6.6

DESCRIPTION:

ctr is an unsupported debug and administrative client for interacting
with the containerd daemon. Because it is unsupported, the commands,
options, and operations are not guaranteed to be backward compatible or
stable from release to release of the containerd project.

COMMANDS:
   plugins, plugin            provides information about containerd plugins
   version                    print the client and server versions
   containers, c, container   manage containers
   content                    manage content
   events, event              display containerd events
   images, image, i           manage images
   leases                     manage leases
   namespaces, namespace, ns  manage namespaces
   pprof                      provide golang pprof outputs for containerd
   run                        run a container
   snapshots, snapshot        manage snapshots
   tasks, t, task             manage tasks
   install                    install a new package
   oci                        OCI tools
   shim                       interact with a shim directly
   help, h                    Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug                      enable debug output in logs
   --address value, -a value    address for containerd's GRPC server (default: "/run/containerd/containerd.sock") [$CONTAINERD_ADDRESS]
   --timeout value              total timeout for ctr commands (default: 0s)
   --connect-timeout value      timeout for connecting to containerd (default: 0s)
   --namespace value, -n value  namespace to use with commands (default: "default") [$CONTAINERD_NAMESPACE]
   --help, -h                   show help
   --version, -v                print the version
~~~

#### nerdctl

contaiNERD CTL - Docker-compatible CLI for containerd

如果已有containerd情况下,可以选择只安装nerdctl; 否则请安装full版本

~~~bash
$ wget https://github.com/containerd/nerdctl/releases/download/v0.22.0/nerdctl-0.22.0-linux-arm64.tar.gz

$ https://github.com/containerd/nerdctl/releases/download/v0.22.0/nerdctl-full-0.22.0-linux-arm64.tar.gz
~~~

~~~bash
$ nerdctl version
~~~

注意 nerdctl读取cni plugins binary directory [$CNI_PATH] (default "/usr/libexec/cni")的位置

可能由于cni配置与插件版本不兼容问题

~~~bash
error during container init: plugin="xxxx" failed (add): imcompatible CNI versions;
condfig is "1.0.0", plugin supports ["0.4.0"]
~~~

 需要更新cni(注意更新的位置)

~~~bash
$ wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-arm64-v1.1.1.tgz

$ tar -C /usr/libexec/cni -zxvf cni-plugins-linux-arm64-v1.1.1.tgz
~~~

Example

~~~bash
$ nerdctl pull redis
$ nerdctl create -p 6379:6379 --name redis-server redis:latest
$ nerdctl start redis-server

$ nerdctl pull nginx
$ nerdctl run -d -p 80:80 --name ng nginx:latest
~~~

Other Just Like Docker-cli

~~~bash
$ nerdctl images
$ nerdctl ps -a
$ nerdctl logs -f 
$ nerdctl rmi
$ nerdctl rm
~~~

#### buildkit

首先要安装 buildctl buildkitd, 在buildkit下

~~~bash
$ wget https://github.com/moby/buildkit/releases/download/v0.10.3/buildkit-v0.10.3.linux-arm64.tar.gz

$ tar -C /usr/local/ -zxf buildkit-v0.10.3.linux-arm64.tar.gz
~~~

/usr/lib/systemd/system/buildkit.service

~~~bash
[Unit]
Description=BuildKit
Requires=buildkit.socket
After=buildkit.socket
Documentation=https://github.com/moby/buildkit

[Service]
Type=notify
ExecStart=/usr/local/bin/buildkitd --oci-worker=false --containerd-worker=true

[Install]
WantedBy=multi-user.target
~~~

/usr/lib/systemd/system/buildkit.socket

~~~bash
[Unit]
Description=BuildKit
Documentation=https://github.com/moby/buildkit

[Socket]
ListenStream=%t/buildkit/buildkitd.sock
SocketMode=0660

[Install]
WantedBy=sockets.target
~~~

~~~bash
$ systemctl daemon-reload 
$ systemctl start buildkit
~~~

Example

~~~bash
$ buildctl build \
	--output type=image,name=ng  \
	--frontend=dockerfile.v0 \
	--local context=. \
	--local dockerfile=.
~~~

说明: buildctl 的镜像在 ctr的buildkit空间下

~~~bash
$ ctr -n buildkit i ls
~~~

