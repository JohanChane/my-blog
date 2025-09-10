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
ssh-copy-id <username>@<server_ip>
```

如果没有`ssh-copy-id`命令，可以手动操作：

```sh
cat ~/.ssh/<file_name>.pub | ssh <username>@<server_ip> "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

## 修改 sshd 的配置

`/etc/ssh/sshd_config` (找到并修改以下参数)：

```
PasswordAuthentication no               # 禁用密码认证
PubkeyAuthentication yes                # 启用公钥认证
ChallengeResponseAuthentication no      # 避免不必要的认证方式，简化安全配置。
UsePAM no                               # 禁用后，SSH 直接依赖自身的认证机制（如密钥）。
```

可选增强安全措施:

```
# 禁用root登录
PermitRootLogin no

# 限制允许的用户
AllowUsers <your_username>

# 更改默认端口
Port 2222
```

## 重启SSH服务

```bash
sudo systemctl restart sshd
```

## 配置 ssh

```
Host <ip/host_name>
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
