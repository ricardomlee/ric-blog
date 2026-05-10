---
description: LEDE
pubDatetime: 2023-03-04
tags:
- Others
title: Build OpenWrt under Arch
---

install dependencies

```bash
# Essential prerequisites
pacman -S --needed base-devel autoconf automake bash binutils bison \
bzip2 fakeroot file findutils flex gawk gcc gettext git grep groff \
gzip libelf libtool libxslt m4 make ncurses openssl patch pkgconf \
python rsync sed texinfo time unzip util-linux wget which zlib
 
# Optional prerequisites, depend on the package selection
pacman -S --needed asciidoc help2man intltool perl-extutils-makemaker swig
```

clone and config lede

```bash
git clone https://github.com/coolsnowwolf/lede
cd lede
# add extra packages
git clone https://github.com/kenzok8/small-package package/small-package
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig
```

make

```bash
make download -j8
make V=s -j1
```

23.11.19 update

disable miniupnpd

build_dir/hostpkg/gn-2023-08-01-811d332b/src/gn/desc_builder.cc, remove return std::move

```bash
make download -j12
# in WSL2
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin make V=s -j12
```