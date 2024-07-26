# ⚙️Secretflow_openNotebook && Lesson_3⚙️

![pic](img/image_1.jpg)

## 🎮SecretFlow🎮

### ⚙️SecretFlow 安装与配置⚙️

SecretFlow 是一个开源的隐私计算框架，支持多方安全计算、同态加密、差分隐私等多种隐私计算技术。具体安装与配置如下：

#### 运行要求

+ Python >= 3.8
+ 操作系统：
    + CentOS 7
    + Anolis8
    + Ubuntu 18.04/20.04
    + macOS 11.1+
    + WSL2
+ 资源 ≥ 8核16GB

#### SecretFlow 安装方式

##### 方式一：docker 镜像

![pic](img/image_2.png)

##### 方式二：pypi

```bash
pip >= 19.3
pip install –U secretflow
pip install –U secretflow-lite
```

##### 方式三：源码

1. 要求 pip >= 19.3
2. 建议使用 conda 管理 python 环境

```bash
git clone https://github.com/SecretFlow/SecretFlow.git
cd SecretFlow
pip install -r requirements.txt

# 安装 SecretFlow
pip install -e .

# 安装 SecretFlow-lite
pip install -e .[lite]
```

#### 配置 SecretFlow

+ 配置环境变量

```bash
export PYTHONPATH=$PYTHONPATH:/path/to/SecretFlow
```

+ 配置密钥

```bash
# 生成 RSA 密钥对
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 将公钥添加到 ~/.ssh/authorized_keys 文件中
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

+ 配置 SSH 代理

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

+ 配置数据库

```sql
CREATE DATABASE secretflow;
CREATE USER secretflow WITH ENCRYPTED PASSWORD 'secretflow';
GRANT ALL PRIVILEGES ON DATABASE secretflow TO secretflow;
```

+ 配置数据库连接

```bash
export DATABASE_URL=postgresql://secretflow:secretflow@localhost/secretflow
```

+ 配置 Redis

```bash
# 安装 Redis 并且配置 Redis 连接
export REDIS_URL=redis://localhost:6379/0
```

+ 配置 RabbitMQ

```bash
# 安装 RabbitMQ
export BROKER_URL=amqp://guest:guest@localhost:5672/
```

+ 配置日志

```bash
# 配置日志文件路径
export LOG_FILE=/path/to/secretflow.log
```

+ 配置 Web 服务

```
server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

# 启动 Web 服务
gunicorn secretflow.wsgi:application -b 127.0.0.1:8000 -w 4 -k gthread
```

至此，SecretFlow 安装配置完成。

### 🎮SecretFlow 仿真🎮

![pic](img/image_3.png)

#### 单机仿真

```python
>>> import secretflow as sf
>>> sf.init(parties=['alice', 'bob'], address='local')
>>> alice = sf.PYU('alice')
>>> bob = sf.PYU('bob')
>>> alice(lambda x : x + 1)(2)
<secretflow.device.device.pyu.PYUObject object at 0x7fe932a1a640>
>>> bob(lambda x : x - 1)(2)
<secretflow.device.device.pyu.PYUObject object at 0x7fe6fef03250>
```

#### 集群仿真

![pic](img/image_4.png)

##### 1. 在第一台机器上部署 Ray 主节点，模拟参与方 alice

```bash
ray start –-head \
--node-ip-address="{ip}" --port="{port} " \
--resources='{"alice": 16} ' \
--include-dashboard=False \
--disable-usage-stats
```

##### 2. 在第二台机器上部署 Ray 从节点，模拟参与方 bob

```bash
ray start \
--address="{Ray主节点的通信地址}" \
--resources=' {"bob": 16} ' \
--include-dashboard=False \
--disable-usage-stats
```

##### 3. 执行 python 代码

```bash
>>> import secretflow as sf
>>> sf.init(parties=['alice', 'bob'], address=' {ip:port}')
>>> alice = sf.PYU('alice')
>>> bob = sf.PYU('bob')
>>> alice(lambda x : x)(2)
<secretflow.device.device.pyu.PYUObject object at 0x7fe932a1a640>
>>> bob(lambda x : x)(2)
<secretflow.device.device.pyu.PYUObject object at 0x7fe6fef03250>
```

##### 4. 创建密态设备 SPU

```python
import spu
cluster_def = {
    'nodes': [{
    'party': 'alice',
    'address': '{ip:port of alice}',
}, {
    'party': 'bob',
    'address': '{ip:port of bob}',
},
],
    'runtime_config': {
    'protocol': spu.spu_pb2.SEMI2K,
    'field': spu.spu_pb2.FM128,
    'sigmoid_mode': spu.spu_pb2.RuntimeConfig.SIGMOID_REAL,
    }
}
    spu = sf.SPU(cluster_def=cluster_def)
```

### ⚙️生产模式⚙️

![pic](img/image_5.png)

![pic](img/image_6.png)
