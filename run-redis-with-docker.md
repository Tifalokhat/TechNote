使用 Docker 部署 Redis 实例非常简单，以下是详细步骤和最佳实践：

---

### **1. 快速启动单节点 Redis**
#### 直接运行最新版 Redis 容器：
```bash
docker run --name my-redis -d -p 6379:6379 redis:latest
```
- `--name my-redis`：为容器命名  
- `-d`：后台运行  
- `-p 6379:6379`：将宿主机的 6379 端口映射到容器  
- `redis:latest`：使用官方最新镜像（可指定版本如 `redis:7.0`）

#### 验证是否运行成功：
```bash
docker ps | grep redis  # 查看容器状态
redis-cli ping         # 返回 "PONG" 表示成功
```

---

### **2. 持久化数据配置**
Redis 默认关闭持久化，需挂载卷保存数据：
```bash
docker run --name my-redis \
  -v /path/on/host:/data \  # 挂载数据目录
  -d redis:latest \
  redis-server --appendonly yes  # 启用AOF持久化
```
- **AOF（推荐）**：记录所有写操作（更安全）  
- **RDB**：可追加 `--save 60 1000`（60秒内1000次修改触发快照）

---

### **3. 自定义配置文件**
#### 步骤：
1. 下载官方配置文件：
   ```bash
   wget https://raw.githubusercontent.com/redis/redis/7.0/redis.conf -O /path/on/host/redis.conf
   ```
2. 修改关键配置（如密码、内存限制）：
   ```ini
   requirepass yourpassword  # 设置密码
   maxmemory 1gb            # 限制内存
   ```
3. 启动容器时加载配置：
   ```bash
   docker run --name my-redis \
     -v /path/on/host/redis.conf:/usr/local/etc/redis/redis.conf \
     -v /path/on/host/data:/data \
     -d redis:latest \
     redis-server /usr/local/etc/redis/redis.conf
   ```

---

### **4. 安全配置**
#### 设置密码访问：
```bash
docker run --name my-redis \
  -d redis:latest \
  redis-server --requirepass yourpassword
```
客户端连接时需验证：
```bash
redis-cli -a yourpassword
```

#### 限制网络访问（仅本地）：
```bash
docker run --name my-redis \
  -p 127.0.0.1:6379:6379 \  # 仅允许本地访问
  -d redis:latest
```

---

### **5. 生产环境建议**
#### **使用 Docker Compose**
创建 `docker-compose.yml`：
```yaml
version: '3'
services:
  redis:
    image: redis:7.0
    container_name: my-redis
    ports:
      - "6379:6379"
    volumes:
      - ./data:/data
      - ./redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    restart: unless-stopped  # 自动重启
```
启动：
```bash
docker-compose up -d
```

#### **关键参数优化**
| 配置项                | 作用                          | 示例值              |
|-----------------------|-----------------------------|--------------------|
| `maxmemory-policy`    | 内存满时的淘汰策略             | `allkeys-lru`      |
| `timeout`             | 客户端空闲超时（秒）           | `300`              |
| `protected-mode`      | 禁止外部无密码访问             | `yes`              |

---

### **6. 常用管理命令**
| 操作                  | 命令                          |
|-----------------------|-----------------------------|
| 进入容器               | `docker exec -it my-redis redis-cli` |
| 查看日志               | `docker logs my-redis`       |
| 停止/启动容器          | `docker stop/start my-redis` |
| 删除容器               | `docker rm -f my-redis`      |

---

### **7. 性能监控**
#### 使用 `redis-cli` 内置工具：
```bash
redis-cli info memory  # 查看内存使用
redis-cli monitor     # 实时监控命令
```
#### 通过 `docker stats`：
```bash
docker stats my-redis  # 查看CPU/内存占用
```

---

通过以上步骤，您可以根据需求灵活部署 Redis 实例。对于集群部署（如 Redis Cluster 或 Sentinel），需额外配置网络和节点关系。