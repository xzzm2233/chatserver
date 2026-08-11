# ChatServer — 集群聊天服务器

基于 [muduo](https://github.com/chenshuo/muduo) 网络库实现的 C++ 集群聊天服务器，可部署在 Nginx TCP 负载均衡环境之上，通过 **Redis 发布/订阅** 机制实现多服务器实例间的跨节点消息路由。

## 功能概览

- **用户管理**：注册、登录、注销
- **一对一聊天**：实时文本消息，支持离线消息缓存
- **好友管理**：添加好友，查看在线状态
- **群组功能**：创建群组、加入群组、群聊
- **集群横向扩展**：多 ChatServer 实例通过 Redis Pub/Sub 互通，结合 Nginx TCP 负载均衡
- **异常容错**：服务器/客户端异常退出时自动重置用户状态

## 技术栈

| 组件 | 技术 |
|------|------|
| 网络框架 | muduo（Reactor 模式，4 I/O 线程） |
| 序列化 | nlohmann/json |
| 数据库 | MySQL（用户、好友、群组、离线消息） |
| 中间件 | Redis Pub/Sub（hiredis 客户端） |
| 负载均衡 | Nginx TCP Stream |
| 构建 | CMake |

## 编译

```bash
cd build && rm -rf * && cmake .. && make
# 或
bash autobuild.sh
```

---

## 后续优化方向

- [ ] **数据库连接池**：当前每次数据库操作都新建 MySQL 连接，高并发下应引入连接池
- [ ] **密码加密**：密码明文存储，应改为服务端加盐哈希（如 bcrypt）
- [ ] **双向映射**：`_userConnMap` 仅支持 `userid → conn` 正向查找，增加 `conn → userid` 反向索引可将异常退出时的查找复杂度从 O(n) 降至 O(1)
- [ ] **心跳机制**：客户端无心跳检测，死连接无法被及时清理
- [ ] **消息确认（ACK）**：发送消息后无投递确认，可能出现消息丢失
- [ ] **SQL 参数化**：当前使用 `sprintf` 拼接 SQL，存在注入风险，应改用参数化查询
- [ ] **Redis 高可用**：单点 Redis，引入 Sentinel 或 Cluster 消除单点故障
- [ ] **连接复用**：Redis 的 publish 和 subscribe 各持一个长连接，subscribe 的阻塞读取在独立线程中运行，可考虑 hiredis 异步模式优化