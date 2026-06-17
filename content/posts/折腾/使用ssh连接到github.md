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
#git config --global url."git@github-foo.com:foo/".insteadOf "git@github.com:foo/" 
#git config --global url."git@github-bar.com:bar/".insteadOf "git@github.com:bar/" 
Host github-foo.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/github-foo

Host github-bar.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/github-bar
```

## 参考

-   [通过 SSH 连接到 GitHub](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh)
-   [启用通过 HTTPS 的 SSH 连接](https://docs.github.com/zh/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)
-   [管理多个帐户](https://docs.github.com/zh/account-and-profile/how-tos/account-management/managing-multiple-accounts)
