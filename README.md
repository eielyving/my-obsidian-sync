# my-obsidian-sync
个人obsidian插件Self-hosted LiveSync部署
## 第一步：在 VPS 上部署数据库 (CouchDB)
由于你提到有 VPS，我们使用 Docker 部署 CouchDB（这是 LiveSync 插件要求的数据库类型）。
### 1.登录你的 VPS，创建一个文件夹：
```mkdir -p obsidian-sync && cd obsidian-sync```
### 2.创建部署文件```nano docker-compose.yml```，粘贴以下内容：
```
version: '3'
services:
  couchdb:
    image: couchdb:latest
    container_name: obsidian_couchdb
    ports:
      - "5984:5984"
    environment:
      - COUCHDB_USER=your_admin_user      # 替换你的用户名
      - COUCHDB_PASSWORD=your_admin_password  # 替换你的密码
    volumes:
      - ./data:/opt/couchdb/data
      - ./etc:/opt/couchdb/etc/local.d
    restart: always
```

### 3.启动服务：```docker-compose up -d```

### 4.重要设置：浏览器打开 ```http://你的VPS_IP:5984/_utils```，登录后进入 ```Settings``` -> ```CORS```，
###   选择 Enable CORS，并在 ```Allow Origins``` 填入 ```app://obsidian.md, capacitor://localhost, http://localhost```。

## 第二步：Windows 端配置
### "Self-hosted LiveSync"配置插件：
1. 在插件设置中选择 Minimal Setup。
2. URI: http://你的VPS_IP:5984
3. Username / Password: 刚才你在 Docker 里设置的账号密码。
4. Database Name: 填 obsidian（插件会自动创建）。
5. 点击 Check and Update，如果显示连接成功，选择 LiveSync 模式。

## 😶‍🌫️VPS安装 Docker 官方密钥和仓库
```
# 安装必要的证书工具
apt-get update
apt-get install ca-certificates curl gnupg -y

# 添加 Docker 官方 GPG 密钥
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

# 添加 Docker 仓库到系统源
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
```
## 正式安装 Docker 引擎和 Compose
```apt-get update
apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
## 3. 验证安装是否成功
```docker --version```
如果看到 ```Docker version 2x.x.x...```，说明 Docker 装好了。
## 4. 启动你的 CouchDB
```
cd ~/obsidian-sync
docker compose up -d
```
## 5. 检查开放 TCP 端口：5984


## 当 ```docker compose up -d``` 成功运行后：

在浏览器输入 ```http://你的VPS公网IP:5984/_utils```。

能看到一个蓝色的登录界面。

输入你在 yml 里设置的 ```COUCHDB_USER``` 和 ```COUCHDB_PASSWORD```。









