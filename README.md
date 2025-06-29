# fnos
install a fnos in docker

# 原理
基于项目[https://github.com/qemus/qemu](https://github.com/qemus/qemu)，在docker容器里通过qemu运行fnos。

# 创建docker-compose.yml，内容如下
```
services:
  fnos:
    image: ghcr.io/qemus/qemu:7.12
    container_name: fnos
    environment:
      BOOT: "https://iso.liveupdate.fnnas.com/x86_64/trim/fnos-0.9.11-946.iso"
      RAM_SIZE: "2G"
      CPU_CORES: "4"
      DISK_SIZE: "16G"
      DISK2_SIZE: "200G"
    devices:
      - /dev/kvm
      - /dev/net/tun
    cap_add:
      - NET_ADMIN
    ports:
      - 8006:8006
    volumes:
      - /dir1:/storage
      - /dir2:/storage2
    restart: unless-stopped
    stop_grace_period: 2m
    networks:
      vlan:
        ipv4_address: 192.168.0.10
networks:
    vlan:
       external: true
```

environment:

BOOT: “https://iso.liveupdate.fnnas.com/x86_64/trim/fnos-0.9.11-946.iso“ #飞牛os最新安装镜像地址

RAM_SIZE: “2G” #qemu设定的内存和核

CPU_CORES: “4”

DISK_SIZE: “16G” #飞牛系统盘大小，对应容器里的/storiage，系统里的目录/dir1

DISK2_SIZE: “200G” #数据盘大小，对应容器里的/storiage2，系统里的目录/dir2

networks：设定飞牛系统本地IP

# 创建macvlan网络
在缺省网络模式下，qemu里运行的fnos无法通过IP从外部访问。假设内网是192.168.0.*
运行：
```
docker network create -d macvlan \
   --subnet=192.168.0.0/24 \
   --gateway=192.168.0.1 \
   --ip-range=192.168.0.0/28 \
   -o parent=ens133 vlan
```   
ens133是本地网卡名，可通过ifconfig命令查看。

# 启动容器
```
docker compose pull
docker compose up -d
```
拉取镜像可能需要梯子。
修改docker-compose.yml后需docker compose down，再重新启动容器。

# 安装fnos
浏览器访问192.168.0.10:8006，按提示完成fnos安装，
安装完成后，打开192.168.0.10:5666体验fnos。

