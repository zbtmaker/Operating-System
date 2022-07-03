# Kafka安装
## 一、使用HomeBrew安装Kafka
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew install kafka
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/op
######################################################################## 100.0%
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/ca
######################################################################## 100.0%
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/op
curl: (22) The requested URL returned error: 404                              

Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/1.1/manifests/1.1.1o
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/1.1/blobs/sha256:630f15
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sh
######################################################################## 100.0%
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/ka
######################################################################## 100.0%
==> Installing dependencies for kafka: openjdk, ca-certificates and openssl@1.1
==> Installing kafka dependency: openjdk
==> Pouring openjdk-18.0.1.1.monterey.bottle.tar.gz
🍺  /usr/local/Cellar/openjdk/18.0.1.1: 641 files, 307.7MB
==> Installing kafka dependency: ca-certificates
==> Pouring ca-certificates-2022-04-26.all.bottle.tar.gz
==> Regenerating CA certificate bundle from keychain, this may take a while...
🍺  /usr/local/Cellar/ca-certificates/2022-04-26: 3 files, 215.5KB
==> Installing kafka dependency: openssl@1.1
==> Pouring openssl@1.1-1.1.1o.monterey.bottle.tar.gz
Error: No such file or directory @ rb_sysopen - /Users/zoubaitao/Library/Caches/Homebrew/downloads/3465cbaa9e4d6661e5b5a1a02f37a8e7d41345392c56dd8d51688c656a554bb1--openssl@1.1-1.1.1o.monterey.bottle.tar.gz
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew install openssl
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/openssl%403-3.0.3.monterey.bottle.tar.gz
curl: (22) The requested URL returned error: 404                              

Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/3/manifests/3.0.3
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/3/blobs/sha256:044032dcdffb16c0bf7949fdf847f5f957e727bef1f982d8a7f156f727c81ad5
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:044032dcdffb16c0bf7949fdf847f5f957e727bef1f982d8a7f156f727c81ad5?se=2022-07-03T03%3A15%3A00Z&sig=4nfhjG6SMSzOW2AkNoPpIfRwignkXVvZkW9n8nd45rk%3D&sp=r&spr=https&sr=b&sv=
######################                                                    31.5%^C
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew install openssl@1.1
openssl@1.1  is already installed but outdated (so it will be upgraded).
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/openssl%401.1-1.1.1o.monterey.bottle.tar.gz
curl: (22) The requested URL returned error: 404                              

Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/1.1/manifests/1.1.1o
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/75e64159933cb7401740964ef8681602932bd4c67aade21021fd522238a05e20--openssl@1.1-1.1.1o.bottle_manifest.json
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/1.1/blobs/sha256:630f1510f71f8ad7f9521d3e7371bee08f1956544a48d9796fb5d1fab4058581
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/9814e80d4f5817f82a97364dbbe57b01123e34f6fa4c196e563d48d871530ecd--openssl@1.1--1.1.1o.monterey.bottle.tar.gz
==> Pouring openssl@1.1--1.1.1o.monterey.bottle.tar.gz
==> Caveats
A CA file has been bootstrapped using certificates from the system
keychain. To add additional certificates, place .pem files in
  /usr/local/etc/openssl@1.1/certs

and run
  /usr/local/opt/openssl@1.1/bin/c_rehash

openssl@1.1 is keg-only, which means it was not symlinked into /usr/local,
because macOS provides LibreSSL.

If you need to have openssl@1.1 first in your PATH, run:
  echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> ~/.zshrc

For compilers to find openssl@1.1 you may need to set:
  export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
  export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"

For pkg-config to find openssl@1.1 you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/openssl@1.1/lib/pkgconfig"

