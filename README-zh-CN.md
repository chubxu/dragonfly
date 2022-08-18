<p align="center">
  <a href="https://dragonflydb.io">
    <img  src="/.github/images/logo-full.svg"
      width="284" border="0" alt="Dragonfly">
  </a>
</p>

[![ci-tests](https://github.com/dragonflydb/dragonfly/actions/workflows/ci.yml/badge.svg)](https://github.com/dragonflydb/dragonfly/actions/workflows/ci.yml) [![Twitter URL](https://img.shields.io/twitter/follow/dragonflydbio?style=social)](https://twitter.com/dragonflydbio)


[Quick Start](https://github.com/dragonflydb/dragonfly/tree/main/docs/quick-start) | [Discord Chat](https://discord.gg/HsPjXGVH85) | [GitHub Discussions](https://github.com/dragonflydb/dragonfly/discussions) | [GitHub Issues](https://github.com/dragonflydb/dragonfly/issues) | [Contributing](https://github.com/dragonflydb/dragonfly/blob/main/CONTRIBUTING.md)

# 有可能，这是宇宙中最快的内存数据库

Dragonfly是一款现代内存数据库，与 Redis 和 Memcached API 完全兼容。Dragonfly 在多线程、无共享架构之上实现了**新的算法与数据结构**。因此，Dragonfly相较于Redis 拥有25倍性能的提升，并且单个Dragonfly实例能够支持数百万 QPS。

Dragonfly的核心特性使其成为高效、高性能并易于使用的Redis替代品。

# 基准测试

<img src="http://assets.dragonflydb.io/repo-assets/aws-throughput.svg" width="80%" border="0"/>

在亚马逊云上的c6gn.16xlarge实例上（64CPU、128G内存、100Gbps网络带宽、38Gbps吞吐量），Dragonfly超过3.8M的QPS，相较于Redis，吞吐量增加了25倍。

Dragonfly 峰值吞吐量的P99在不同机器上的值如下表所示：

| op  |r6g | c6gn | c7g |
|-----|-----|------|----|
| set |0.8ms  | 1ms | 1ms   |
| get | 0.9ms | 0.9ms |0.8ms |
|setex| 0.9ms | 1.1ms | 1.3ms

*ps：所有基准测试均使用 memtier_benchmark执行，并且标明了每个服务类型、实例类型和相应的线程数。memtier 在单独的 c6gn.16xlarge 机器上运行。对于setex的基准测试，我们将超时时间设置为500，因此，所有设置的key都能够在基准测试中存活。*

在pipeline模式下（--pipeline=30），Dragonfly的SET操作达到10M的QPS，GET操作达到15M的QPS。

## Memcached / Dragonfly

在亚马逊的c6gn.16xlarge 实例上，我们还对比了memcached和Dragonfly。正如下面表格所示，Dragonfly在读写负载方面，吞吐量高于memcached，并且延迟相当。对于写负载，因为memcached的写路径竞争问题，Dragonfly还有更好的延迟性能。

### SET benchmark

|Server|QPS(thousands qps)|latency 99%|99.9%|
|-|-|-|-|
|Dragonfly|🟩 3844|🟩 0.9ms|🟩 2.4ms|
|Memcached|806|1.6ms|3.2ms|


### GET benchmark

|Server|QPS(thousands qps)|latency 99%|99.9%|
|-|-|-|-|
|Dragonfly|🟩 3717|1ms|2.4ms|
|Memcached|2100|🟩 0.34ms|🟩 0.6ms|

Memcached 在get基准测试中表现出较低的延迟，但吞吐量也较低。

## 内存效率

在接下来的测试中，我们分别使用Dragonfly 和 Redis填充了大约5GB的数据到内存中（使用`debug populate 5000000 key 1024`命令）。然后我们使用memtier发送更新请求，同时利bgsave命令同步数据。下图清晰的展示了两台服务器在内存效率方面的表现。

<img src="http://assets.dragonflydb.io/repo-assets/bgsave-memusage.svg" width="70%" border="0"/>

在空闲状态下，Dragonfly 的内存效率比 Redis 高 30%。并且在数据同步期间，Dragonfly也几乎看不见任何可见的内存增长。于此同时，Redis在峰值情况下，内存大约增至3倍。在同步速度方面，Dragonfly同样更为快速，仅仅用了几十秒。对于内存效率对比的更多信息可以参考[dashtable doc](https://github.com/chubxu/dragonfly/blob/main/docs/dashtable.md)。

# 服务器运行

Dragonfly运行在Linux上，它使用了Linux相对较新的io-uring API进行I/O操作。因此，他需要Linux5.10+。Debian/Bullseye, Ubuntu 20.04.4+来满足这些需求。

## docker运行

```Bash
docker run --network=host --ulimit memlock=-1 docker.dragonflydb.io/dragonflydb/dragonfly

redis-cli PING  # redis-cli can be installed with "apt install -y redis-tools"
```

你需要添加`*`--ulimit memlock=-1`*`命令，因为Linux 发行版将容器的默认 memlock 限制配置为 64m，而 Dragonfly 需要更多。

## Releases版本

我们维护了x86和arm64架构的二进制版本。您需要安装 libunwind8 库才能运行二进制文件。

## 从源代码构建

在Ubuntu 20.04+的机器上，你需要安装如下依赖进行构建。

```Bash
git clone --recursive https://github.com/dragonflydb/dragonfly && cd dragonfly

# to install dependencies
sudo apt install ninja-build libunwind-dev libboost-fiber-dev libssl-dev \
     autoconf-archive libtool cmake g++

# Configure the build
./helio/blaze.sh -release

# Build
cd build-opt && ninja dragonfly

# Run
./dragonfly --alsologtostderr
```

# 配置

在适用的情况下，Dragonfly 支持常见的 redis 参数。例如，你可以运行如下脚本：`dragonfly --requirepass=foo --bind localhost`。

Dragonfly 目前支持以下 Redis 特定参数：

- `port`
- `bind`
- `requirepass`
- `maxmemory`
- `dir`：默认情况下，dragonfly的docker镜像使用`/data`目录作为快照目录。你可以使用 `-v` docker选项来映射你的主机目录。
- `dbfilename`

此外，Dragonfly还有自己的特定的参数选项：

- `memcache_port`：