# qemu环境搭建

## HomeBrew安装QEMU

今天在在搭建qemu环境的时候，想用homebrew来安装qemu，但是因为我的电脑是macOs 12.0.1，homebrew在安装的时候报错

```bash
zoubaitao@zoubaitaodeMacBook-Pro bin % brew install qemu
Warning: You are using macOS 12.
We do not provide support for this pre-release version.
You will encounter build failures with some formulae.
Please create pull requests instead of asking for help on Homebrew's GitHub,
Twitter or any other official channels. You are responsible for resolving
any issues you experience while you are running this
pre-release version.

==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/glib-2.70.0_1.big_sur.bottle.tar.gz
curl: (22) The requested URL returned error: 404                              

Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/glib/manifests/2.70.0_1
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/f91910d34674d1ed6bd93a3b233c0a1c0efaae5e6988fc044708ee0a074709aa--glib-2.70.0_1.bottle_manifest.json
==> Downloading https://ghcr.io/v2/homebrew/core/glib/blobs/sha256:5346e79808e56e758840d86b100c14b855afe106a96a47c79d44235f9ba33ca5
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/827b32b5cbc8cb8cec8b21bd90bff915a27074f248ee58fcd1337fcf082ccd3b--glib--2.70.0_1.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/bdw-gc-8.0.6.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/477af3b1ab0fb4288153d059c5e369d5fa1736b1242db9d840c980d594354c15--bdw-gc-8.0.6.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/m4-1.4.19.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/ae8fc6e9ae067450874476a7fe37ccb73f4344870015280eeaa091c54d7afb2a--m4-1.4.19.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/libtool-2.4.6_4.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/89f8d8582340c28c78ddce8cfaf3f31e7cab10e6b25bee3f403ad591b0faf4ca--libtool-2.4.6_4.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/libunistring-0.9.10.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/d64af73a9eac07f88461ff30b5110b57453791a22a02ce6d3c6050c95b36c328--libunistring-0.9.10.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/pkg-config-0.29.2_3.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/c393e3a39326eab27929f0f2ce40cb425e78bd8812166e6d835a08a8bf0c5f56--pkg-config-0.29.2_3.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/guile-3.0.7_2.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/7eb76a7aa1cabca0d60171fb3a419d8638fcd518a61b6ece830c8a14861b5d22--guile-3.0.7_2.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/libidn2-2.3.2.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/65939dbe0d1ba8abd1a1d4b1615d834829dfc56a897f58d914db3124f36c8fad--libidn2-2.3.2.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/libtasn1-4.17.0.big_sur.bottle.tar.gz
curl: (22) The requested URL returned error: 404                              

Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/libtasn1/manifests/4.17.0
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/b8db7f6ac2eeccbeb4b809750a552cb1e3e02185b9b3436db6137babe9a4a7e3--libtasn1-4.17.0.bottle_manifest.json
==> Downloading https://ghcr.io/v2/homebrew/core/libtasn1/blobs/sha256:a045a7b16828e7c18bad248feb37207815daea23ff313c1cd6e94e94b222bb73
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/45ca8246d536b1b05ce571c831291268e4ebd15c352435278d4330a61cb23944--libtasn1--4.17.0.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/nettle-3.7.3.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/bf840496cd9303763b5b64cbbf1d8dabe4fa768839d782dba38f67c0041c6349--nettle-3.7.3.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/p11-kit-0.24.0_1.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/fdc5b29201bbb7826c673e6f9992b0908ed48d93297e057b818d592695d8b905--p11-kit-0.24.0_1.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/libevent-2.1.12.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/6d6493108fc89bf355d402b6060505749a31bf09ce6f93286b35618efdf9b3c0--libevent-2.1.12.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/libnghttp2-1.45.1.big_sur.bottle.tar.gz
curl: (22) The requested URL returned error: 404                              

Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/libnghttp2/manifests/1.45.1
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/a10e8809f2b80ff379c8ac93e0e41e69e5a179b36e2ac19968bcfec8abe11b73--libnghttp2-1.45.1.bottle_manifest.json
==> Downloading https://ghcr.io/v2/homebrew/core/libnghttp2/blobs/sha256:6edccdb5f700fa3602caa4ed902c18cdab02e64f33bdaf318a867b30b972a472
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/0ddc64f9fd99263fe024b4c46452c0f244578238ff5a2b0321e5742e8e08522e--libnghttp2--1.45.1.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/unbound-1.13.2_1.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/8a1c51a112c8670c9cf2e9d8785accebd685efad5f46caa84f55cf5419ca2271--unbound-1.13.2_1.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/gnutls-3.6.16_1.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/0d41ff11721a5f0a949e36b81a3bd3dc8cd5153953b0d443ad0defbf6480cf05--gnutls-3.6.16_1.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/jpeg-9d.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/7ed5b41c2937740eca747a8077502454971fbbe02cfb5cfbd9b9e7379345d0cd--jpeg-9d.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/libpng-1.6.37.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/3bd2e2a75fbfc893d9acc20eeafc5274e260ed2ca39483ccbb1450a734bc6775--libpng-1.6.37.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/libslirp-4.6.1.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/a3aa28e68f799f7506c8b98b336c384cfaca74a3a19c41702ea3fc067d030e88--libslirp-4.6.1.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/libssh-0.9.6.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/ceec13b10e97f0f72fa5243855589db89c8eefbd70e85754abef7c0be67b8505--libssh-0.9.6.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/libusb-1.0.24.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/92c8073ef3933693d31390df9fe06253c054bd0763e60333db9a101fd8b1edc5--libusb-1.0.24.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/lzo-2.10.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/0020e09d8a2c473efa8db2af4b402358f5184578c801c7a7650de6c8bedca06a--lzo-2.10.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/ncurses-6.2.big_sur.bottle.1.tar.gz
curl: (22) The requested URL returned error: 404                              

Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/ncurses/manifests/6.2-1
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/735bd3a2fbb4f743a332c92e1041154b795201ddbb12453423345e043de77e59--ncurses-6.2-1.bottle_manifest.json
==> Downloading https://ghcr.io/v2/homebrew/core/ncurses/blobs/sha256:37587c0fbfd02b432e8a65522feadbb865fb754c3fe911b3a584abafb0f0effb
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/206740f72acd4a2f3409255128cc7645ae2a6cef440d0fa3f854fcd23192a720--ncurses--6.2.big_sur.bottle.1.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/pixman-0.40.0.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/ddf94c89d763f2c63c00ce2090ff16d5abd832ca0e1e9beb2245da3cc159ce41--pixman-0.40.0.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/snappy-1.1.9.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/47fc1df980728dd58a2cac57f9c43dd59963fa95af5158f0913932bedd48ef1c--snappy-1.1.9.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/vde-2.3.2_1.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/24d4d44bcce4130d150fcf3f4ccc7150c271aa3cc98a52d173c5715bbc0bc564--vde-2.3.2_1.big_sur.bottle.tar.gz
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/qemu-6.1.0_1.big_sur.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/aba842c85c34d2eadc457803368fd4695b402f40af82391caddd73a4d3ca2e44--qemu-6.1.0_1.big_sur.bottle.tar.gz
==> Installing dependencies for qemu: glib, bdw-gc, m4, libtool, libunistring, pkg-config, guile, libidn2, libtasn1, nettle, p11-kit, libevent, libnghttp2, unbound, gnutls, jpeg, libpng, libslirp, libssh, libusb, lzo, ncurses, pixman, snappy and vde
==> Installing qemu dependency: glib
==> Pouring glib-2.70.0_1.big_sur.bottle.tar.gz
Error: No such file or directory @ rb_sysopen - /Users/zoubaitao/Library/Caches/Homebrew/downloads/aa26834a05512a01e0486972bcbb9bae1931130bb99879312fff0be2027118c3--glib-2.70.0_1.big_sur.bottle.tar.gz
```

