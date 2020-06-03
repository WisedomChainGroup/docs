# 二、Wisdom客户端
## 2.1 客户端种类
&#160;&#160;&#160;&#160;&#160;&#160;全节点和矿工节点
## 2.2 核心节点部署
### 2.2.1 配置网络防火墙
&#160;&#160;&#160;&#160;&#160;&#160;WDC P2P和RPC默认端口均为19585，在docker容器端口映射时，可根据需求修改。请根据需求修改网络防火墙设置，决定是否开放该端口。

### 2.2.2服务器硬件
&#160;&#160;&#160;&#160;&#160;&#160;磁盘空间建议 500GB，内存16GB，CPU 8核。

### 2.2.3安装docker、docker-compose

#### 	2.2.3.1 Ubuntu

```
apt install -y docker-compose
```

#### 	2.2.3.2 CentOS

```
yum install -y docker

sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname-s)-$(uname-m)" -o/usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

### 2.2.4 Docker镜像地址
&#160;&#160;&#160;&#160;&#160;&#160;		节点程序镜像：wisdomchain/wdc_core

&#160;&#160;&#160;&#160;&#160;&#160;
数据库镜像：wisdomchain/wdc_pgsql

### 2.2.5 wdc.yml文件示例
<font color=red>* 以下内容为示例性内容，实际部署时，卷映射、环境变量、端口映射等根据实际情况，需要调整。</font>
```yml
version: '3.1'

services:

    wdc_pgsql:
        image: wisdomchain/wdc_pgsql
        restart: always
        container_name: wdc_pgsql
        privileged: true
        volumes:
            -/opt/wdc_pgsql:/var/lib/postgresql/data # pgsql 数据目录
        ports:
            -127.0.0.1:5433:5432
        environment:
            POSTGRES_USER: wdcadmin
            POSTGRES_PASSWORD: PqR_w9hk6Au-jq5ElsFcEjq!wvULrYXeF3*oDKp5i@A/D5m03VaB1M/hyKY
            WDC_POSTGRES_USER: replica
            WDC_POSTGRES_PASSWORD: replica

    wdc_core:
        image: wisdomchain/wdc_core
        restart: always
        container_name: wdc_core
        privileged: true
        volumes:
            -/opt/wdc_logs:/logs #程序日志目录
            -/opt/wdc_leveldb:/leveldb
            -./entry_point.sh:/entry_point.sh
            -/opt/ipc:/root/ipc
            -./libs:/libs
            -/opt/fast-sync:/fast-sync
        entrypoint: /usr/bin/env bash /entry_point.sh -d wdc_pgsql:5432 -c '/usr/bin/env bash /run_wdc_core.sh'
        ports:
            -19585:19585
            -9585:9585
        environment:
            LOGGING_CONFIG: 'https://wisdom-config.oss-cn-hangzhou.aliyuncs.com/public-chain/logback.xml' #如发现LOGGING_CONFIG相关路径报错，可以去除此项
            DATA_SOURCE_URL: 'jdbc:postgresql://wdc_pgsql:5432/postgres'
            DB_USERNAME: 'replica'
            DB_PASSWORD: 'replica'
            WDC_MINER_COINBASE: 'WX1********XN1T21573hYata'
            P2P_MODE: 'grpc'
            P2P_ADDRESS: 'wisdom://192.**.***.156:9585'
            BOOTSTRAPS: 'wisdom://47.74.183.249:9585,wisdom://47.74.216.251:9585,wisdom://47.96.67.155:9585,wisdom://47.74.86.106:9585'
            MAX_BLOCKS_PER_TRANSFER: '256'
            ENABLE_DISCOVERY: 'true'
            ENABLE_MINING: 'true'
            FAST_SYNC_DIRECTORY: '/fast-sync'
            DATABASE_DIRECTORY: '/leveldb'
