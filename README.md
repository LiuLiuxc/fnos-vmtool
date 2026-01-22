# 虚拟机管理器

一个基于Docker的libvirt虚拟机管理工具，提供Web界面来控制虚拟机的启动、停止、暂停、恢复和重启。

专为飞牛OS设计，通过virsh命令直接管理虚拟机，无需HTTP API或SSH。

## 功能特性

- 🔐 **用户认证** - 基于会话的登录系统，保护管理操作
- 📊 查看所有虚拟机列表和状态
- ▶️ 启动虚拟机
- ⏹ 停止虚拟机
- ⏸ 暂停虚拟机
- ↺ 恢复虚拟机
- ↻ 重启虚拟机
- ⚡ 强制关机虚拟机
- 🔄 自动刷新状态（2秒间隔）

## 系统要求

- 安装了libvirt (KVM/QEMU)
- 可以使用virsh命令
- Docker环境
- Root权限

## 配置

配置通过环境变量设置，可以通过以下方式配置：

### 方式1: 使用 .env 文件 (推荐)

```bash
# 1. 复制示例文件
cp .env.example .env

# 2. 编辑 .env 文件，按需修改配置
nano .env

# 3. 启动服务（会自动读取 .env 文件）
docker-compose up -d
```

### 方式2: 直接修改 docker-compose.yml

编辑 `docker-compose.yml` 中的 `environment` 部分。

### 方式3: 运行时指定

```bash
docker run -d \
  -e ADMIN_USERNAME=myuser \
  -e ADMIN_PASSWORD="your-password" \
  -e SECRET_KEY="your-secret-key" \
  # ... 其他参数
```

### 环境变量说明

| 环境变量 | 说明 | 默认值 |
|---------|------|--------|
| `APP_PORT` | 应用端口 | `55005` |
| `ADMIN_USERNAME` | 管理员用户名 | `admin` |
| `ADMIN_PASSWORD` | 管理员密码（明文，直接设置更方便） | `admin123` |
| `ADMIN_PASSWORD_HASH` | 管理员密码哈希（向后兼容） | 空 |
| `SECRET_KEY` | Flask会话加密密钥 | `dev-secret-key-change-in-production` |

**注意**: 如果同时设置了 `ADMIN_PASSWORD` 和 `ADMIN_PASSWORD_HASH`，优先使用 `ADMIN_PASSWORD`。

### 生成密码哈希（可选）

如果需要使用密码哈希（向后兼容）：

```bash
# 生成密码哈希（用于 ADMIN_PASSWORD_HASH）
python -c "from werkzeug.security import generate_password_hash; print(generate_password_hash('你的强密码'))"

# 生成安全密钥（用于 SECRET_KEY）
python -c "import secrets; print(secrets.token_hex(32))"
```

**更简单的方法**：直接设置 `ADMIN_PASSWORD` 环境变量，无需生成哈希：

```bash
# .env 文件示例
APP_PORT=55005
ADMIN_USERNAME=admin
ADMIN_PASSWORD=你的密码  # 直接设置明文密码
SECRET_KEY=a3f8c9d2e1b5a6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c
```

### 配置示例

```bash
# .env 文件示例
APP_PORT=55005
ADMIN_USERNAME=admin
ADMIN_PASSWORD=你的密码  # 直接设置明文密码，更方便
SECRET_KEY=a3f8c9d2e1b5a6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c
```

## 快速开始

### 使用Docker Compose（推荐）

```bash
# 构建并启动服务
docker-compose up -d

# 访问Web界面
# 在浏览器中打开: http://localhost:55005
```

### 使用Docker

```bash
# 构建镜像
docker build -t fnos-vmtool .

# 运行容器（需要root权限和libvirt访问）
docker run -d \
  -p 55005:55005 \
  --name fnos-vmtool \
  --privileged \
  -v /var/run/libvirt/libvirt-sock:/var/run/libvirt/libvirt-sock \
  fnos-vmtool

# 访问Web界面
# 在浏览器中打开: http://localhost:55005
```

### 直接运行（开发模式）

```bash
# 1. 配置环境变量（可选）
cp .env.example .env
# 编辑 .env 文件

# 2. 安装依赖
pip install -r requirements.txt

# 3. 运行服务
cd backend
python app.py

# 访问Web界面: http://localhost:55005

# 默认用户名: admin
# 默认密码: admin123
# 建议首次登录后立即修改密码
```

## API接口

### 获取所有虚拟机
```
GET /api/vms
```

### 启动虚拟机
```
POST /api/vms/{vm_id}/start
```

### 停止虚拟机
```
POST /api/vms/{vm_id}/stop
```

### 暂停虚拟机
```
POST /api/vms/{vm_id}/pause
```

### 恢复虚拟机
```
POST /api/vms/{vm_id}/resume
```

### 重启虚拟机
```
POST /api/vms/{vm_id}/restart
```

### 强制关机
```
POST /api/vms/{vm_id}/destroy
```

### 健康检查
```
GET /api/health
```

## 技术细节

- **基础镜像**: Python 3.11 Alpine（最小化镜像尺寸）
- **运行用户**: root（无需sudo，直接执行virsh命令）
- **依赖**: 仅安装libvirt-client（virsh命令必需）
- **后端**: Flask + Flask-CORS
- **前端**: 纯HTML/CSS/JavaScript

## 注意事项

1. 确保libvirt服务正在运行：`systemctl status libvirtd`
2. Docker容器需要访问libvirt的socket文件
3. 必须使用privileged模式以获得足够权限
4. 容器以root用户运行，直接调用virsh命令（无需sudo）
5. 默认应用运行在 `55005` 端口

## 新功能说明

### 用户认证系统
- 基于Flask session的自定义认证机制
- 保护所有虚拟机管理API端点（`/api/vms/*`）
- 默认用户名：`admin`，默认密码：`admin123`
- **生产环境建议**：
  - 修改默认用户名和密码
  - 在 `.env` 文件中设置 `ADMIN_PASSWORD`（直接设置明文密码，更方便）
  - 或者设置 `ADMIN_PASSWORD_HASH`（密码哈希，向后兼容）

## 安全建议

### 生成安全密码（推荐）

**简单方法 - 直接设置密码**：
```bash
# 在 .env 文件中直接设置密码
ADMIN_USERNAME=admin
ADMIN_PASSWORD=你的强密码  # 直接设置，无需哈希
```

**高级方法 - 使用密码哈希（向后兼容）**：
```bash
# 生成密码哈希
python -c "from werkzeug.security import generate_password_hash; print(generate_password_hash('你的强密码'))"

# 在 .env 文件中配置
ADMIN_USERNAME=admin
ADMIN_PASSWORD_HASH=生成的哈希值
```

生成安全密钥：
```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

### 更新已运行容器
```bash
# 停止并移除旧容器
docker compose down

# 使用新配置重新构建和启动
docker compose build --no-cache
docker compose up -d
```
