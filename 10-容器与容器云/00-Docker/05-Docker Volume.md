[TOC]

### Volume

》推荐使用卷来持久化数据; 存储驱动Overlay2; 卷和容器是非耦合关系

|                 Comparisons                  |       Named Volumes       |          Bind Mounts          |
| :------------------------------------------: | :-----------------------: | :---------------------------: |
|                Host Location                 |      Docker chooses       |          You control          |
|          Mount Example (using `-v`)          | my-volume:/usr/local/data | /path/to/data:/usr/local/data |
| Populates new volume with container contents |            Yes            |              No               |
|           Supports Volume Drivers            |            Yes            |              No               |

Named Volumes 即是Docker管理

~~~bash
/var/lib/docker/volume
~~~

~~~bash
$ docker volume ls
$ docker volume create xVolume
$ docker volume rm|prune xVolume
~~~

Bind Mounts 是用户管理的(自定义的目录/文件), 需要绝对路径的.

#### Example

##### --mount

~~~bash
$ docker run -it --mount source=bizvol,target=/vol  demo /bin/bash
~~~

不指定type默认创建volume, 不存在该卷就会自动创建;  

~~~bash
➜  ~ docker volume ls
DRIVER    VOLUME NAME
local     bizvol
➜  ~ docker volume inspect bizvol
[
    {
        "CreatedAt": "2022-09-24T04:19:44Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/bizvol/_data",
        "Name": "bizvol",
        "Options": null,
        "Scope": "local"
    }
]
~~~

如果指定type=bind 就是 bind mount, 即是source需要是绝对路径并且必须存在

~~~bash
$ docker run -it --mount type=bind,source=$(PWD)/bizvol3,target=/vol  demo /bin/bash
~~~

##### --volume

-v 挂载卷, 不存在就会创建

~~~bash
$ docker run -it -v bizvol3:/vol demo /bin/bash
~~~

bind mount

~~~bash
$ docker run -it -v $(PWD)/bizvol3:/vol demo /bin/bash
~~~

注意

- [ ] 路径必须使用绝对路径
- [ ] -v如果src不存在则会创建成目录; 所以如果你要挂载一个文件, 那么它必须已经存在, 否则就会被挂载为目录.

