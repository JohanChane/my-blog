+++
title = "用 kitty + ssh + zellij 作为 ssh 客户端"
date = "2025-09-11T00:00:00+08:00"
categories = ["折腾"]
tags = ["ssh", "ssh客户端"]
+++

## ssh 客户端添加心跳包

因为有些路由器喜欢丢弃闲置过长的 TCP 连接 (即不传输任何数据)。

Edit `~/.ssh/config`:

```
Host *
    ServerAliveInterval 60
    TCPKeepAlive yes
    IdentitiesOnly yes
```

## 添加 ssh Host

Edit `~/.ssh/config`:

```
Host <alias_name>           # Your preferred alias for this connection
    HostName <host_ip>      # Actual hostname or IP address
    User <user_name>        # Username for authentication
    IdentityFile <private_key_path>  # Path to private key file
    Port 22                 # Optional: SSH port (default is 22)
```

## 服务器使用 zellij 防止 ssh 断开而导致耗时比较长的工作中断

zellij 的基本用法:

```sh
# New a session
zellij
# OR
zellij new -n mysession

# Detach session
`C-O + D`: detach current session

# Attach session
zellij list-sessions
zellij attach <the_session>
```

## 上传/下载文件

在本地使用 `scp/rsync` 即可。

## 常见错误

-   [I get errors about the terminal being unknown or opening the terminal failing or functional keys like arrow keys don’t work?](https://sw.kovidgoyal.net/kitty/faq/#i-get-errors-about-the-terminal-being-unknown-or-opening-the-terminal-failing-or-functional-keys-like-arrow-keys-don-t-work)

    ```sh
    使用 kitten ssh <host_name>
    # 如果还不行, 则
    infocmp -a xterm-kitty | ssh <host_name> tic -x -o \~/.terminfo /dev/stdin    # 会生成 ~/.terminfo/x/xterm-kitty
    ```