==> Summary
🍺  /usr/local/Cellar/openssl@1.1/1.1.1o: 8,089 files, 18.5MB
==> `brew cleanup` has not been run in the last 30 days, running now...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
Removing: /usr/local/Cellar/ca-certificates/2021-10-26... (3 files, 208.5KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/ca-certificates--2021-10-26... (117.6KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/fontconfig--2.13.1... (1.2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/freetype--2.11.0... (957.0KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/gdbm--1.22... (256.5KB)
Removing: /usr/local/Cellar/openjdk/17.0.1_1... (639 files, 305.2MB)
Removing: /usr/local/Cellar/openjdk/18.0.1... (641 files, 307.7MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/openjdk--18.0.1.monterey.bottle.tar.gz... (181.3MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/openjdk--17.0.1_1... (180.2MB)
Removing: /usr/local/Cellar/openssl@1.1/1.1.1l_1... (8,073 files, 18.5MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/python@3.9--3.9.9... (13.8MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/sqlite--3.37.0... (2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/librsvg--2.50.7... (37.2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/libtool--2.4.6_4... (1004.8KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/netpbm--10.86.26... (1.8MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/xorgproto--2021.5... (695.5KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/jasper--2.0.33... (396.4KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/pango--1.50.0... (815.3KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/jpeg--9d... (311.5KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/glib--2.70.2... (6.3MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/gdk-pixbuf--2.42.6... (782.8KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/harfbuzz--3.1.2... (2.2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/graphviz--2.49.3... (3.7MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/libtiff--4.3.0.monterey.bottle.tar.gz... (1.2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/icu4c--69.1... (27.2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/libx11--1.7.2... (2.1MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/gobject-introspection--1.70.0_2... (1.8MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/libxcb--1.14_1... (919.9KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/webp--1.2.1_1... (888.6KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/fribidi--1.0.11... (94KB)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/libpng... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/freetype... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/openjdk@11... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/maven... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/fontconfig... (49.6KB)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/gradle... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/openjdk... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/maven@3.5... (64B)
Pruned 0 symbolic links and 2 directories from /usr/local
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew install kafka
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/kafka-3.2.0.monterey.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/c03f1e5a559c1c99f9082da89614f93f10db9e72b4d9b568478e2ec94a952cdf--kafka-3.2.0.monterey.bottle.tar.gz
==> Pouring kafka-3.2.0.monterey.bottle.tar.gz
==> Caveats
To restart kafka after an upgrade:
  brew services restart kafka
Or, if you don't want/need a background service you can just run:
  /usr/local/opt/kafka/bin/kafka-server-start /usr/local/etc/kafka/server.properties
==> Summary
🍺  /usr/local/Cellar/kafka/3.2.0: 200 files, 99.4MB
==> Running `brew cleanup kafka`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew service restart kafka
Error: Unknown command: service
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services restart kafka
==> Successfully started `kafka` (label: homebrew.mxcl.kafka)
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services start zookeeper
==> Successfully started `zookeeper` (label: homebrew.mxcl.zookeeper)
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services start kafka
Bootstrap failed: 5: Input/output error
Try re-running the command as root for richer errors.
Error: Failure while executing; `/bin/launchctl bootstrap gui/501 /Users/zoubaitao/Library/LaunchAgents/homebrew.mxcl.kafka.plist` exited with 5.
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services stop kafka
Stopping `kafka`... (might take a while)
==> Successfully stopped `kafka` (label: homebrew.mxcl.kafka)
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services stop zookeeper
Stopping `zookeeper`... (might take a while)
==> Successfully stopped `zookeeper` (label: homebrew.mxcl.zookeeper)
```
## 二、配置Kafka集群
## 三、启停Kafka
启动zookeeper
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services start zookeeper
==> Successfully started `zookeeper` (label: homebrew.mxcl.zookeeper)
```

启动Kafka
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services start kafka
==> Successfully started `kafka` (label: homebrew.mxcl.kafka)

```

 关停Kafka
 ```bash
 zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services stop kafka 
Stopping `kafka`... (might take a while)
==> Successfully stopped `kafka` (label: homebrew.mxcl.kafka)
 ```

关停zookeeper
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services stop zookeeper
Stopping `zookeeper`... (might take a while)
==> Successfully stopped `zookeeper` (label: homebrew.mxcl.zookeeper)
```
## 四、

## 参考文档
[mac环境下使用brew安装Kafka(详细过程)](https://cloud.tencent.com/developer/article/1780636)