```

### 2.2.6 卷映射（volumes）
&#160;&#160;&#160;&#160;&#160;&#160;可根据需要，映射到不同的目录。

&#160;&#160;&#160;&#160;&#160;&#160;其中，wdc_pgsql volumes映射的是PostgreSql数据库数据目录，docker容器删除后，该目录不会自动删除，节点数据仍然保留。如果想要全新启动WDC Core，请备份该目录后，删除或清空该目录。

&#160;&#160;&#160;&#160;&#160;&#160;wdc_core volumes映射的是WDC Core节点程序日志目录。

&#160;&#160;&#160;&#160;&#160;&#160;容器内部的一些文件，也可以根据需要映射到宿主机目录。比如wdc_core容器内部的/fast-sync，可以映射为： /opt/fast-sync:/fast-sync。

### 2.2.7 网络端口映射（ports）
&#160;&#160;&#160;&#160;&#160;&#160;wdc_pgsql的端口映射，为了保障安全，建议映射到IP地址127.0.0.1，只允许本机访问。如果不想通过外部客户端访问数据库，也可以去掉该端口映射。

&#160;&#160;&#160;&#160;&#160;&#160;wdc_pgsql和wdc_core的外部端口号，可根据需求修改。

### 2.2.8 环境变量（environment）
&#160;&#160;&#160;&#160;&#160;&#160;数据库用户名密码可以自定义，但要保证WDC_POSTGRES_USER与DB_USERNAME保持一致，WDC_POSTGRES_PASSWORD与DB_PASSWORD一致

&#160;&#160;&#160;&#160;&#160;&#160;ENABLE_MINING 表示是否启动挖矿

&#160;&#160;&#160;&#160;&#160;&#160;WDC_MINER_COINBASE 为挖矿coinbase地址，必须设置，否则节点无法启动。生成地址的方法参见下一节“矿工地址生成”

&#160;&#160;&#160;&#160;&#160;&#160;DATA_SOURCE_URL 的值，利用docker容器互联，不必修改。如果需要修改，需确保URL中的主机名为pgsql容器名，端口与pgsql容器内部的数据库端口相同

&#160;&#160;&#160;&#160;&#160;&#160;BOOTSTRAPS：种子节点列表，英文逗号分隔

&#160;&#160;&#160;&#160;&#160;&#160;ENABLE_DISCOVERY：是否允许节点发现，true/false

&#160;&#160;&#160;&#160;&#160;&#160;P2P_ADDRESS：自己节点ip、port

&#160;&#160;&#160;&#160;&#160;&#160;ports：9585是p2p端口，19585是rpc端口

&#160;&#160;&#160;&#160;&#160;&#160;MAX_BLOCKS_PER_TRANSFER：最大同步区块数（官方建议默认256）

&#160;&#160;&#160;&#160;&#160;&#160;FAST_SYNC_DIRECTORY: '/fast-sync'：快照文件路径（此路径为容器内的路径，需要与wdc.yml中配置volumes下的保持一致）

&#160;&#160;&#160;&#160;&#160;&#160;DATABASE_DIRECTORY: '/leveldb'：leveldb存储路径（此路径为容器内的路径，需要与wdc.yml中配置volumes下的保持一致）

### 2.2.9 矿工地址生成
&#160;&#160;&#160;&#160;&#160;&#160;使用手机APP生成地址。APP可在官网下载： https://www.wisdchain.com/

&#160;&#160;&#160;&#160;&#160;&#160;目前下载页面为： https://www.wisdchain.com/user/application_1

### 2.2.10 准备entry_point.sh
&#160;&#160;&#160;&#160;&#160;&#160;wdc.yml文件中，wdc_core服务的入口脚本内容：

```shell
#!/bin/bash
#set -x
#******************************************************************************
# @file    : entrypoint.sh
# @author  : wangyubin
# @date    : 2018-08- 1 10:18:43
#
# @brief   : entry point for manage service start order
# history  : init
#******************************************************************************

: ${SLEEP_SECOND:=2}

wait_for() {
    echo Waiting for $1 to listen on $2...
    while ! nc -z $1 $2; do echo waiting...; sleep $SLEEP_SECOND; done
}

declare DEPENDS
declare CMD

while getopts "d:c:" arg
do
    case $arg in
        d)
            DEPENDS=$OPTARG
            ;;
        c)
            CMD=$OPTARG
            ;;
        ?)
            echo "unkonw argument"
            exit 1
            ;;
    esac
done

for var in ${DEPENDS//,/ }
do
    host=${var%:*}
    port=${var#*:}
    wait_for $host $port
done

eval $CMD

```

### 2.2.11 启动docker镜像
&#160;&#160;&#160;&#160;&#160;&#160;更新镜像：
```
docker pull wisdomchain/wdc_core
	
docker pull wisdomchain/wdc_pgsql
	
docker-compose -f wdc.yml up -d
```

### 2.2.12 查看日志
&#160;&#160;&#160;&#160;&#160;&#160;命令 docker logs -f <CONTAINER ID> 查看节点程序控制台输出/opt/wdc_logs目录为节点程序日志文件目录，如果再YML文件映射到了其他目录，请到相应目录查看。

## 2.3 节点启动流程

```
//停止并删除容器，确保节点没有启动
docker-compose -f wdc.yml down

//获得最新版本镜像
docker pull wisdomchain/wdc_core

//修改wdc.yml，如果有必要的话

//启动新版镜像
docker-compose -f wdc.yml up -d
```

## 2.4 参数配置

|参数项| 参数说明|作用
|---|---|---
|<div style="width:155pt">DATA_SOURCE_URL</div> | <div style="width:70pt">数据库配置</div>|默认为jdbc:postgresql://wdc_pgsql:5432/postgres，利用docker容器互联，不必修改。如果需要修改，需确保URL中的主机名为pgsql容器名，端口与pgsql容器内部的数据库端口相同
|DB_USERNAME|数据库名称|默认为replica
|DB_PASSWORD|数据库密码|默认为replica
|ENABLE_MINING|是否开启挖矿|true是开启，false是关闭
|WDC_MINER_COINBASE|矿工地址|WDC生成的地址
|P2P_MODE|通讯模式|grpc
|P2P_ADDRESS|节点地址|模式为wisdom://192.168.1.156:9585
|BOOTSTRAPS|种子节点|默认为wisdom://47.74.183.249:9585,wdom://47.74.216.251:9585,wisdom://47.96.67.155:9585,wisdom://47.74.86.106:9585
|MAX_BLOCKS_PER_TRANSFER|最大同步区块数|默认256
|ENABLE_DISCOVERY|是否开启节点发现|true是开启，false是关闭
|FAST_SYNC_DIRECTORY|快照文件路径（此路径为容器内的路径，需要与wdc.yml中配置volumes下的保持一致）|/fast-sync
|DATABASE_DIRECTORY|leveldb存储路径（此路径为容器内的路径，需要与wdc.yml中配置volumes下的保持一致）|/leveldb


## 2.5 控制台工具
详见：
https://github.com/WisedomChainGroup/java-wisdomcore/tree/master/wisdom-core/src/main/java/org/wisdom/ipc

