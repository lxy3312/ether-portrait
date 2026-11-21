# 部署到自己的服务器，绑自己的域名

这个工具就是几个静态文件：`index.html`、`examples/` 和 `docs/`。没有后端，没有数据库，任何能放网页的地方都能放。

下面有两种办法，选一种就行。办法一最省事；办法二是放在你自己的云服务器上。

服务器在国外或香港，绑域名不用备案。服务器在中国大陆的话，域名要先做 ICP 备案才能用。

## 先买一个域名

在任意一家域名注册商买一个，比如 Cloudflare、Namecheap、阿里云国际站、腾讯云国际站。买完以后，在注册商的后台找到「DNS 解析」或者「DNS 记录」，下面两种办法都要在这里加一条记录。

下面的例子都假设你想用 `ether.example.com` 打开这个工具，把 `example.com` 换成你自己的域名。

## 办法一：域名指向 GitHub Pages

不用自己的服务器，免费，自带 HTTPS。

1. 在 DNS 里加一条记录：
   - 类型：`CNAME`
   - 主机记录（名称）：`ether`
   - 记录值（目标）：`lxy3312.github.io`

   想直接用不带前缀的 `example.com`，就改成加四条 `A` 记录，主机记录填 `@`，记录值分别是：
   `185.199.108.153`、`185.199.109.153`、`185.199.110.153`、`185.199.111.153`
2. 打开 GitHub 上这个仓库，点 Settings → Pages，在 Custom domain 里填 `ether.example.com`，点 Save。
3. 等几分钟到一两个小时，DNS 生效、证书签好以后，在同一个页面勾上 Enforce HTTPS。

GitHub 会在仓库里自动加一个叫 `CNAME` 的文件，记着你的域名，别删它。

## 办法二：放到你自己的云服务器

下面的命令按 Ubuntu 或 Debian 写的，大部分云服务器默认就是这两种系统。

1. 在 DNS 里加一条记录：
   - 类型：`A`
   - 主机记录：`ether`
   - 记录值：你服务器的公网 IP
2. 在云服务器的控制台，把安全组或防火墙的 80 和 443 端口打开。
3. 登录服务器：

   ```bash
   ssh root@你的服务器IP
   ```

4. 装 nginx 和 git：

   ```bash
   sudo apt update
   sudo apt install -y nginx git
   ```

5. 把这个仓库拉到服务器上：

   ```bash
   sudo git clone https://github.com/lxy3312/ether-portrait.git /var/www/ether-portrait
   ```

6. 新建 nginx 配置：

   ```bash
   sudo nano /etc/nginx/sites-available/ether-portrait
   ```

   粘贴下面这段，把 `ether.example.com` 换成你的域名，然后按 `Ctrl+O` 回车保存，`Ctrl+X` 退出：

   ```nginx
   server {
       listen 80;
       server_name ether.example.com;

       root /var/www/ether-portrait;
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }

       # 不让别人下载 .git 文件夹
       location ~ /\.git {
           deny all;
       }

       gzip on;
       gzip_types text/html text/css application/javascript image/svg+xml;
   }
   ```

7. 启用配置，检查一下，重新加载：

   ```bash
   sudo ln -s /etc/nginx/sites-available/ether-portrait /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```

   现在用 `http://ether.example.com` 应该就能打开了。

8. 加上 HTTPS（免费证书，会自动续期）：

   ```bash
   sudo apt install -y certbot python3-certbot-nginx
   sudo certbot --nginx -d ether.example.com
   ```

   按提示填一个邮箱、同意条款就行。完成以后就能用 `https://ether.example.com` 打开了。

9. 以后这个仓库更新了，在服务器上跑一下就是最新版：

   ```bash
   cd /var/www/ether-portrait && sudo git pull
   ```

## 遇到问题

- 打开是 404：看看 nginx 配置里 `root` 的路径对不对，`/var/www/ether-portrait/index.html` 在不在。
- 打不开、一直转圈：多半是安全组没开 80 或 443 端口，或者 DNS 还没生效。可以用 `ping ether.example.com` 看看解析出来的 IP 对不对。
- 「示例」按钮没反应：一定要通过网址打开。直接双击 `index.html` 的时候，浏览器不让网页读旁边的文件。
- 国内打开 GitHub Pages 偶尔很慢：用办法二，服务器选香港的会快很多。
