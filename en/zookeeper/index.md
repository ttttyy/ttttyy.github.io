# ZooKeeper论文阅读笔记

&emsp;&emsp;ZooKeeper是一个分布式开源协调服务，它为更加复杂的协调原语提供了一个简单高效的内核，并为开发者提供了API以便他们能开发自己的协调原语。
### ZooKeeper服务

#### 服务概览
ZooKeeper给clients提供一个数据节点(znode)集合的抽象，client可以对这些znode进行数据操作，这些节点按层次结构组织。
client可以创造两种znode,

- regular, client可以显式地创建和删除这些znode
- ephermeral,client可以显示地创建或删除，也可以让系统在创建了znode的session结束时自动删除

另外，client可以在创建znode时设置sequential flag，拥有sequential flag的znode会有一个单调增的sequence value,即父节点的sequence value小于子节点的值。
ZooKeeper实现了watch机制，client在数据变化时可以不用轮询就可以及时收到通知。一个watch只会在一个session中触发一次。如果在一个session中数据变化了两次，client只会收到一次通知，因为在数据第一次变化后watch已被触发失效了。

{{< admonition type=tip title="This is a tip" open=true >}}
session：一个client从连接到ZooKeeper到断开连接的过程称为一个session
{{< /admonition >}}

#### Client API

- create(path, data, flags)：在path下新建一个znode，将data保存在其中，flags用来设置znode类型，以及设置sequential标志；

- delete(path, version)：如果version参数匹配，删除path对应的znode；

- exists(path, watch)：检查path对应的znode是否存在，watch让client观测这个znode；

- getdata(path, watch)：返回znode对应的数据和元数据，包括版本信息；

- setdata(path, data, version): 如果version参数匹配，将data写入znode；

- getChildren(path, watch)：返回znode的子节点

- sync(path)：等待所有未决的更新。

以上这些API都有阻塞和非阻塞两个版本。

### ZooKeeper保证

- 所有更新串行写入；
- 同一个client的请求按先到先服务顺序执行

### 原语示例

#### 配置管理

将所有配置存储到一个znode中，其他节点通过watch机制获取更新。

#### 会合
有些分布式系统包含一个主节点和多个工作节点，但这些节点都通过一个调度器调度，工作节点不知道主节点的地址和端口等配置信息。因此可以将主节点信息放入一个znode，工作节点通过znode获取主节点信息。

#### 组成员关系
当一个组成员进程启动时，会在组对应的znode下创建一个ephemeral类型的子节点，该进程结束后子节点自动删除。进程也可以通过组znode的子节点获取组信息。

#### 简单锁
锁可以通过创建一个ephemeral znode实现，如果创建成功，client获取锁。如果已经存在锁，client只能等待之前的锁被删除后才能创建自己的锁。

#### 读写锁

写锁与普通锁类似，与其他锁互斥。而读锁间互相兼容，与写锁互斥。

#### 双栅栏

双栅栏用来保证多个client的计算同时开始和同时结束。client开始计算前添加znode到barrier的znode之下，结束计算时删除znode。当barrier的znode下的子节点数达到阈值后才会启动计算。client可以等待一个特殊的ready的znode的创建，当数量到达阈值后创建。client退出的时候需要等待子znode全部被删除，同样可以通过删除ready删除。

### ZooKeeper实现

<div align=center><img src="zkcomponents.jpg" width="800"></div>

#### 请求处理器

leader收到写入请求后，会将其转换为幂等的事务，根据请求内容计算出新的数据、版本号和时间戳，等待应用到数据库中。

#### 原子广播

ZooKeeper使用原子广播协议Zab,Zab使用简单多数投票确保一致性。Zab保证广播收到和发出的顺序是一致的。leader节点广播之前需要确保已经收到了前一个leader的广播

#### 多副本数据库

当服务器故障后，使用周期性的快照和快照之后的日志恢复。创建快照的时候并不需要锁定，因为事务都是幂等的，因此再次应用已经应用的修改没有影响。

#### 客户端-服务器交互

- 当server执行一个写入操作时，会通知watch的client并清楚watch。每个服务器只负责通知自己连接的客户端；
- 每个读取请求对应着一个zxid，对应服务器上看到的最后一个写入事务的ID。因为读取是在服务器本地进行，可能在读取之前的一些写入没有同步到客户端连接的服务器，ZooKeeper提供了sync操作，保证sync之后的读取操作都能够获得发生在sync之前的写入结果；
- 如果一个client连接到一个新的server，新server通过比较zxid,server zxid要大于client zxid,确保它的view不能低于client的view；
- 在超时时间内，server没有收到client的消息，server就会认为client session failure。因此，client要在在没有活动时发送heartbeat避免超时。

