# frp内网穿透http代理
* 准备一台公网服务器跟服务器对应的域名；
* 准备一台内网客户端一台；

## Github下载fpr安装包：https://github.com/fatedier/frp/releases
1. 同时在服务端和客户端下载解压：

   ```bash
   #wget https://github.com/fatedier/frp/releases/download/v0.52.3/frp_0.52.3_linux_amd64.tar.gz
   #tar -zxvf frp_0.52.3_linux_amd64.tar.gz
   #cd frp_0.52.3_linux_amd64
   ```

2. 服务端配置：`vim frps.toml`

   ```bash
   bindPort = 7000
   vhostHTTPPort = 80
   auth.method = "token"
   auth.token = "123456789"
   ```

   运行服务端：

   ```bash
   #./frps -c ./frps.toml
   ```

3. 内网客户端：vim frpc.toml

   ```bash
   serverAddr = "xxx.xxx.xxx.xxx"
   serverPort = 7000
   auth.method = "token"
   auth.token = "123456789"
   [[proxies]]
   name = "web"
   type = "http"
   localPort = 8080
   customDomains = ["your.domian.com"]
   ```

   运行客户端：

   ```bash
   #./frpc -c ./frpc.toml
   ```

## 改进：

由于每次启动都会占用终端，将 `frp` 客户端 (`frpc`) 设置为服务，使其能够像使用 `service frp-client start` 这样的命令启动，可以通过创建一个系统服务来实现。下面的步骤适用于使用 systemd 的系统（例如最新版本的 Ubuntu、Debian、CentOS 等）。

### 服务端：创建 frps 服务文件

1. **创建服务文件**：
   打开一个新的服务文件`frps.service`：

   ```bash
   sudo vim /etc/systemd/system/frps.service
   ```

2. **添加服务配置**：
   在打开的文件中添加以下内容：

   ```ini
   [Unit]
   Description=frp Server Service
   After=network.target
   
   [Service]
   Type=simple
   User=root  # 可以根据需要更改用户
   ExecStart=/path/to/frps -c /path/to/frps.toml
   
   [Install]
   WantedBy=multi-user.target
   ```

   替换 `/path/to/frpcs 和 `/path/to/frps.toml` 为你的 `frps` 可执行文件和配置文件的实际路径。

3. **重新加载 systemd**：
   保存并关闭文件后，重新加载 systemd 管理器配置：

   ```bash
   sudo systemctl daemon-reload
   ```

4. **启动 frpc 服务**：
   启动 `frpc` 服务：

   ```bash
   sudo systemctl start frps
   ```

5. **设置开机启动**：
   如果希望 `frpc` 在系统启动时自动运行，可以启用服务：

   ```bash
   sudo systemctl enable frps
   ```

#### 管理 frps 服务

- **启动服务**：`sudo systemctl start frps`
- **停止服务**：`sudo systemctl stop frps`
- **重启服务**：`sudo systemctl restart frps`
- **查看服务状态**：`sudo systemctl status frps`

这样一来，`frps` 就被设置为了一个系统服务，你可以使用 `systemctl` 命令来管理它。这将允许你以更传统的方式（如 `service frps start`）来控制 `frpc`。

### 客户端：创建 frpc 服务文件

1. **创建服务文件**：
   打开一个新的服务文件`frpc.service`：

   ```bash
   sudo vim /etc/systemd/system/frpc.service
   ```

2. **添加服务配置**：
   在打开的文件中添加以下内容：

   ```ini
   [Unit]
   Description=frp Client Service
   After=network.target
   
   [Service]
   Type=simple
   User=root  # 可以根据需要更改用户
   ExecStart=/path/to/frpc -c /path/to/frpc.toml
   
   [Install]
   WantedBy=multi-user.target
   ```

   替换 `/path/to/frpc` 和 `/path/to/frpc.toml` 为你的 `frpc` 可执行文件和配置文件的实际路径。

3. **重新加载 systemd**：
   保存并关闭文件后，重新加载 systemd 管理器配置：

   ```bash
   sudo systemctl daemon-reload
   ```

4. **启动 frpc 服务**：
   启动 `frpc` 服务：

   ```bash
   sudo systemctl start frpc
   ```

5. **设置开机启动**：
   如果希望 `frpc` 在系统启动时自动运行，可以启用服务：

   ```bash
   sudo systemctl enable frpc
   ```

#### 管理 frpc 服务

- **启动服务**：`sudo systemctl start frpc`
- **停止服务**：`sudo systemctl stop frpc`
- **重启服务**：`sudo systemctl restart frpc`
- **查看服务状态**：`sudo systemctl status frpc`

这样一来，`frpc` 就被设置为了一个系统服务，你可以使用 `systemctl` 命令来管理它。这将允许你以更传统的方式（如 `service frp-client start`）来控制 `frpc`。