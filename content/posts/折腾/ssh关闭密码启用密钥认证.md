+++
title = "禁用SSH密码认证并启用密钥认证"
date = "2025-02-01T00:00:00+08:00"
categories = ["折腾"]
tags = ["ssh"]
+++

## 生成公私钥

```sh
ssh-keygen -t ed25519 -f ~/.ssh/<file_name>
```

## 将公钥上传到服务器

```sh
ssh-copy-id -i <pub_path> <username>@<server_ip>
```

如果没有`ssh-copy-id`命令，可以手动操作：

```sh
cat ~/.ssh/<file_name>.pub | ssh <username>@<server_ip> "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

```sh
# 登录测试 (不要退出, 防止接下来配置出错否无法登录服务器)
ssh -i <key_file> -p <port> <user>@<host_name>
# 查看 ssh 的连接
ssh -O check <user>@<host_name>
# 结束 ssh 的连接
ssh -O exit <user>@<host_name>
```

## 修改 sshd 的配置

`/etc/ssh/sshd_config` (找到并修改以下参数)：

```
# 取消注释并修改为一个高端口号（如 2022, 2222等）, 为了避开自动化扫描脚本
Port 2222

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
PermitEmptyPasswords no

# 禁止 root 用户直接登录
PermitRootLogin no

# 启用公钥认证 (通常默认是启用的，但请确认)
PubkeyAuthentication yes

# 可选但推荐：配置允许登录的用户或用户组
# 将 ‘your_user’ 替换为你的实际用户名
AllowUsers your_user
# 或者允许一个用户组，例如 ‘ssh-users’
# AllowGroups ssh-users
```

防火墙开放端口:

```sh
sudo ufw allow 2222
```

## 重启SSH服务

```bash
sudo systemctl restart sshd   # 之前的连接不会中断
```

## 配置 ssh

```
Host <ip/host_name>
    #HostName <host_name>    # 真实 ip/host_name
    Port <port>
    User <user_name>
    IdentityFile ~/.ssh/<file_name>
```

## 测试连接

**先不要关闭当前会话**，新开一个终端测试连接：

```bash
ssh <username>@<server_ip>
```

确认可以正常登录后，再关闭当前会话。

## 恢复方法

如果无法通过密码或密钥认证, 可以通过服务器控制台（如 AWS/Aliyun 的 VNC 或 KVM）直接访问。
