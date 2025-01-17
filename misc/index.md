---
title: 杂项 - huangfeihu.xyz
toc:
  depth_from: 1
  depth_to: 4
  ordered: false
---
@import "/mystyle.less"

# 杂项 {ignore=True}
> 返回[首页](../index.html)

---------------------------

[TOC]

## Redis
Redis 是一个现在非常流行的非关系型数据库。Redis 中的数据是以键值对的形式存放在主存当中，由此有很高的查询速度，经常用在后台的请求缓存。同时为了实现持久化，可以将Redis的数据同步到外存中。
### 数据类型
Redis有五种数据类型：
- string(字符串)：一个键最大能存储 512MB。
- hash(哈希)：每个 hash 可以存储 $2^{32}-1$ 键值对（40多亿）。
- list(列表)：列表最多可存储 $2^{32}-1$ 元素 (4294967295，每个列表可存储40多亿)。
- set(集合)：集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 $ O(1) $ 。
- zset(sorted set: 有序集合)：Redis zset 和 set 一样也是string类型元素的集合，且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
### 持久化
Redis有两种持久化的方式：快照（RDB文件）和追加式文件（AOF文件）：
- RDB持久化方式会在一个特定的间隔保存那个时间点的一个数据快照。
- AOF持久化方式则会记录每一个服务器收到的写操作。在服务启动时，这些记录的操作会逐条执行从而重建出原来的数据。写操作命令记录的格式跟Redis协议一致，以追加的方式进行保存。
- Redis的持久化是可以禁用的，就是说你可以让数据的生命周期只存在于服务器的运行时间里。
- 两种方式的持久化是可以同时存在的，但是当Redis重启时，AOF文件会被优先用于重建数据。
### 架构模式

### 分布式锁

### 缓存穿透和缓存雪崩
#### 缓存穿透
一般的缓存系统，都是按照key去缓存查询，如果不存在对应的value，就应该去后端系统查找（比如DB）。一些恶意的请求会故意查询不存在的key,请求量很大，就会对后端系统造成很大的压力。这就叫做缓存穿透。
##### 如何避免？
- 对查询结果为空的情况也进行缓存，缓存时间设置短一点，或者该key对应的数据insert了之后清理缓存。
- 对一定不存在的key进行过滤。可以把所有的可能存在的key放到一个大的Bitmap中，查询时通过该bitmap过滤。
#### 缓存雪崩
当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，会给后端系统带来很大压力。导致系统崩溃。
##### 如何避免？
- 在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。
- 做二级缓存，A1为原始缓存，A2为拷贝缓存，A1失效时，可以访问A2，A1缓存失效时间设置为短期，A2设置为长期
- 不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。

## Docker

### 隔离策略

Linux中的PID、IPC、网络等资源是全局的，而NameSpace机制是一种资源隔离方案，在该机制下这些资源就不再是全局的了，而是属于某个特定的NameSpace，各个NameSpace下的资源互不干扰。
虽然有了NameSpace技术可以实现资源隔离，但进程还是可以不受控的访问系统资源，比如CPU、内存、磁盘、网络等，为了控制容器中进程对资源的访问，Docker采用control groups技术(也就是cgroup)，有了cgroup就可以控制容器中进程对系统资源的消耗了，比如你可以限制某个容器使用内存的上限、可以在哪些CPU上运行等等。
有了这两项技术，容器看起来就真的像是独立的操作系统了。

### 和虚拟机相比的优劣

Docker不是虚拟化方法。它依赖于实际实现基于容器的虚拟化或操作系统级虚拟化的其他工具。为此，Docker最初使用LXC驱动程序，然后移动到libcontainer现在重命名为runc。Docker主要专注于在应用程序容器内自动部署应用程序。应用程序容器旨在打包和运行单个服务，而系统容器则设计为运行多个进程，如虚拟机。因此，Docker被视为容器化系统上的容器管理或应用程序部署工具。
- 与虚拟机不同，容器不需要引导操作系统内核，因此可以在不到一秒的时间内创建容器。此功能使基于容器的虚拟化比其他虚拟化方法更加独特和可取。由于基于容器的虚拟化为主机增加了很少或没有开销，因此基于容器的虚拟化具有接近本机的性能
- 对于基于容器的虚拟化，与其他虚拟化不同，不需要其他软件。主机上的所有容器共享主机的调度程序，从而节省了额外资源的需求。
- 与虚拟机映像相比，容器状态(Docker或LXC映像)的大小很小，因此容器映像很容易分发。
- 容器中的资源管理是通过cgroup实现的。Cgroups不允许容器消耗比分配给它们更多的资源。虽然主机的所有资源都在虚拟机中可见，但无法使用。这可以通过在容器和主机上同时运行top或htop来实现。所有环境的输出看起来都很相似。

### 联合文件系统(UnionFS)

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统就是UnionFS。UnionFS是一种分层、轻量级并且高性能的文件系统。联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

## RabbitMQ

### 为什么要使用RabbitMQ

- 在分布式系统下具备异步,削峰,负载均衡等一系列高级功能;
- 拥有持久化的机制，进程消息，队列中的信息也可以保存下来。
- 实现消费者和生产者之间的解耦。
- 对于高并发场景下，利用消息队列可以使得同步访问变为串行访问达到一定量的限流，利于数据库的操作。
- 可以使用消息队列达到异步下单的效果，排队中，后台进行逻辑下单。
 
## Linux

### 栈大小
默认8MB，可以使用`ulimit`调整。

### 软链接和硬链接