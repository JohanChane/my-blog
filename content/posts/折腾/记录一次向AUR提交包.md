+++
title = "记录一次向 AUR 提交包"
date = "2024-08-02T17:42:00+08:00"
categories = ["折腾"]
tags = ["ArchLinux", "AUR", "ssh"]
+++

## 配置 ssh 的公私钥

生成公私钥:

```sh
ssh-keygen -f ~/.ssh/aur    # 默认是 ed25519
```

`~/.ssh/config`:

```
Host aur.archlinux.org
  IdentityFile ~/.ssh/aur
  User aur
```

向 aur 账号添加公钥:

```
My Account -> SSH 公钥 -> 更新
```

## 创建软件包仓库并提交

```sh
# 如果仓库不存在, 则会自动创建此仓库
git clone ssh://aur@aur.archlinux.org/<your_pkg_name>.git
```

编写 `PKGBUILD` (叫 AI 帮写一个, 再修改即可)。注意: arch 不要使用 'any', 有如下架构: `arch=('i686' 'pentium4' 'x86_64' 'arm' 'armv7h' 'armv6h' 'aarch64')`。See [PKGBUILD ref](https://wiki.archlinuxcn.org/wiki/PKGBUILD)

在编写完成 `PKGBUILD` 之后，本地测试安装:

```sh
# 本地测试安装
makepkg -si
```

本地测试安装没有问题之后，需要执行命令生成 `.SRCINFO`:

```sh
makepkg --printsrcinfo > .SRCINFO   # 最好每次修改 PKGBUILD 都生成一下
```

最后提交到仓库即可。

## 扩展

有些 rust 项目可能要加 `options=(!lto)`, 表示禁用 LTO (Link Time Optimization, 链接时优化) 优化。

For example:

```
==> Starting build()...
   Compiling clashtui v0.2.0 (/home/johan/Desktop/clashtui-git/src/clashtui/clashtui)
error: linking with `cc` failed: exit status: 1
  |
  = note: LC_ALL="C" PATH="/home/johan/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/bin:/home/johan/.cargo/bin:/home/johan/.local/share/nvim/mason/bin:/home/johan/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/bin:/usr/bin/site_perl:/usr/bin/vendor_perl:/usr/bin/core_perl:/usr/lib/rustup/bin" VSLANG="1033" "cc" "-m64" "/tmp/rustcNbtJ86/symbols.o" "/home/johan/Desktop/clashtui-git/src/clashtui/clashtui/target/release/deps/clashtui-0d6afe05a5124b5a.clashtui.4596715e9d95ac94-cgu.00.rcgu.o" "-Wl,--as-needed" "-L" "/home/johan/Desktop/clashtui-git/src/clashtui/clashtui/target/release/deps" "-L" "/home/johan/Desktop/clashtui-git/src/clashtui/clashtui/target/release/build/ring-63655233faed3ca5/out" "-L" "/home/johan/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/lib" "-Wl,-Bstatic" "/tmp/rustcNbtJ86/libring-623dc7df014ce26e.rlib" "/home/johan/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/lib/libcompiler_builtins-13fc9d1ed9c7a2bc.rlib" "-Wl,-Bdynamic" "-lgcc_s" "-lutil" "-lrt" "-lpthread" "-lm" "-ldl" "-lc" "-Wl,--eh-frame-hdr" "-Wl,-z,noexecstack" "-L" "/home/johan/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/lib" "-o" "/home/johan/Desktop/clashtui-git/src/clashtui/clashtui/target/release/deps/clashtui-0d6afe05a5124b5a" "-Wl,--gc-sections" "-pie" "-Wl,-z,relro,-z,now" "-Wl,--strip-all" "-nodefaultlibs"
  = note: /usr/bin/ld: /home/johan/Desktop/clashtui-git/src/clashtui/clashtui/target/release/deps/clashtui-0d6afe05a5124b5a.clashtui.4596715e9d95ac94-cgu.00.rcgu.o: in function `ring::ec::curve25519::scalar::MaskedScalar::from_bytes_masked':
          clashtui.4596715e9d95ac94-cgu.00:(.text._ZN4ring2ec10curve255196scalar12MaskedScalar17from_bytes_masked17h9b48566625b8cf9dE+0x20): undefined reference to `ring_core_0_17_8_x25519_sc_mask'
          /usr/bin/ld: /home/johan/Desktop/clashtui-git/src/clashtui/clashtui/target/release/deps/clashtui-0d6afe05a5124b5a.clashtui.4596715e9d95ac94-cgu.00.rcgu.o: in function `ring::limb::limbs_reduce_once_constant_time':
          /home/johan/.cargo/registry/src/index.crates.io-6f17d22bba15001f/ring-0.17.8/src/limb.rs:135:(.text._ZN125_$LT$ring..ec..suite_b..ecdsa..verification..EcdsaVerificationAlgorithm$u20$as$u20$ring..signature..VerificationAlgorithm$GT$6verify17h8b4fc1da0cefaaa7E+0x11c): undefined reference to `ring_core_0_17_8_LIMBS_reduce_once'
...
          /home/johan/.cargo/registry/src/index.crates.io-6f17d22bba15001f/ring-0.17.8/pregenerated/aesni-x86_64-elf.S:881:(.text+0xd1a): undefined reference to `ring_core_0_17_8_OPENSSL_ia32cap_P'
          /usr/bin/ld: /tmp/rustcNbtJ86/libring-623dc7df014ce26e.rlib(72a7603bd286d55e-x86_64-mont-elf.o): in function `ring_core_0_17_8_bn_mul_mont':
          /home/johan/.cargo/registry/src/index.crates.io-6f17d22bba15001f/ring-0.17.8/pregenerated/x86_64-mont-elf.S:26:(.text+0x1c): undefined reference to `ring_core_0_17_8_OPENSSL_ia32cap_P'
          /usr/bin/ld: /tmp/rustcNbtJ86/libring-623dc7df014ce26e.rlib(72a7603bd286d55e-x86_64-mont-elf.o): in function `bn_sqr8x_mont':
          /home/johan/.cargo/registry/src/index.crates.io-6f17d22bba15001f/ring-0.17.8/pregenerated/x86_64-mont-elf.S:788:(.text+0x8b7): undefined reference to `ring_core_0_17_8_OPENSSL_ia32cap_P'
          /usr/bin/ld: /tmp/rustcNbtJ86/libring-623dc7df014ce26e.rlib(72a7603bd286d55e-x86_64-mont5-elf.o):/home/johan/.cargo/registry/src/index.crates.io-6f17d22bba15001f/ring-0.17.8/pregenerated/x86_64-mont5-elf.S:24: more undefined references to `ring_core_0_17_8_OPENSSL_ia32cap_P' follow
          collect2: error: ld returned 1 exit status
  = note: some `extern` functions couldn't be found; some native libraries may need to be installed or have their path specified
  = note: use the `-l` flag to specify native libraries to link
  = note: use the `cargo:rustc-link-lib` directive to specify the native libraries to link with Cargo (see https://doc.rust-lang.org/cargo/reference/build-scripts.html#cargorustc-link-libkindname)

error: could not compile `clashtui` (bin "clashtui") due to 1 previous error
==> ERROR: A failure occurred in build().
    Aborting...
```

## 参考

-   [AUR 提交准则](https://wiki.archlinuxcn.org/wiki/AUR_%E6%8F%90%E4%BA%A4%E5%87%86%E5%88%99)
-   [PKGBUILD](https://wiki.archlinuxcn.org/wiki/PKGBUILD)
