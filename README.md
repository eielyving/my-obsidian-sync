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

# HTTPS：Cloudflare
操作逻辑：
1. DNS 设置：在 Cloudflare 后台，将你的域名（如 ```sync.yourname.xyz```）指向你的 VPS IP。
2. 开启小黄云：确保 Proxy 状态为 On（即云朵变黄）。
3. SSL/TLS 设置：在 Cloudflare 的 SSL 设置里，选择 "Flexible" 模式。
注意： 开启 Cloudflare 后，你在 Obsidian 插件里填写的地址就要改成：```https://sync.yourname.xyz``` (不需要加 5984 端口，因为 Cloudflare 默认处理 443 端口。如果你非要用 5984 端口，Cloudflare 免费版是不支持的，建议把 Docker 里的端口映射改为 ```- "80:5984"```，或者在 Cloudflare 里用 ```Origin Rules``` 转发端口)。

# 一键脚本
```
mkdir -p obsidian-sync && cd obsidian-sync

# 1. 从你的 GitHub 仓库下载文件 (已设为公开)
curl -O https://raw.githubusercontent.com/eielyving/my-obsidian-sync/main/docker-compose.yml
curl -O https://raw.githubusercontent.com/eielyving/my-obsidian-sync/main/local.ini

# 2. 快速生成环境配置文件
read -p "设置你的数据库用户名: " db_user
read -p "设置你的数据库密码: " db_pass
echo "COUCHDB_USER=$db_user" > .env
echo "COUCHDB_PASSWORD=$db_pass" >> .env

# 3. 启动
docker compose up -d

echo "------------------------------------------------"
echo "部署成功！"
echo "如果你开启了 Cloudflare，地址为: https://你的域名"
echo "如果你直接用 IP，地址为: http://你的IP"
echo "------------------------------------------------"
```


