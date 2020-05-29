# tgtunnel
Telegram Tunnel for Russian VPS

OpenConnect VPN客户端能安装在众多主流Linux系统中，如Fedora、RHEL、CentOS、Arch Linux、OpenSUSE和Debian等，参考在Debian 10 Buster上安装OpenConnect VPN Server的方法，Fedora可用sudo dnf install openconnect命令安装，RHEL、CentOS可用sudo yum install openconnect命令安装，Arch Linux可用sudo pacman -S openconnect命令安装。除了支持Linux外，还支持MacOS、Windows和OpenWRT，对于Android和iOS，可以使用Cisco AnyConnect客户端。以下为你讲解让OpenConnect VPN客户端随系统启动时自动连接的方法。  

Step 1 Install ocserv  
 `https://github.com/travislee8964/ocserv-auto`  
 
Step 2 Setup routing table  
 ```  
 #TG
route = 91.108.4.0/255.255.252.0
route = 91.108.8.0/255.255.248.0
route = 91.108.16.0/255.255.248.0
route = 91.108.36.0/255.255.254.0
route = 91.108.38.0/255.255.254.0
route = 91.108.56.0/255.255.252.0
route = 149.154.160.0/255.255.240.0
```  

Step 3 Setup auto-run

系统启动时自动连接  
为了让OpenConnect VPN客户端在引导时自动连接到服务器，我们可以创建一个systemd服务单元：  
`vi /etc/systemd/system/openconnect.service`  
将以下行放入该文件，请注意更换自己的参数，如vpn.example.com地址：  
```  
[Unit]
Description=OpenConnect VPN Client
After=network-online.target
Wants=network-online.target

[Service]
Type=idle
ExecStart=/bin/bash -c '/bin/echo -n 密码 | /usr/sbin/openconnect 服务器地址:端口号 -u 登录名 --passwd-on-stdin'
ExecStop=/usr/bin/pkill openconnect
Restart=always
RestartSec=2

[Install]
WantedBy=multi-user.target
```  

保存并关闭文件，然后启用此服务，以便它在引导时启动：  

`systemctl enable openconnect.service`  

文件内容说明：  
1、在After=network-online.target和Wants=network-online.target之后，在网络启动后运行此服务。  
2、实际上，该服务仍然可以在网络启动之前运行，如果此服务失败，我们会在2秒后添加Restart=always和RestartSec=2以重新启动此服务。  
3、Systemd无法识别管道重定向，因此，在ExecStart指令中，我们将命令包装在单引号中并使用Bash shell运行它。  
4、由于OpenConnect VPN客户端将作为在后台运行的systemd服务运行，因此无需在openconnect命令中添加-b参数。  

Ref: https://ywnz.com/linuxjc/5716.html  
