# fnos
install a fnos in docker

# 原理
基于项目https://github.com/qemus/qemu，在docker容器里通过qemu运行fnos。

# docker-compose.yml

services:
  fnos:
    image: qemux/qemu
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

environment:
BOOT: “https://iso.liveupdate.fnnas.com/x86_64/trim/fnos-0.9.11-946.iso“ #飞牛os最新安装镜像地址
RAM_SIZE: “2G” #qemu设定的内存和核
CPU_CORES: “4”
DISK_SIZE: “16G” #飞牛系统盘大小
DISK2_SIZE: “200G” #数据盘大小
networks：设定飞牛系统本地IP

# 创建macvlan网络
在缺省网络模式下，qemu里运行的fnos无法通过IP从外部访问。
运行：
docker network create -d macvlan \
   --subnet=192.168.0.0/24 \
   --gateway=192.168.0.1 \
   --ip-range=192.168.0.0/28 \
   -o parent=ens133 vlan
   
ens133是本地网卡名，可通过ifconfig命令查看。

# 启动容器
docker compose pull
docker compose up -d

拉取镜像可能需要梯子。
