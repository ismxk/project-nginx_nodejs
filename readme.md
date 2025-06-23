项目概述
在这个项目中，Nginx 将充当反向代理，所有访问 HTTP 请求将通过 Nginx 转发到后端的 Node.js 应用程序。

Node.js 应用会响应客户端请求。

我们会在 Windows 环境下安装和配置 Nginx。

1. 安装 Node.js 和 Nginx
1.1 安装 Node.js
下载 Node.js 安装包：

访问 Node.js 官网 下载 Windows 版本的安装包。

安装 Node.js：

按照提示安装并选择默认选项。

安装完成后，在命令行（PowerShell 或 CMD）中检查 Node.js 是否成功安装：

bash
node -v
npm -v


1.2 安装 Nginx（Windows 版）
下载 Nginx Windows 版：

访问 Nginx 官网 下载 Windows 版本的 Nginx 压缩包。

解压缩 Nginx：

将下载的 nginx-xxx.zip 解压到一个目录（例如 C:\nginx）。

启动 Nginx：

打开 CMD（命令提示符），进入 Nginx 解压目录：

bash
cd C:\nginx
启动 Nginx：

bash
start nginx
在浏览器中访问 http://localhost，如果看到 Nginx 默认页面，说明安装成功。


2. 构建 Node.js 应用
2.1 创建一个简单的 Node.js 应用
在某个目录（例如 C:\project\node-app）中创建一个新的 Node.js 项目：

bash
mkdir C:\project\node-app
cd C:\project\node-app
npm init -y


安装 express 模块（一个简单的 Web 框架）：

bash
npm install express


创建一个简单的 Node.js 服务器 server.js：

javascript
// server.js
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello from Node.js server!');
});

app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
启动 Node.js 应用：

bash
node server.js
在浏览器中访问 http://localhost:3000，你应该能看到 "Hello from Node.js server!"。


3. 配置 Nginx 作为反向代理
3.1 修改 Nginx 配置文件
找到 Nginx 配置文件 nginx.conf，该文件位于 Nginx 解压目录下的 conf 文件夹中，路径为：

bash
C:\nginx\conf\nginx.conf
打开 nginx.conf 文件，并找到 server 块。

修改配置文件，让 Nginx 将请求代理到 Node.js 应用（假设 Node.js 应用运行在 http://localhost:3000）：

nginx
server {
    listen       80;	#可改
    server_name  localhost;	#可改

    location / {
        proxy_pass http://localhost:3000;  # 将请求转发到 Node.js 服务器
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
保存并关闭 nginx.conf 文件。

3.2 重启 Nginx
回到 Nginx 安装目录，停止 Nginx：

bash
C:\nginx>nginx -s stop
重启 Nginx：

bash
C:\nginx>start nginx


4. 测试项目
打开浏览器，访问 http://localhost，这时应该会看到来自 Node.js 应用的响应：

pgsql
Hello from Node.js server!
说明 Nginx 已成功作为反向代理，将请求转发到 Node.js 服务器。


5. 项目总结
在这个项目中，你完成了以下步骤：

安装和配置 Node.js：构建了一个简单的 Node.js 应用，监听端口 3000 并返回一个简单的响应。

安装和配置 Nginx（Windows 版）：将 Nginx 配置为反向代理服务器，代理客户端请求到 Node.js 应用。

验证项目：通过访问 http://localhost，确保 Nginx 成功将请求转发给 Node.js 应用，并且可以看到正确的响应。


6. 扩展功能
6.1 启用 HTTPS（SSL）
为了在生产环境中启用 HTTPS，你需要配置 SSL 证书。Nginx 支持通过 ssl_certificate 和 ssl_certificate_key 指令启用 HTTPS。

6.2 负载均衡
你可以配置 Nginx 在多台 Node.js 服务器之间进行负载均衡。例如，将多个 Node.js 服务器添加到 upstream 块中：

nginx
upstream backend {
    server localhost:3000;
    server localhost:3001;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend;
    }
}
这样 Nginx 会根据轮询方式将请求分发到多个 Node.js 实例。


7.BUG 修复
nginx反向代理不转发 url：

1.停止 nginx：
cd C:\nginx
nginx -s stop

2.确保 nginx 进程已关闭：
tasklist | findstr nginx
如果命令返回空结果，则表示 Nginx 已成功关闭。如果有结果，表示 Nginx 仍在运行，你可能需要使用任务管理器手动终止它。

若非空，表明 Nginx 没有完全停止，可能是由于某些原因没有成功关闭，则通过 taskkill 命令直接终止所有 nginx.exe 进程：
taskkill /F /IM nginx.exe

再次使用 tasklist | findstr nginx 查看是否还存在 Nginx 进程。如果没有了，表示 Nginx 已成功关闭。

3.终止 Nginx 后，可以使用 netstat 或 tasklist 确认 80 或 443 端口是否仍然被占用：
netstat -ano | findstr :80
如果没有返回结果，说明这些端口没有被占用。

若非空，则表明端口被占用，可以根据 PID 查找具体是哪一个进程在占用这些端口：
tasklist /FI "PID eq 2560"

可以根据占用该端口的进程 ID（PID）来进一步终止进程：
taskkill /F /PID [PID号]

4.但通常占用端口的进程不能被终止，如微信等等
修改 Nginx 配置文件：让 Nginx 监听不同的端口，如 8080（listen 8080;）

5.启动/重启nginx：
start nginx/nginx -s reload

6.打开浏览器，访问 http://localhost:8080
转发到 Nginx 代理的应用（http://localhost:3000 上的 Node.js 应用）。
