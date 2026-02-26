# Docker 从入门到实战：10分钟搞定容器化部署

> 还在为"我本地能跑啊"发愁？Docker 一次构建，到处运行。本文用最简单的方式带你上手。

## 为什么要用 Docker？

三个字：**一致性**。

开发环境、测试环境、生产环境完全一致，再也不会出现"我电脑上没问题"的尴尬。

## 核心概念（30秒理解）

- **镜像 (Image)**：应用的快照，类似安装包
- **容器 (Container)**：镜像的运行实例，类似安装后的程序
- **Dockerfile**：构建镜像的配方
- **docker-compose**：多容器编排工具

## 实战：部署一个 Python Web 应用

### 1. 写一个简单的 FastAPI 应用

```python
# app.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello Docker!"}

@app.get("/health")
def health():
    return {"status": "ok"}
```

### 2. 创建 Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 3. requirements.txt

```
fastapi==0.115.0
uvicorn==0.32.0
```

### 4. 构建和运行

```bash
# 构建镜像
docker build -t my-api .

# 运行容器
docker run -d -p 8000:8000 --name my-api my-api

# 查看日志
docker logs my-api

# 访问
curl http://localhost:8000
```

## 进阶：docker-compose 多服务编排

实际项目通常有多个服务。比如 API + 数据库 + Redis：

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

volumes:
  pgdata:
```

一条命令启动所有服务：

```bash
docker-compose up -d
```

## 常用命令速查

```bash
# 镜像操作
docker images              # 列出镜像
docker rmi <image>         # 删除镜像
docker pull nginx          # 拉取镜像

# 容器操作
docker ps                  # 运行中的容器
docker ps -a               # 所有容器
docker stop <container>    # 停止
docker rm <container>      # 删除
docker exec -it <c> bash   # 进入容器

# 清理
docker system prune -a     # 清理所有未使用资源
```

## 生产环境最佳实践

1. **使用多阶段构建**减小镜像体积
2. **不要用 root 用户**运行应用
3. **使用 .dockerignore** 排除不需要的文件
4. **固定版本号**，不要用 `latest` 标签
5. **健康检查**确保服务可用

```dockerfile
# 多阶段构建示例
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

FROM python:3.11-slim
COPY --from=builder /install /usr/local
WORKDIR /app
COPY . .
RUN useradd -m appuser
USER appuser
HEALTHCHECK CMD curl -f http://localhost:8000/health || exit 1
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 总结

Docker 的核心价值就是**环境一致性**和**快速部署**。掌握 Dockerfile + docker-compose 就能覆盖 90% 的使用场景。

---

*觉得有用就点赞收藏，后续更新 K8s 实战教程 👍*
