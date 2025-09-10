+++
title = "使用 ssh 连接到 github"
date = "2025-04-17T00:00:00+08:00"
categories = ["折腾"]
tags = ["github", "ssh"]
+++

## 生成公私钥

```sh
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/github
```

`~/.ssh/config`:

```sh
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github
```

## 将公钥上传到 github

-   公钥: `~/.ssh/github.pub`
-   上传到 github: Settings -> SSH and GPG keys

## 测试是否成功

```sh
ssh -T git@github.com
```

## ssh over https

如果使用机场访问 github, 可以会出现无法连接的情况, 因为有些机场会屏蔽 22 端口以防止网络攻击。改为 https 协议即可。

```sh
Host github.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/github
```

## 有多个 github 用户时

ssh config:

```
Host github.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/github

Host github.com-<the other user name>
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/github-joshua
```

clone:

```sh
git clone git@github.com:xxx/xxx.git
git clone git@github.com-<the other user name>:xxx/xxx.git
```

## 参考

-   [通过 SSH 连接到 GitHub](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh)
-   [启用通过 HTTPS 的 SSH 连接](https://docs.github.com/zh/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)