所以放弃了，因为看到qemu官网的Mac安装有两种方式，分别是homebrew。

```bash
 brew install qemu
```

和MacPorts

```bash
 sudo port install qemu
```

因为使用Homebrew安装失败，所以只能先安装MacPorts，然后安装qemu。



## 安装MacPorts

[链接](https://www.macports.org/install.php)，这里提供了安装包和源码构建的两种方式，但是在安装MacPorts前，需要先从App Store中安装Xcode和[App Store Developer](https://developer.apple.com/download/all/)中下载Command Line Tools for Xcode。或者使用下面的命令安装Command Line Tools for Xcode。

```bash
xcode-select --install
```

然后下载安装包，一直安装就可以了。安装完MacPorts之后，MacPorts安装器已经设置好了环境变量，可以通过下面的命令查看是否安装成功。

```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % port version
Version: 2.7.1
```

MacPort安装成功之后，就可以通过上面提到的命令安装qemu。

```bash
 sudo port install qemu
```



## 安装QEMU

安装完QEMU之后，但是不知道QEMU安装在哪里，所以我使用了下面的命令查看

```bash
zoubaitao@zoubaitaodeMacBook-Pro bin % where qemu-system-i386
/opt/local/bin/qemu-system-i386
```

所以我们可以进入到这个，目录查看安装的虚拟机

```bash
zoubaitao@zoubaitaodeMacBook-Pro bin % ls
2to3-3.9		bzmore			gio			iconv			lzmadec			msggrep			pcretest		qemu-io			toe			xzegrep
autopoint		c_rehash		gio-querymodules	idle3.9			lzmainfo		msginit			port			qemu-nbd		tput			xzfgrep
bunzip2			c_rehash-3		glib-compile-resources	infocmp			lzmore			msgmerge		port-tclsh		qemu-storage-daemon	tset			xzgrep
bzcat			captoinfo		glib-compile-schemas	infotocap		msgattrib		msgunfmt		portf			qemu-system-aarch64	unlzma			xzless
bzcmp			clear			glib-genmarshal		lzcat			msgcat			msguniq			portindex		qemu-system-arm		unxz			xzmore
bzdiff			daemondo		glib-gettextize		lzcmp			msgcmp			ncurses6-config		portmirror		qemu-system-i386	xgettext
bzegrep			envsubst		glib-mkenums		lzdiff			msgcomm			ncursesw6-config	pydoc3.9		qemu-system-x86_64	xmlwf
bzfgrep			gdbus			gobject-query		lzegrep			msgconv			ngettext		python3.9		recode-sr-latin		xz
bzgrep			gdbus-codegen		gresource		lzfgrep			msgen			openssl			python3.9-config	reset			xzcat
bzip2			gettext			gsettings		lzgrep			msgexec			openssl-3		qemu-edid		sqlite3			xzcmp
bzip2recover		gettext.sh		gtester			lzless			msgfilter		pcre-config		qemu-ga			tabs			xzdec
bzless			gettextize		gtester-report		lzma			msgfmt			pcregrep		qemu-img		tic			xzdiff
zoubaitao@zoubaitaodeMacBook-Pro bin % pwd
/opt/local/bin
```

到此QEMU就安装完成了