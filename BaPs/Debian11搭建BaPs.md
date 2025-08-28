# 前言
本文以Debian11为主要参考，其他系统请根据实际情况来操作

> 部分内容参考了[BAPS.pdf](https://github.com/diffghjkl/Git-Public/blob/main/BaPs/BAPS.pdf)






# 服务端
## 安装 GO语言
通过 [Go 官方下载页面](https://go.dev/dl/) 查看最新稳定版本

下载 Go 压缩包（以架构为 amd64 & go1.21.6 为例）
```bash
wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz
```

删除任何可能存在的旧版本，然后解压
```bash
# 删除之前安装的 Go (如果存在)
sudo rm -rf /usr/local/go

# 将下载的压缩包解压到 /usr/local
sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz
```

通过编辑 shell 配置文件来设置环境变量（例如 ~/.bashrc 或 ~/.profile，如果你使用 Zsh，则是 ~/.zshrc）
```bash
nano ~/.bashrc
```

在文件的末尾添加以下几行：
```bash
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
> 其中：  
> `PATH=$PATH:/usr/local/go/bin`：将 Go 的可执行文件目录添加到系统的 PATH 中  
> `GOPATH=$HOME/go`：设置 `GOPATH`，这是你的工作空间目录（用于存放代码和二进制依赖）  
> `PATH=$PATH:$GOPATH/bin`：将 `GOPATH` 下的 `bin` 目录添加到 `PATH` ，这样通过 `go install` 安装的命令行工具就可以直接运行了  

让配置立即生效：
```bash
source ~/.bashrc
```

验证GO是否安装成功
```bash
go version
```

检查GO环境配置信息
```bash
go env
```
> 确认 GOROOT（Go 的安装目录）和 GOPATH 是否已正确设置  






## 安装服务端
> 以下版本选择其一即可  
### BaPs（GPT修改版）
下载服务端
```
wget https://github.com/diffghjkl/Git-Public/releases/download/0.1/game.zip
```

安装 unzip（解压zip文件） 和 zip（创建zip压缩包）
```
sudo apt update
apt install unzip zip
```

验证安装
```
unzip -v
zip -v
```

解压文件（以解压到 `/root/1/` 路径下为例）
```
unzip game.zip -d /root/1
```


#### 首次运行
请确保 `resources/Excel/` 目录下有配置文件
若无，请执行以下命令:
```
cd /root/game/BaPs
python3 fetch_resources.py
```


#### 非首次运行
```
cd /root/game/BaPs
go run ./cmd/BaPs
```






### BaPs-lite
下载并解压：
```
wget https://github.com/diffghjkl/Git-Public/releases/download/source-5683c59/BaPs_lite_5683c59_linux-amd64.zip

wget https://github.com/diffghjkl/Git-Public/releases/download/source-5683c59/data.zip

unzip BaPs_lite_5683c59_linux-amd64.zip -d /root/game/BaPs-lite

unzip data.zip -d /root/game/BaPs-lite/data
```

赋予权限后运行：
```
cd /root/game/BaPs-lite

chmod +x BaPs_lite_linux_amd64

./BaPs_lite_linux_amd64 -g true
```






## 安装 Screen（可选）
```bash
sudo apt install screen
```

验证安装
```bash
screen --version
```

运行命令（示例）： （创建一个名为test的screen）
```Shell
screen -S test
```

重新连接命令（示例）： （重新连接一个名为test的screen）
```Shell
screen -r test
```






## 安装 Mitmproxy
```
wget https://downloads.mitmproxy.org/12.1.2/mitmproxy-12.1.2-linux-x86_64.tar.gz

tar -xzf mitmproxy-12.1.2-linux-x86_64.tar.gz

sudo cp mitmproxy mitmdump mitmweb /usr/local/bin/
```

验证安装
```
mitmproxy --version
```
> 现在可以运行任何 mitmproxy 的可执行文件了  
执行 `mitmweb` 命令来启动WEB界面  
> 首次运行时会自动在 `~/.mitmproxy/` 目录下生成 CA 证书；监听地址为默认 `0.0.0.0:8080`，Web 界面默认为 `127.0.0.1:8081`  


### 访问Mitmproxy的WEB端口被拒绝
仅执行 `mitmweb` 命令时，绑定的 IP 为默认的 `127.0.0.1`  
> 1.这意味着仅本机发起的请求被允许，从外部连接则会被拒绝    
> 2.加入命令行参数（如 `mitmweb --web-host 0.0.0.0`）时，只会在当前这次运行会话中有效  

创建或编辑配置文件
```bash
mkdir -p ~/.mitmproxy
nano ~/.mitmproxy/config.yaml
```

在文件中添加以下内容：
```yaml
web_host: 0.0.0.0
```
> 还可以同时设置其他默认值：  
> listen_host: 0.0.0.0（代理监听地址）  
> web_port: 8081（Web界面端口）  
> listen_port: 8080（代理端口）  


### 配置连接隧道
在 `mitmproxy` 目录下创建 `bapsproxy.py`：
```python
# KitanoSakura
# 脚本还没完善，请使用WireGuard进行代理
from mitmproxy import http

# 定义重定向规则
redirects = {
    "https://ba-jp-sdk.bluearchive.jp": "http://127.0.0.1:5000",
    "https://prod-gateway.bluearchiveyostar.com:5100/api/gateway": "http://127.0.0.1:5000/getEnterTicket/gateway",
    "https://prod-game.bluearchiveyostar.com:5000/api/gateway": "http://127.0.0.1:5000/api/gateway",
    "https://prod-logcollector.bluearchiveyostar.com:5300": "http://127.0.0.1:5000/game/log",
}

def request(flow: http.HTTPFlow) -> None:
    for original_url, redirected_url in redirects.items():
        if flow.request.pretty_url.startswith(original_url):
            flow.request.url = flow.request.pretty_url.replace(original_url, redirected_url)
            print(f"Redirecting {original_url} to {redirected_url}")
            break
```

在 `mitmproxy` 目录下创建启动脚本 `proxy.sh` ：
```bash
#!/bin/bash
mitmweb -m wireguard --no-http2 -s bapsproxy.py --ignore-hosts 你的服务端地址
```
> 使用 `ip a` 或 `hostname -I` 查看本机 IP

赋予脚本权限：
```bash
chmod +x proxy.sh
```

启动Mitmproxy:
```bash
./proxy.sh
```






## 获取 Mitmproxy 的 CA 证书
> 证书在首次运行 `mitmproxy`, `mitmdump` 或 `mitmweb` 后生成在 `~/.mitmproxy/` 目录下  

进入证书目录:
```bash
cd ~/.mitmproxy/
```

查看生成的证书文件:
```
ls -l
```
目录下包含的文件：
- `mitmproxy-ca-cert.cer`：仅包含Mitmproxy的根证书（公钥），DER编码（二进制格式）【首选用于客户端安装，二进制格式兼容性最好】
- `mitmproxy-ca-cert.pem`：仅包含Mitmproxy的根证书（公钥），PEM编码（文本格式）。内容与.cer文件相同，编码不同【若客户端支持该格式，他是次选】
- `mitmproxy-ca.p12`：包含根证书和对应的私钥，并用密码（默认mitmproxy）加密打包【不要使用在客户端上】
- `mitmproxy-ca.pem`：同时包含根证书和私钥，PEM编码（文本格式）【不要使用在客户端上】
- `mitmproxy-dhparam.pem`：Diffie-Hellman密钥交换参数（用于增强加密连接的安全性，与证书信任无关）

[可选项] 将证书复制到你想放的路径（以 `Downloads` 目录 & `mitmproxy-ca-cert.cer` 为例）
```
cp mitmproxy-ca-cert.cer ~/Downloads/mitmproxy-ca-cert.cer
```

然后通过FTP等手段将证书文件下载至客户端即可





# 客户端
## 安装 CA 证书

### Android
> Tips1:  
> Android 7.0 (Nougat) 及以下：  
> 系统默认**信任所有用户安装的证书**  
> 应用开发者需要**显式编写代码**来限制应用不信任用户证书  
> 绝大多数应用为了省事和兼容性，都不会这么做。因此，**用户安装的证书对所有应用都有效**  

> Tips2:  
> Android 7.0 (Nougat) 以上：  
> 谷歌引入了名为“网络安全配置”（Network Security Configuration）的功能  
> 策略发生了**根本性改变**：应用默认**只信任系统证书**，而**不信任用户安装的证书**  
> 如果应用想要信任用户证书，开发者必须在应用的配置文件中**显式地声明**这一点  
> 绝大多数涉及敏感数据（如游戏、金融、社交）的应用都不会声明这一点，以增强安全性  


#### 方法一：安装为用户证书(仅适用于安卓7及以下)
在手机上通过 `设置` -> `安全` -> `加密与凭据` -> `安装证书` -> `CA 证书` 安装 `mitmproxy-ca-cert.cer` 证书  
安装成功后，通过手机 Wi-Fi 的手动代理，配置 `主机名`（服务器IP）& `端口`（Mitmproxy 的监听端口，默认为 8080）  

#### 方法二：安装为系统证书（支持安卓7以上&需要Root）
> MuMu模拟器需开启ROOT权限&可写系统盘  

ADB连接（端口默认5555）
```bash
adb connect 127.0.0.1:5555
```

获取root权限
```
adb root
```


将证书文件推送到系统证书目录
```
adb push 证书所在路径\mitmproxy-ca-cert.cer /sdcard/
```

```
adb shell
mount -o rw,remount /system
cp /sdcard/mitmproxy-ca-cert.cer /system/etc/security/cacerts/c8750f0d.0
chmod 644 /system/etc/security/cacerts/c8750f0d.0
mount -o ro,remount /system
exit
```

重启设备
> MuMu模拟器请手动重启
```
adb reboot
```

安装成功后，通过手机 Wi-Fi 的手动代理，配置 `主机名`（服务器IP）& `端口`（Mitmproxy 的监听端口，默认为 8080）

