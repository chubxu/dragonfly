<p align="center">
  <a href="https://dragonflydb.io">
    <img  src="/.github/images/logo-full.svg"
      width="284" border="0" alt="Dragonfly">
  </a>
</p>

[![ci-tests](https://github.com/dragonflydb/dragonfly/actions/workflows/ci.yml/badge.svg)](https://github.com/dragonflydb/dragonfly/actions/workflows/ci.yml) [![Twitter URL](https://img.shields.io/twitter/follow/dragonflydbio?style=social)](https://twitter.com/dragonflydbio)


[Quick Start](https://github.com/dragonflydb/dragonfly/tree/main/docs/quick-start) | [Discord Chat](https://discord.gg/HsPjXGVH85) | [GitHub Discussions](https://github.com/dragonflydb/dragonfly/discussions) | [GitHub Issues](https://github.com/dragonflydb/dragonfly/issues) | [Contributing](https://github.com/dragonflydb/dragonfly/blob/main/CONTRIBUTING.md)

### 有可能，这是宇宙中最快的内存数据库

Dragonfly是一款现代内存数据库，与 Redis 和 Memcached API 完全兼容。Dragonfly 在多线程、无共享架构之上实现了**新的算法与数据结构**。因此，Dragonfly相较于Redis 拥有25倍性能的提升，并且单个Dragonfly实例能够支持数百万 QPS。

Dragonfly的核心特性使其成为高效、高性能并易于使用的Redis替代品。

### 基准测试

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

### Memcached / Dragonfly